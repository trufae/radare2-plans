# Opcode Navigation Refactoring Plan

## Problem Statement

The codebase contains duplicated and scattered logic for seeking to the nth instruction forward/backward across multiple locations:

1. **cmd_seek_opcode_backward/forward** (libr/core/cmd_seek.inc.c) - The main implementation handling `so` command
2. **r_core_seek_next/previous** (libr/core/vmenus.c) - Used for visual mode navigation, incomplete (TODO for backward case)
3. **r_core_prevop_addr/r_core_prevop_addr_force** (libr/core/visual.c) - Low-level helpers for finding previous instruction
4. **prevop_addr** (libr/core/visual.c) - Static helper for single backward instruction traversal
5. **r_core_asm_bwdisassemble** (libr/core/casm.c) - Another backward disassembly implementation
6. **r_core_asm_bwdis_len** (libr/core/casm.c) - Wrapper combining backward navigation with info extraction
7. Scattered uses in **cmd_print.inc.c, cmd_search.inc.c, disasm.c** - Multiple callers with different semantics

### Key Issues

- **Code Duplication**: The forward/backward traversal logic is reimplemented multiple times
- **Inconsistent Analysis Usage**: Some implementations use basic block metadata (`RAnalBlock`), others don't
- **Missing Hint Support**: Most implementations ignore `RAnalHint` data that may override opcode sizes
- **Meta Type Ignorance**: Navigating through data regions (strings, arrays) isn't properly handled
- **Inconsistent Seeking**: The `r_core_seek_previous` for opcodes is stubbed (TODO), forcing reliance on fallback code
- **Mixed Concerns**: Decoding, seeking, and information gathering are tangled together
- **API Fragmentation**: Three separate function families for essentially the same operation

## Solution Overview

Create a unified, clean public API in **libr/anal** that:

1. Handles instruction-level navigation (forward/backward by N instructions)
2. Respects analysis information when available (basic blocks, hints, metadata)
3. Gracefully falls back when analysis is absent
4. Distinguishes between true instruction navigation vs. size-only traversal
5. Provides detailed feedback about what was traversed (addresses, sizes, types)
6. Eliminates duplication and unifies concepts across the codebase

The new functions will live in `libr/anal` (not libr/core) because:
- This is core analysis functionality independent of UI/CLI
- It should not depend on RCore context, only RIO and RAnalysis
- It benefits both core and external tools
- Follows the principle of pushing analysis logic down to the analysis library

## Unified Single-Function Design (Recommended)

**Approach**: Create ONE clean, simple public API that hides all complexity internally:

```c
// Single public function - complexity is internal
R_API ut64 r_anal_sibling(RAnalysis *anal, RIO *io, ut64 addr, int nth);
```

**Semantics**:
- `nth > 0`: Move forward by `nth` instructions
- `nth < 0`: Move backward by `-nth` instructions  
- `nth == 0`: Returns current address (or UT64_MAX on error)
- Returns: Target address, or UT64_MAX on error

**Internal Complexity** (hidden in single file):
- Basic block information lookup
- Hint processing for size overrides
- Metadata region handling (strings, data arrays)
- Architecture minop/maxop fallback
- Buffer-based instruction decoding
- Boundary checking
- prevop_addr_force logic for backward navigation

**Rationale**: 
- Simple, intuitive public API
- Consistent naming with radare2 conventions (r_anal_* for analysis functions)
- All complexity encapsulated in implementation
- Easily testable
- Easy to document and use
- Scales to handle both forward and backward with single function
- All supporting logic stays internal, never exposed

## Step-by-Step Implementation Plan

### Phase 1: Foundation (New API in libr/anal)

#### 1.1 Create single implementation file: `libr/anal/sibling.c`

Single public function:
```c
R_API ut64 r_anal_sibling(RAnalysis *anal, RIO *io, ut64 addr, int nth);
```

Internal static helper functions (never exposed):
- `sibling_forward()` - navigate forward by N instructions
  - Check if addr is in analyzed basic block, use block info if available
  - Fall back to architecture minop/maxop for non-analyzed regions
  - Check for hints overriding opcode size
  - Check for metadata regions (strings, data arrays)
  - Use buffer-based decoding to avoid repeated I/O
  - Handle boundary conditions (address overflow)
  - Return final address
  
- `sibling_backward()` - navigate backward by N instructions
  - Use prevop_addr_force logic as fallback
  - Iterate backward checking hints and metadata
  - Handle basic block boundaries
  - Return final address

- `get_opcode_size_at()` - unified size determination
  - Check hint for size override
  - Check basic block metadata
  - Check architecture minop/maxop
  - Fall back to decoding if needed

- `get_current_metadata_type()` - determine region type
  - Check if at string/data region
  - Return metadata type if applicable

**Key Design**:
- Single file, all complexity internal
- No structs exported to public API
- Simple ut64 return value
- All buffer management internal
- Unified handling for forward/backward via `nth` sign

### Phase 2: Consolidate in libr/core

#### 2.1 Refactor `cmd_seek_opcode_forward()` 

**MODIFY**: `static int cmd_seek_opcode_forward(RCore *core, int numinstr)` in `libr/core/cmd_seek.inc.c`

Replace entire implementation with single call:
```c
static int cmd_seek_opcode_forward(RCore *core, int numinstr) {
	R_RETURN_VAL_IF_FAIL(core && numinstr > 0, 0);
	ut64 oaddr = core->addr;
	ut64 new_addr = r_anal_sibling(core->anal, core->io, oaddr, numinstr);
	if (new_addr == UT64_MAX) {
		return 0;
	}
	r_io_sundo_push(core->io, oaddr, r_print_get_cursor(core->print));
	r_core_seek(core, new_addr, true);
	return (int)(new_addr - oaddr);
}
```

#### 2.2 Refactor `cmd_seek_opcode_backward()`

**MODIFY**: `static int cmd_seek_opcode_backward(RCore *core, int numinstr)` in `libr/core/cmd_seek.inc.c`

Replace entire implementation with single call:
```c
static int cmd_seek_opcode_backward(RCore *core, int numinstr) {
	if (numinstr <= 0) {
		R_LOG_DEBUG("Invalid instruction number");
		return 0;
	}
	ut64 oaddr = core->addr;
	ut64 new_addr = r_anal_sibling(core->anal, core->io, oaddr, -numinstr);
	if (new_addr == UT64_MAX) {
		return 0;
	}
	r_io_sundo_push(core->io, oaddr, r_print_get_cursor(core->print));
	r_core_seek(core, new_addr, true);
	return (int)(oaddr - new_addr);
}
```

#### 2.3 Implement `r_core_seek_next()` opcode case

**MODIFY**: `r_core_seek_next()` in `libr/core/vmenus.c`

Replace the "opc" branch:
```c
if (strstr(type, "opc")) {
	next = r_anal_sibling(core->anal, core->io, core->addr, 1);
	found = (next != UT64_MAX && next != core->addr);
}
```

#### 2.4 Implement `r_core_seek_previous()` opcode case

**MODIFY**: `r_core_seek_previous()` in `libr/core/vmenus.c`

Replace the TODO warning with:
```c
if (strstr(type, "opc")) {
	next = r_anal_sibling(core->anal, core->io, core->addr, -1);
	found = (next != UT64_MAX && next != core->addr);
}
```

#### 2.5 Refactor cmd_print.inc.c usage

**MODIFY**: Line ~7354 in `libr/core/cmd_print.inc.c`

Replace:
```c
if (!r_core_prevop_addr(core, core->addr, l, &start)) {
    start = r_core_prevop_addr_force(core, core->addr, l);
}
```

With:
```c
start = r_anal_sibling(core->anal, core->io, core->addr, -l);
if (start == UT64_MAX) {
    return 0;  // or handle error appropriately
}
```

#### 2.6 Refactor cmd_search.inc.c usage

**MODIFY**: Lines ~4448 and ~4468 in `libr/core/cmd_search.inc.c`

Replace both uses of `r_core_prevop_addr*`:
- Line ~4448: Replace with `r_anal_sibling(core->anal, core->io, core->addr, -n)`
- Line ~4468: Replace with `r_anal_sibling(core->anal, core->io, core->addr, -n)`

### Phase 3: Deprecation & Cleanup

#### 3.1 Delete or deprecate old functions

In libr/core/visual.c:
- **DELETE**: `static prevop_addr()` - logic now in r_anal_sibling()
- **KEEP BUT DEPRECATE**: `r_core_prevop_addr()` - add `R_DEPRECATED` macro, document to use `r_anal_sibling()` instead
- **KEEP BUT DEPRECATE**: `r_core_prevop_addr_force()` - add `R_DEPRECATED` macro, same as above

In libr/core/cmd_seek.inc.c:
- ~~DELETE: `static int cmd_seek_opcode_backward()`~~ - Now just thin wrapper (MODIFIED, not deleted)
- ~~DELETE: `static int cmd_seek_opcode_forward()`~~ - Now just thin wrapper (MODIFIED, not deleted)

In libr/core/casm.c:
- **EVALUATE**: `r_core_asm_bwdisassemble()` - check if still used after refactoring cmd_search.inc.c
- **EVALUATE**: `r_core_asm_bwdis_len()` - check if still used after refactoring disasm.c

#### 3.2 Update r_core.h

Add new function:
```c
// New unified opcode navigation function
R_API ut64 r_anal_sibling(RAnalysis *anal, RIO *io, ut64 addr, int nth);
```

Deprecate old functions:
```c
// DEPRECATED: Use r_anal_sibling() instead
R_API bool r_core_prevop_addr(RCore* core, ut64 start_addr, int numinstrs, ut64* prev_addr) R_DEPRECATED;
R_API ut64 r_core_prevop_addr_force(RCore *core, ut64 start_addr, int numinstrs) R_DEPRECATED;
```

#### 3.3 Update meson.build/Makefile

Add `sibling.c` to libr/anal sources (usually via glob pattern `libr/anal/*.c`).

### Phase 4: Testing

#### 4.1 Create/update test file: `test/db/cmd/cmd_seek`

Add test cases to existing cmd_seek tests:
- Single forward/backward instruction with `r_anal_sibling()`
- Multiple forward/backward instructions
- At boundaries (start of file, end of file)
- Through analyzed code (basic blocks)
- Through hints overriding sizes
- Through data/string regions
- Seek history preservation
- Fixed-size vs variable-size architectures
- Test visual mode navigation (n/N keys)

#### 4.2 Compile and run full test suite

```bash
make -C libr/anal        # Verify compilation
r2r test/db/cmd/cmd_seek  # Run all seek tests
r2r test/db/cmd/cmd_seek_navigate  # If new test file created
```

## Summary of Changes

### Functions to Delete

1. `static ut64 prevop_addr()` - libr/core/visual.c (L1199-1267)
   - Logic integrated into `r_anal_sibling()`

### Functions to Add

1. `R_API ut64 r_anal_sibling(RAnalysis *anal, RIO *io, ut64 addr, int nth)` - libr/anal/sibling.c (public API)
   - Returns sibling instruction address (forward if nth>0, backward if nth<0)

**Internal helpers (never exposed, static only)**:
2. `static ut64 sibling_forward()` - libr/anal/sibling.c
3. `static ut64 sibling_backward()` - libr/anal/sibling.c
4. `static ut32 get_opcode_size_at()` - libr/anal/sibling.c
5. `static int get_current_metadata_type()` - libr/anal/sibling.c

### Functions to Modify

1. `static int cmd_seek_opcode_forward()` - libr/core/cmd_seek.inc.c
   - Replace ~50 lines of implementation with single `r_anal_sibling()` call
   
2. `static int cmd_seek_opcode_backward()` - libr/core/cmd_seek.inc.c
   - Replace ~60 lines of implementation with single `r_anal_sibling()` call
   
3. `r_core_seek_next()` - libr/core/vmenus.c (L4005-4043)
   - Replace "opc" branch with `r_anal_sibling(core->anal, core->io, core->addr, 1)`
   
4. `r_core_seek_previous()` - libr/core/vmenus.c (L4045-4070)
   - Replace TODO warning with `r_anal_sibling(core->anal, core->io, core->addr, -1)`
   
5. cmd_print.inc.c (L7354) - pd command implementation
   - Replace `r_core_prevop_addr()` + `r_core_prevop_addr_force()` with `r_anal_sibling()`
   
6. cmd_search.inc.c (L4448, L4468) - search command implementation
   - Replace both `r_core_prevop_addr*()` calls with `r_anal_sibling()`

7. `r_core_prevop_addr()` and `r_core_prevop_addr_force()` - libr/core/visual.c
   - Add `R_DEPRECATED` macro, keep for compatibility but document to use `r_anal_sibling()` instead

### Files Changed

- **New**: libr/anal/sibling.c (~300-400 lines)
- **Modified**: libr/core/cmd_seek.inc.c (~110 lines deleted, ~10 lines added = -100 net)
- **Modified**: libr/core/vmenus.c (~20 lines changed)
- **Modified**: libr/core/cmd_print.inc.c (~10 lines changed)
- **Modified**: libr/core/cmd_search.inc.c (~10 lines changed)
- **Modified**: libr/core/visual.c (~70 lines deleted - prevop_addr function)
- **Modified**: libr/include/r_core.h (add r_anal_sibling, deprecate old functions)
- **Modified**: test/db/cmd/cmd_seek (add new test cases)

### Expected Outcome

- **LOC Reduction**: ~200-250 lines deleted (removing duplicated logic and prevop_addr)
- **Code Unification**: All opcode navigation uses single, clean API
- **Feature Completion**: `r_core_seek_previous()` finally implemented for opcodes
- **Analysis Integration**: Proper support for hints and metadata
- **Maintainability**: Centralized logic makes future changes easier

## Implementation Checklist

Phase 1: Foundation
- [ ] Create libr/anal/sibling.c with r_anal_sibling() implementation
  - [ ] Implement sibling_forward() helper
  - [ ] Implement sibling_backward() helper  
  - [ ] Implement get_opcode_size_at() with BB/hint/metadata logic
  - [ ] Implement get_current_metadata_type() helper
  - [ ] Handle boundary conditions (overflow, underflow)
  - [ ] Test with fixed-size and variable-size architectures

Phase 2: Consolidation
- [ ] Refactor cmd_seek_opcode_forward() in libr/core/cmd_seek.inc.c
- [ ] Refactor cmd_seek_opcode_backward() in libr/core/cmd_seek.inc.c
- [ ] Update r_core_seek_next() in libr/core/vmenus.c (opc branch)
- [ ] Implement r_core_seek_previous() in libr/core/vmenus.c (opc branch)
- [ ] Update cmd_print.inc.c caller (~L7354)
- [ ] Update cmd_search.inc.c callers (~L4448, ~L4468)

Phase 3: Cleanup
- [ ] Delete static prevop_addr() from libr/core/visual.c
- [ ] Add R_DEPRECATED to r_core_prevop_addr() and r_core_prevop_addr_force()
- [ ] Add r_anal_sibling() declaration to libr/include/r_core.h
- [ ] Update libr/anal/Makefile/meson.build to include sibling.c

Phase 4: Testing
- [ ] Update/create test cases in test/db/cmd/cmd_seek
- [ ] Compile: `make -C libr/anal`
- [ ] Run seek tests: `r2r test/db/cmd/cmd_seek`
- [ ] Verify visual mode navigation (n/N keys work)
- [ ] Test hint and metadata handling
- [ ] Test boundary conditions

Validation
- [ ] No regressions in cmd_seek tests
- [ ] Visual mode opcode navigation works (both n and N keys)
- [ ] LOC reduction verified (~200+ lines deleted)
- [ ] Code compiles without warnings
- [ ] No functional changes to user-visible behavior

## Benefits

1. **Code Reduction**: Remove ~200-250 lines of duplicated logic across 5+ files
2. **Unified API**: Single function `r_anal_sibling()` handles all opcode navigation
3. **Analysis Integration**: Hints, metadata, and basic blocks properly respected
4. **Feature Completion**: Finally implements backward opcode seeking in visual mode
5. **Maintainability**: Centralized logic easier to understand, test, and modify
6. **Performance**: Buffer-based decoding, optimized for fixed-size architectures
7. **Consistency**: All callers use same semantics and error handling
8. **API Clarity**: Clean, simple public API with hidden complexity
