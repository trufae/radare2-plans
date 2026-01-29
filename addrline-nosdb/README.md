# Migration Plan: Remove sdb_addrinfo and Use RBinAddrLineStore Exclusively

## Overview
This document outlines the complete migration plan to remove the legacy `sdb_addrinfo` storage mechanism and transition fully to the optimized `RBinAddrLineStore` system in radare2.

**Current Status**: `sdb_addrinfo` is marked for deprecation in `libr/include/r_bin.h:493`.

**Goal**: Eliminate the sdb-based storage entirely, reducing memory overhead and improving performance through the hash-table and string-pool based `RBinAddrLineStore`.

---

## 1. Architecture Overview

### Current State (Legacy + New Hybrid)
- **sdb_addrinfo**: A simple key-value database storing address-to-line mappings as strings
  - Key format: Hex address (e.g., `0x401000`)
  - Value format: `file:line` or `file|line|column` (format varies)
  - Uses bidirectional mapping: address → file:line AND file:line → address

- **RBinAddrLineStore**: Modern optimized storage
  - Internal structure: `AddrLineStore` with:
    - `HtUP *ht`: Hash table for O(1) address lookups
    - `RList *list`: Linked list for sequential iteration
    - `RStrpool *pool`: Deduplicates file paths to save memory
  - Public API: Function pointers in `RBinAddrLineStore` struct
  - Data model: `RBinAddrline` items with address, file/path indices, line, column

### Migration Target
- Eliminate all references to `sdb_addrinfo`
- Use `RBinAddrLineStore` exclusively through public API (`r_bin_addrline_*` functions)
- Remove the fallback code paths that check for sdb_addrinfo

---

## 2. Use Case Analysis

### 2.1 DWARF Parser (libr/bin/dwarf.c)
**Current Usage:**
- **Lines 701, 711, 713**: Storing and retrieving compilation directory metadata (`DW_AT_comp_dir`)
  - These are used to resolve include directories during line header parsing
  - Code: `sdb_const_get()`, `sdb_get()` to fetch compilation directory
  
- **Line 1206-1234**: `add_sdb_addrline()` function
  - Internal helper that populates BOTH sdb_addrinfo AND the new addrline API
  - Currently writes to sdb for `add_sdb_addrline` (but currently commented out for actual sdb writes)
  
- **Line 1266**: Check if `sdb_addrinfo` exists before adding line info
  - Guards: `if (binfile && binfile->sdb_addrinfo && hdr->file_names)`

- **Line 2621**: `parse_info_raw()` called with `bf->sdb_addrinfo` parameter
  - Used to pass compilation directory metadata to the DWARF info parser

**Migration Action:**
- Move compilation directory metadata from `sdb_addrinfo` to a separate field in `RBinFile`
- Options:
  1. Add `RStrpool comp_dirs` or `RStrBuf comp_dir` to `RBinFile`
  2. Use `addrline.al_add_cu()` to store compilation unit info (currently stubbed)
  3. Store in `RBinAddrline.path` field (already exists but unused)
- Remove all `sdb_addrinfo` reads in DWARF parser
- Ensure `add_sdb_addrline()` only calls `bf->addrline.al_add()`

---

### 2.2 Command Metadata Display (libr/core/cmd_meta.inc.c)

#### Use Case 2.2.1: CL Command (Lines 495-568)
**Purpose:** Display and query address-to-line mappings

**Current Dual-Path Logic:**
- **Lines 501-505**: If `addrline.used`, iterate with `r_bin_addrline_foreach()`
  - Calls `print_addrinfo2()` callback (lines 315-335)
- **Lines 558-559, 564-565**: Fallback to `sdb_foreach()` on old sdb_addrinfo
  - Calls `print_addrinfo()` callback (line 338-372) or `print_addrinfo_json()` (line 210-263)

**Migration Action:**
- Remove lines 558-559 and 564-565 (sdb fallback)
- Remove `print_addrinfo()` and `print_addrinfo_json()` functions (marked as "R2_600 - DEPRECATE")
- Ensure `print_addrinfo2()` and `print_addrinfo2_json()` are always used
- Keep the logic that tries new API first, but make it mandatory

#### Use Case 2.2.2: CL Add Command (Lines 374-399)
**Purpose:** Manually add address-to-line metadata via `CL filename:line 0xaddr`

**Current Implementation:**
- Lines 375-383: Already using the new API: `bf->addrline.al_add()`
- Lines 385-397: Commented-out legacy code using sdb_addrinfo (kept for reference)

**Migration Action:**
- Already migrated. Just remove the #else block (lines 384-397) and clean up

#### Use Case 2.2.3: CL Query Command (Lines 548-568)
**Purpose:** Query line info for a specific address

**Current Implementation:**
- Lines 557, 563: Tries new API first
- Lines 558-559, 564-565: Falls back to sdb if addrline is empty

**Migration Action:**
- Remove fallback code
- Remove checks for `bf->sdb_addrinfo`
- Assert that if no address data found, return gracefully (don't iterate sdb)

---

### 2.3 Debug Info Display (libr/core/cmd_print_list.c)

#### Use Case 2.3.1: Source File List (Lines 45-75)
**Purpose:** Extract unique source files from debug info for display in the UI

**Current Implementation:**
- Lines 52-69: Directly iterates `sdb_foreach_list(bf->sdb_addrinfo)`
  - Parses keys to find file paths (looks for `/` or `.c` extensions)
  - Deduplicates and sorts results

**Migration Action:**
- Use `r_bin_addrline_files()` API instead (already exists!)
  - This function is in `libr/bin/addrline.c` and uses the internal store
  - Simplifies the logic significantly
- Remove the manual sdb iteration and string parsing

---

### 2.4 Debug DWARF Writing (libr/core/p/core_writedwarf.c)

#### Use Case 2.4.1: Line Export for DWARF Output (Lines 718-729)
**Purpose:** Export stored line information when writing DWARF debug info

**Current Implementation:**
- Lines 719-721: If addrline is used, iterate with `r_bin_addrline_foreach()`
  - Calls `addrline_cb()` callback
- Lines 723-727: Fallback for legacy sdb_addrinfo
  - Marked as "Falling back to legacy sdb_addrinfo"
  - Currently does nothing: `sdb_foreach(bin->cur->sdb_addrinfo, NULL, NULL)`

**Migration Action:**
- Remove lines 722-727 (the fallback block)
- Make the addrline path mandatory
- The addrline callback should be sufficient

---

### 2.5 File Operations (libr/bin/bfile.c)

#### Use Case 2.5.1: Allocation (Line 745)
**Current:** `bf->sdb_addrinfo = sdb_new0();`
**Migration Action:** Remove this allocation entirely

#### Use Case 2.5.2: Deallocation (Line 1015)
**Current:** `sdb_free(bf->sdb_addrinfo);`
**Migration Action:** Remove this deallocation

#### Use Case 2.5.3: File Merge (Line 1435)
**Current:** `sdb_merge(dst->sdb_addrinfo, src->sdb_addrinfo);`
**Migration Action:** 
- Already handles addrline merge on line 1437-1438
- But sdb_merge is redundant if no more sdb_addrinfo
- Verify that addrline merge is complete, then remove sdb_merge line

#### Use Case 2.5.4: Store Initialization (Lines 681-698)
**Current:** `addrline_store_init()` already properly initializes the new store
**Note:** No changes needed here; this is already correct

#### Use Case 2.5.5: File Namespace Registration (libr/bin/bobj.c:289)
**Current:** `sdb_ns_set(bdb, "addrinfo", bf->sdb_addrinfo);`
**Migration Action:**
- This registers the sdb_addrinfo as a namespace in the bin's SDB
- Once removed, ensure no code depends on querying `bin.cur.addrinfo` from SDB
- The addrline store is not persisted in SDB; it's in-memory only
- This is intentional for the optimized design

---

### 2.6 Serialization/Persistence
**Note:** The old sdb_addrinfo was persisted via SDB's serialization mechanism.

**Current Status:**
- `RBinAddrLineStore` is **NOT** persisted to disk
- It's populated fresh each time from debug info (DWARF, PDB, etc.)
- This is acceptable because line info is reconstructed from binary debug sections

**Migration Action:**
- Verify that no code expects persistent storage of line info
- Confirm that lazy loading from debug sections on each run is acceptable
- If persistence is needed, design a new serialization format

---

### 2.7 Other Minor Uses

#### libr/core/cmd_search.inc.c (Lines 1850-1915)
**Function:** `esil_addrinfo()` - ESIL operation for address info lookup
**Status:** Already uses the addrline API
**Action:** Verify it doesn't reference sdb_addrinfo (it doesn't)

#### libr/bin/dwarf.c (Line 1266)
**Code:** `if (binfile && binfile->sdb_addrinfo && hdr->file_names)`
**Action:** Remove the sdb_addrinfo check once sdb_addrinfo is eliminated

---

## 3. API Reference for Migration

### Public API to Use (in libr/bin/addrline.c)

```c
// Iteration
R_API bool r_bin_addrline_foreach(RBin *bin, RBinDbgInfoCallback cb, void *user)

// Lookup
R_API const RBinAddrline *r_bin_addrline_get(RBin *bin, ut64 addr)
R_API const RBinAddrline *r_bin_dbgitem_at(RBin *bin, ut64 addr)

// String resolution
R_API const char *r_bin_addrline_str(RBin *bin, ut32 idx)

// Formatting
R_API char *r_bin_addrline_tostring(RBin *bin, ut64 addr, int origin)

// File list
R_API RList *r_bin_addrline_files(RBin *bin)

// Modification
R_API void r_bin_addrline_reset(RBin *bin)
R_API void r_bin_addrline_reset_at(RBin *bin, ut64 addr)
```

### Internal Storage API (for implementation details)

```c
// Only used in bfile.c implementation:
als->al_add(als, addr, file, path, line, column)
als->al_reset(als)
als->al_foreach(als, cb, user)
als->al_get(als, addr)
als->al_files(als)
als->al_str(als, idx)
```

---

## 4. Detailed Migration Steps

### Phase 1: Code Cleanup (Precursor)
**Goal:** Remove obviously dead/commented code to reduce noise

1. **libr/core/cmd_meta.inc.c (Lines 384-397)**
   - Remove the `#else` block with legacy sdb code
   - Keep only the new API path

### Phase 2: Remove Fallback Code Paths
**Goal:** Make the new API mandatory by removing all sdb fallbacks

1. **libr/core/cmd_meta.inc.c (Lines 557-567)**
   - Remove sdb fallback in CL query command
   - Change:
     ```c
     if (!r_bin_addrline_foreach(...)) {
       if (bf && bf->sdb_addrinfo) {
         sdb_foreach(...);
       }
     }
     ```
   - To: Just call `r_bin_addrline_foreach()` without fallback

2. **libr/core/cmd_print_list.c (Lines 52-69)**
   - Replace `sdb_foreach_list(bf->sdb_addrinfo)` with `r_bin_addrline_files()`
   - Simplify the file extraction logic

3. **libr/core/p/core_writedwarf.c (Lines 722-727)**
   - Remove the legacy fallback block
   - Keep only the addrline path

### Phase 3: Migrate DWARF Metadata Storage
**Goal:** Move compilation directory metadata out of sdb_addrinfo

**Current Problem:**
- Compilation directories stored as sdb entries with keys like `"DW_AT_comp_dir"` or `"DW_AT_comp_dir#<offset>"`
- Needed during DWARF parsing to resolve include paths

**Solution Options:**

**Option A (Recommended): Use RBinFile Fields**
```c
// Add to RBinFile struct:
struct {
  char *comp_dir;           // Main compilation directory
  RHashTable *comp_dirs;    // Map of offset -> comp_dir (for multiple CUs)
} dwarf_metadata;
```

**Option B: Extend RBinAddrline**
- Use the existing `path` field to store compilation unit info
- But this conflates address-line with CU metadata

**Option C: Use al_add_cu() Stub**
- Properly implement `al_add_cu()` to store CU metadata
- Create internal structures for CU information
- Would require extending RBinAddrLineStore API

**Recommended Implementation: Option A**
1. Add `RBinDwarfMetadata` structure to `RBinFile`
2. Migrate DWARF parser to store comp_dir there
3. Update DWARF parsing to retrieve from new location

**Files to Modify:**
- `libr/include/r_bin.h`: Add metadata fields to RBinFile
- `libr/bin/bfile.c`: Initialize/cleanup metadata in file creation/destruction
- `libr/bin/dwarf.c`: Replace all sdb_addrinfo metadata reads/writes

### Phase 4: Remove sdb_addrinfo Struct Member
**Goal:** Remove the deprecated field from RBinFile

**Files to Modify:**
- `libr/include/r_bin.h:493`
  - Remove: `Sdb *sdb_addrinfo; // deprecate`
  - Remove: `void *addrinfo_priv; // future use`

### Phase 5: Remove Allocations and Deallocations

**libr/bin/bfile.c:**
1. Line 745: Remove `bf->sdb_addrinfo = sdb_new0();`
2. Line 1015: Remove `sdb_free(bf->sdb_addrinfo);`
3. Line 1435: Remove or verify `sdb_merge(dst->sdb_addrinfo, src->sdb_addrinfo);`

### Phase 6: Remove Namespace Registration

**libr/bin/bobj.c:**
1. Line 289: Remove `sdb_ns_set(bdb, "addrinfo", bf->sdb_addrinfo);`
2. Verify no code expects to query `bin.cur.addrinfo` from the SDB tree

### Phase 7: Remove Deprecated Callback Functions

**libr/core/cmd_meta.inc.c:**
1. Remove `print_addrinfo()` function (lines 338-372)
2. Remove `print_addrinfo_json()` function (lines 210-263)
3. Mark `print_addrinfo2()` and `print_addrinfo2_json()` as the canonical versions

### Phase 8: Remove Guard Conditions

**libr/bin/dwarf.c:**
1. Line 1266: Remove the `sdb_addrinfo` check
   - Change: `if (binfile && binfile->sdb_addrinfo && hdr->file_names)`
   - To: `if (binfile && hdr->file_names)`

2. Verify the `add_sdb_addrline()` function only calls the addrline API
   - Remove any sdb write attempts

### Phase 9: Testing

1. **Run existing tests:**
   ```bash
   r2r test/db/formats/dwarf
   r2r test/db/cmd/cmd_meta
   r2r test/db/cmd/cmd_print
   ```

2. **Verify backward compatibility:**
   - Ensure old binaries with no debug info still work
   - Ensure hybrid binaries (some from sdb, some from new API) work

3. **Manual testing:**
   ```bash
   # Load a binary with DWARF info
   r2 -A /path/to/binary
   
   # Query line info
   CL*            # List all line info
   CL 0x401000    # Query specific address
   
   # Verify source file listing
   ls~source      # Should show source files
   ```

---

## 5. Impact Analysis

### What Changes
- **Memory Usage**: Reduced (string pool deduplication is more efficient)
- **Query Performance**: O(1) address lookups vs O(n) for sdb
- **Code Complexity**: Reduced (fewer dual-path implementations)
- **Serialization**: No longer persisted to SDB (if needed, design new format)

### What Stays the Same
- **Public API**: `CL` command interface unchanged
- **Data**: Same address-to-line information is stored
- **Source Integration**: Source file access unchanged
- **Debug Info Loading**: DWARF/PDB parsing still works

### Potential Issues
1. **Persistence**: If code relied on SDB serialization of line info
   - **Fix**: Verify no persistence needed, or design new format
   
2. **External Tools**: If tools queried `bin.cur.addrinfo` from SDB
   - **Fix**: Document API migration, provide new query methods

3. **Plugin Compatibility**: If plugins directly accessed sdb_addrinfo
   - **Fix**: Update plugins to use public API

---

## 6. Deprecation Timeline

### Current (v5.10+)
- Both sdb_addrinfo and RBinAddrLineStore are populated
- New code prefers RBinAddrLineStore
- Old fallback code still functional

### Post-Migration (v5.11+)
- sdb_addrinfo removed entirely
- Only RBinAddrLineStore in use
- Cleaner, faster codebase

---

## 7. Implementation Checklist

- [ ] **Phase 1**: Remove commented legacy code (cmd_meta.inc.c lines 384-397)
- [ ] **Phase 2**: Remove sdb fallback in CL command (cmd_meta.inc.c, cmd_print_list.c, core_writedwarf.c)
- [ ] **Phase 3**: Migrate DWARF metadata to RBinFile fields
- [ ] **Phase 4**: Remove sdb_addrinfo from RBinFile struct
- [ ] **Phase 5**: Remove allocations/deallocations (bfile.c)
- [ ] **Phase 6**: Remove namespace registration (bobj.c)
- [ ] **Phase 7**: Remove deprecated callbacks (cmd_meta.inc.c)
- [ ] **Phase 8**: Remove guard conditions (dwarf.c)
- [ ] **Phase 9**: Run tests and verify no regressions
- [ ] **Code Review**: Ensure all SDB references removed
- [ ] **Documentation**: Update any API docs or migration guides

---

## 8. Files Summary

### Files to Modify
| File | Changes | Complexity |
|------|---------|------------|
| `libr/include/r_bin.h` | Remove sdb_addrinfo field | Low |
| `libr/bin/bfile.c` | Remove allocations/deallocations/merge | Low |
| `libr/bin/dwarf.c` | Migrate metadata storage, remove sdb reads | Medium |
| `libr/bin/bobj.c` | Remove namespace registration | Low |
| `libr/core/cmd_meta.inc.c` | Remove fallbacks and deprecated functions | Medium |
| `libr/core/cmd_print_list.c` | Use r_bin_addrline_files() instead of sdb | Low |
| `libr/core/p/core_writedwarf.c` | Remove fallback code | Low |

### Files to Verify (No Changes Expected)
| File | Reason |
|------|--------|
| `libr/bin/addrline.c` | Public API, should remain stable |
| `libr/core/cmd_search.inc.c` | Already uses correct API |

---

## 9. References

- **Header**: `libr/include/r_bin.h` (RBinFile, RBinAddrLineStore, RBinAddrline structs)
- **Implementation**: `libr/bin/addrline.c` (Public API functions)
- **Storage**: `libr/bin/bfile.c` (Internal AddrLineStore implementation)
- **DWARF Parser**: `libr/bin/dwarf.c` (Population of address-line data)
- **Commands**: `libr/core/cmd_meta.inc.c` (CL command implementation)

---

## 10. Success Criteria

✓ All `sdb_addrinfo` references removed from codebase
✓ No fallback code checking for sdb_addrinfo existence
✓ All tests pass (`r2r test/db/formats/dwarf`, `test/db/cmd/cmd_meta`, etc.)
✓ Binary loading and debug info display works correctly
✓ Source file extraction works via `r_bin_addrline_files()`
✓ Manual `CL` command works for querying and adding line info
✓ Code compiles with no warnings
✓ Performance is maintained or improved

