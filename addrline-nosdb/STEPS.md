# STEPS.md: Sequential Non-Breaking Changes for Sdb AddrLine Deprecation

## Overview

This document provides a step-by-step migration plan to remove `sdb_addrinfo` dependency from radare2's addrline functionality. Each step is designed to be non-breaking and can be committed individually while maintaining system functionality.

## Phase 1: Cleanup Dead Code (Low Risk)

### Step 1.1: Remove Commented Legacy Code in CL Command
**File:** `libr/core/cmd_meta.inc.c`  
**Lines:** 384-397  
**Action:** Remove the `#else` block containing commented sdb_addrinfo code  
**Rationale:** This code is already disabled and cleanup reduces technical debt  

```c
// Remove this entire block (lines 384-397):
#else
    Sdb *s = bf->sdb_addrinfo;
    char aoffset[SDB_NUM_BUFSZ];
    // ... legacy sdb code ...
#endif
```

---

## Phase 2: Simplify Source File Extraction (Low Risk)

### Step 2.1: Replace sdb_foreach_list with addrline API
**File:** `libr/core/cmd_print_list.c`  
**Lines:** 52-69  
**Action:** Replace manual sdb iteration with `r_bin_addrline_files()`  
**Rationale:** New API is more efficient and simpler  

**Current:**
```c
// sdb_foreach_list(bf->sdb_addrinfo) with manual parsing
```

**New:**
```c
RList *files = r_bin_addrline_files (core->bin);
// Use the file list directly
```

---

## Phase 3: Remove Fallback Code Paths (Medium Risk)

### Step 3.1: Eliminate sdb Fallback in CL Query Command
**File:** `libr/core/cmd_meta.inc.c`  
**Lines:** 557-567  
**Action:** Remove sdb fallback blocks, require addrline API  

**Current Logic:**
```c
if (!r_bin_addrline_foreach(...)) {
    if (bf && bf->sdb_addrinfo) {
        sdb_foreach(...);  // Remove this fallback
    }
}
```

**New Logic:**
```c
r_bin_addrline_foreach(...);  // Always use new API
```

### Step 3.2: Remove sdb Fallback in DWARF Writing
**File:** `libr/core/p/core_writedwarf.c`  
**Lines:** 722-727  
**Action:** Remove legacy fallback block  

**Current:**
```c
if (addrline.used) {
    // Use new API
} else {
    // Fallback to sdb_addrinfo (REMOVE THIS)
}
```

**New:**
```c
// Always use addrline path, no fallback
```

---

## Phase 4: Migrate DWARF Compilation Directory Storage (Medium Risk)

### Step 4.1: Add Compilation Directory Fields to RBinFile
**File:** `libr/include/r_bin.h`  
**Location:** RBinFile struct (around line 493)  
**Action:** Add metadata fields before removing sdb_addrinfo  

```c
// Add these fields to RBinFile:
struct {
    char *comp_dir;           // Main compilation directory
    RHashTable *comp_dirs;    // Map of offset -> comp_dir for multiple CUs
} dwarf_metadata;
```

### Step 4.2: Initialize/Cleanup DWARF Metadata in RBinFile
**File:** `libr/bin/bfile.c`  
**Action:** Add initialization in file creation, cleanup in destruction  

```c
// In r_bin_file_new():
// bf->dwarf_metadata.comp_dir = NULL;
// bf->dwarf_metadata.comp_dirs = NULL;

// In r_bin_file_free():
// free(bf->dwarf_metadata.comp_dir);
// r_hashtable_free(bf->dwarf_metadata.comp_dirs);
```

### Step 4.3: Migrate DWARF Parser to Use New Metadata Storage
**File:** `libr/bin/dwarf.c`  
**Lines:** 701, 711, 713  
**Action:** Replace sdb_const_get/sdb_get calls with direct field access  

**Current:**
```c
const char *comp_dir = sdb_const_get (bf->sdb_addrinfo, k, 0);
include_dir_alloc = sdb_get (bf->sdb_addrinfo, comp_dir_key, 0);
include_dir_alloc = sdb_get (bf->sdb_addrinfo, "DW_AT_comp_dir", 0);
```

**New:**
```c
const char *comp_dir = bf->dwarf_metadata.comp_dir;
// Additional logic for comp_dirs lookup if needed
```

---

## Phase 5: Remove sdb_addrinfo Struct Member (Medium Risk)

### Step 5.1: Remove sdb_addrinfo from RBinFile Definition
**File:** `libr/include/r_bin.h`  
**Line:** 493  
**Action:** Remove the deprecated field and future use placeholder  

```c
// Remove these lines:
Sdb *sdb_addrinfo; // deprecate
void *addrinfo_priv; // future use to store abi-safe addrline info instead of k/v
```

---

## Phase 6: Remove Memory Management (Low Risk)

### Step 6.1: Remove sdb_addrinfo Allocation
**File:** `libr/bin/bfile.c`  
**Line:** 745  
**Action:** Remove initialization  

```c
// Remove this line:
bf->sdb_addrinfo = sdb_new0 ();
```

### Step 6.2: Remove sdb_addrinfo Deallocation
**File:** `libr/bin/bfile.c`  
**Line:** 1015  
**Action:** Remove cleanup  

```c
// Remove this line:
sdb_free(bf->sdb_addrinfo);
```

### Step 6.3: Remove sdb_addrinfo Merge
**File:** `libr/bin/bfile.c`  
**Line:** 1435  
**Action:** Remove merge operation (addrline merge already handled separately)  

```c
// Remove this line (addrline merge ishandled on lines 1437-1438):
sdb_merge(dst->sdb_addrinfo, src->sdb_addrinfo);
```

---

## Phase 7: Remove Namespace Registration (Low Risk)

### Step 7.1: Remove SDB Namespace Registration
**File:** `libr/bin/bobj.c`  
**Line:** 289  
**Action:** Remove namespace registration  

```c
// Remove this line:
sdb_ns_set(bdb, "addrinfo", bf->sdb_addrinfo);
```

### Step 7.2: Remove Commented Namespace Retrieval
**File:** `libr/bin/bobj.c`  
**Line:** 298  
**Action:** Remove commented fallback line  

```c
// Remove this commented line:
// bf->sdb_addrinfo = sdb_ns (bf->sdb, "addrinfo", 1);
```

---

## Phase 8: Remove Deprecated Callback Functions (Low Risk)

### Step 8.1: Remove Legacy Callback Functions
**File:** `libr/core/cmd_meta.inc.c`  
**Lines:** 338-372, 210-263  
**Action:** Remove deprecated functions marked as "R2_600 - DEPRECATE"  

**Remove:**
- `print_addrinfo()` function (lines 338-372)
- `print_addrinfo_json()` function (lines 210-263)

**Keep:**
- `print_addrinfo2()` and `print_addrinfo2_json()` (canonical versions)

---

## Phase 9: Remove Guard Conditions (Low Risk)

### Step 9.1: Remove sdb_addrinfo Existence Check in DWARF Parser
**File:** `libr/bin/dwarf.c`  
**Line:** 1266  
**Action:** Remove unnecessary guard condition  

**Current:**
```c
if (binfile && binfile->sdb_addrinfo && hdr->file_names) {
```

**New:**
```c
if (binfile && hdr->file_names) {
```

### Step 9.2: Clean up add_sdb_addrline Function
**File:** `libr/bin/dwarf.c`  
**Lines:** 1206-1234  
**Action:** Ensure function only calls addrline API, remove any sdb references  

---

## Phase 10: Update Function Parameters (Low Risk)

### Step 10.1: Remove sdb_addrinfo Parameter from parse_info_raw
**File:** `libr/bin/dwarf.c`  
**Line:** 2621  
**Action:** Update function signature and call  

**Current:**
```c
RBinDwarfDebugInfo *info = parse_info_raw (bf, bf->sdb_addrinfo, da, buf, section->bytes.len);
```

**New:**
```c
RBinDwarfDebugInfo *info = parse_info_raw (bf, da, buf, section->bytes.len);
```

**Also update:** Function signature declaration and implementation

---

## Phase 11: Testing and Validation (Critical)

### Step 11.1: Run Dwarf Tests
```bash
r2r test/db/formats/dwarf
```

### Step 11.2: Run Meta Command Tests
```bash
r2r test/db/cmd/cmd_meta
```

### Step 11.3: Run Print Command Tests
```bash
r2r test/db/cmd/cmd_print
```

### Step 11.4: Manual Verification
```bash
# Load binary with DWARF info
r2 -A /path/to/binary

# Test line info commands
CL*            # List all line info
CL 0x401000    # Query specific address
```

### Step 11.5: Verify Source File Listing
```bash
ls~source      # Should show source files correctly
```

---

## Implementation Order and Risk Assessment

| Phase | Risk Level | Dependencies | Can Be Committed Separately |
|-------|------------|--------------|----------------------------|
| 1     | Low        | None         | Yes                        |
| 2     | Low        | None         | Yes                        |
| 3     | Medium     | Phase 2      | Yes, after Phase 2         |
| 4     | Medium     | Phase 3      | Yes, after Phase 3         |
| 5     | Medium     | Phase 4      | Yes, after Phase 4         |
| 6     | Low        | Phase 5      | Yes, after Phase 5         |
| 7     | Low        | Phase 6      | Yes, after Phase 6         |
| 8     | Low        | Phase 7      | Yes, after Phase 7         |
| 9     | Low        | Phase 8      | Yes, after Phase 8         |
| 10    | Low        | Phase 9      | Yes, after Phase 9         |
| 11    | Critical   | All phases   | Final validation           |

---

## Rollback Strategy

Each phase can be independently rolled back:

1. **Phases 1-2, 6-10**: Simple to revert as they only remove/comment code
2. **Phase 3**: Requires enabling fallback code paths if reverted
3. **Phase 4**: Requires reverting DWARF metadata storage back to sdb
4. **Phase 5**: Requires re-adding sdb_addrinfo struct member
5. **Phase 11**: Validation phase - no code changes to rollback

---

## Success Criteria

✓ All `sdb_addrinfo` references removed from codebase  
✓ No fallback code checking for sdb_addrinfo existence  
✓ All tests pass without regressions  
✓ Binary loading and debug info display works correctly  
✓ Source file extraction works via new API  
✓ Manual CL commands work for querying and adding line info  
✓ Code compiles with no warnings  
✓ Performance is maintained or improved  

---

## Notes for Implementers

1. **Commit Message Guidelines**: Each phase should have a clear commit message explaining the specific change and its rationale.

2. **Testing Between Phases**: Run basic tests after each phase to ensure no regressions before proceeding.

3. **Documentation Updates**: Update relevant documentation after completing all phases.

4. **Plugin Compatibility**: Verify no external plugins depend on sdb_addrinfo access.

5. **Performance Monitoring**: Monitor memory usage and query performance improvements.

---

## Estimated Timeline

- **Phases 1-2**: 1 day (cleanup and low-risk changes)
- **Phases 3-5**: 2-3 days (medium-risk structural changes)
- **Phases 6-10**: 1-2 days (cleanup and finalization)
- **Phase 11**: 1 day (testing and validation)
- **Total**: 5-7 days for complete migration

This timeline assumes focused work with proper testing at each stage.