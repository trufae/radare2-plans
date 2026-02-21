# Egg Codegen Backend Tasks

This document outlines the specific issues and solutions for the Egg language codegen backend system, based on analysis of the current callback structure across all supported architectures.

## Critical Issues (Must Fix)

### 1. Missing `retvar` Field in ARM/ARM64/ESIL Backends

**Problem**: 
- ARM, ARM64, and ESIL backends don't define the `retvar` field
- accessing `.ret` variable causes crashes/undefined behavior
- Used in `egg_lang.c:511` for return value resolution

**Current Status**:
```c
// emit_x86.c - WORKING
.retvar = R_AX,  // "eax" or "rax"

// emit_arm.c - MISSING
// No .retvar defined

// emit_arm64.c - MISSING  
// No .retvar defined

// emit_esil.c - MISSING
// No .retvar defined
```

**Solution**:
```c
// emit_arm.c
.retvar = "r0",

// emit_arm64.c  
.retvar = "x0",

// emit_esil.c
.retvar = "r0",
```

**Files to modify**: `emit_arm.c`, `emit_arm64.c`, `emit_esil.c`

### 2. Missing `get_ar` Callback Implementation

**Problem**:
- Only x86 backend implements `get_ar` for `.rarg` variable access
- Register arguments (`.rarg0`, `.rarg1`, etc) broken on ARM/ARM64/ESIL
- Used in `egg_lang.c:521-523` for register argument resolution

**Current Status**:
```c
// emit_x86.c - WORKING
.get_ar = emit_get_ar,
static void emit_get_ar(REgg *egg, char *out, int idx) { ... }

// emit_arm.c - MISSING
// No .get_ar defined

// emit_arm64.c - MISSING  
// No .get_ar defined

// emit_esil.c - MISSING
// No .get_ar defined
```

**Solution**: Add implementation for each architecture:
```c
// emit_arm.c
static void emit_get_ar(REgg *egg, char *out, int idx) {
    const char *regs[] = {"r0", "r1", "r2", "r3", "r4", "r5", "r6", "r7"};
    if (idx >= 0 && idx < 8) {
        strcpy(out, regs[idx]);
    }
}
.get_ar = emit_get_ar,

// emit_arm64.c
static void emit_get_ar(REgg *egg, char *out, int idx) {
    const char *regs[] = {"x0", "x1", "x2", "x3", "x4", "x5", "x6", "x7"};
    if (idx >= 0 && idx < 8) {
        strcpy(out, regs[idx]);
    }
}
.get_ar = emit_get_ar,

// emit_esil.c  
static void emit_get_ar(REgg *egg, char *out, int idx) {
    snprintf(out, 32, "r%d", idx);
}
.get_ar = emit_get_ar,
```

**Files to modify**: `emit_arm.c`, `emit_arm64.c`, `emit_esil.c`

## Design Improvements (Should Fix)

### 3. Over-Segmented While Loop Handling

**Problem**:
- `while_end` + `get_while_end` callbacks create unnecessary complexity
- Complex interaction with generic branching logic
- Each backend reimplements similar label generation

**Current Implementation**:
```c
// Two callbacks for while loops
.while_end = emit_while_end,
.get_while_end = emit_get_while_end,
```

**Solution**: Simplify to single callback or use generic `branch`:
```c
// Remove get_while_end, use generic branch with loop-specific labels
// Or create unified loop handling
```

**Files to modify**: All `emit_*.c` files, `r_egg.h`

### 4. Inconsistent String Management

**Problem**:
- `set_string` callback reimplemented differently in each backend
- String encoding/escaping logic duplicated
- Architecture-specific optimizations make code hard to maintain

**Current Issues**:
```c
// Each backend has its own string handling
// emit_x86.c - Complex word-based processing with endian handling
// emit_arm.c - Word-alignment manual calculation  
// emit_esil.c - Simple character-by-character processing
```

**Solution**: 
1. Move string processing to common code in `egg_lang.c`
2. Use `equ` callback for architecture-specific label generation
3. Implement unified escape sequence handling

**Files to modify**: `egg_lang.c`, all `emit_*.c` files

### 5. Redundant Stack Management

**Problem**:
- `restore_stack` callback duplicates `frame_end` functionality
- Inconsistent usage across backends
- Some backends leave it as no-op

**Current Status**:
```c
// emit_x86.c - Implements stack restoration  
// emit_arm.c - Empty implementation
// emit_esil.c - Empty implementation
```

**Solution**: 
1. Merge `restore_stack` logic into `frame_end`
2. Remove `restore_stack` callback entirely
3. Handle stack restoration in common code

**Files to modify**: `r_egg.h`, all `emit_*.c` files

## Proposed Simplified Interface

### Core Required Callbacks (16 vs current 24)
```c
typedef struct r_egg_emit_t {
    const char *arch;           // Architecture identifier
    int size;                   // Register size in bytes  
    const char *retvar;         // Return value register name (REQUIRED!)
    
    // Basic control flow
    void (*init)(REgg *egg);
    void (*call)(REgg *egg, const char *addr, int ptr);
    void (*jmp)(REgg *egg, const char *addr, int ptr);
    
    // Function management
    void (*frame)(REgg *egg, int sz);
    void (*frame_end)(REgg *egg, int sz, int ctx);
    
    // Variables and registers
    const char* (*regs)(REgg *egg, int idx);
    void (*get_var)(REgg *egg, int type, char *out, int idx);
    void (*get_ar)(REgg *egg, char *out, int idx);  // MAKE REQUIRED
    
    // Data and operations  
    void (*branch)(REgg *egg, char *b, char *g, char *e, char *n, int sz, const char *dst);
    void (*mathop)(REgg *egg, int ch, int sz, int type, const char *eq, const char *p);
    void (*load)(REgg *egg, const char *str, int sz);
    void (*load_ptr)(REgg *egg, const char *str);
    
    // Syscalls and system calls
    char *(*syscall)(REgg *egg, int num);
    void (*syscall_args)(REgg *egg, int nargs);
    
    // Utility (can have default implementations)
    void (*comment)(REgg *egg, const char *fmt, ...);
    void (*equ)(REgg *egg, const char *key, const char *value);
    void (*trap)(REgg *egg);
} REggEmit;
```

### Removed Callbacks
- `set_string` → Handle in common code
- `while_end` + `get_while_end` → Use generic `branch`  
- `restore_stack` → Merge into `frame_end`
- `get_result` → Use `retvar` field directly
- `push_arg` → Derive from `regs` + `load`

## Implementation Plan

### Phase 1: Critical Fixes (Priority: High)
- [ ] Add `retvar` to ARM/ARM64/ESIL backends
- [ ] Implement `get_ar` for ARM/ARM64/ESIL backends  
- [ ] Test `.ret` and `.rarg` variable access on all architectures
- [ ] Update documentation with new requirements

### Phase 2: Interface Cleanup (Priority: Medium)
- [ ] Simplify while loop handling
- [ ] Unify string processing in common code
- [ ] Remove redundant `restore_stack` callback
- [ ] Add default implementations for utility callbacks
- [ ] Update all backends to use simplified interface

### Phase 3: Testing & Validation (Priority: Medium)  
- [ ] Create comprehensive test suite for all backends
- [ ] Test all Egg language features on each architecture
- [ ] Validate no regressions in existing functionality
- [ ] Update examples and documentation

## Testing Strategy

### Critical Fixes Testing
```egg
// Test .ret variable access
test_ret@global(32, 0) {
    .var0 = 42;
    .ret = .var0;
}

// Test .rarg variable access  
test_rarg@global(32, 0) {
    .ret = .rarg0;  // Should work on all architectures
}
```

### Architecture-Specific Testing
- Test on x86, x86-64, ARM, ARM64, ESIL backends
- Verify register calling conventions
- Check stack alignment and frame management
- Validate syscall generation for different OS targets

## Benefits of Implementation

### Reliability Improvements
- Elimination of crashes on ARM/ARM64/ESIL architectures
- Consistent behavior across all supported backends
- Better error handling and validation

### Maintainability Benefits  
- Reduced code duplication across backends
- Simpler callback interface (16 vs 24 callbacks)
- Common code handling for shared functionality
- Easier to add new architectures

### Language Alignment
- Callback interface matches Egg language features
- Clear separation between required and optional functionality
- Better support for future language extensions

## Notes

- All changes should maintain backward compatibility where possible
- Critical fixes (Phase 1) can be implemented independently
- Interface changes (Phase 2) require coordinated updates across all backends
- Testing should be comprehensive to avoid regressions
- Documentation should be updated with new callback requirements