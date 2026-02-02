# RMutaBatch Refactoring Implementation Plan

## Overview
Implement a batch processing API for r_muta to enable efficient multi-algorithm hashing without r_hash dependency. The batch API processes the same input data through multiple algorithms in a single pass.

## Core Design Principles
1. **Minimal API** - 4 core functions only
2. **Caller-owned iteration** - No hidden output management
3. **No special callbacks** - Iteration logic stays in rahash2.c
4. **Pure r_muta** - Zero r_hash dependencies in batch code
5. **Radare2 style** - Follow AGENTS.md (tabs, spaces in comments, function space before parenthesis)

---

## Files to Create

### 1. `libr/muta/muta_batch.c`
**Purpose**: Implementation of batch processing API

**Structure**:
```c
struct r_muta_batch_t {
    RList *sessions;      // RMutaSession* items
    RMuta *muta;          // Reference to parent (for future use)
};
```

**Functions to implement**:

#### `r_muta_batch_new(RMuta *muta, const char *algo_csv)`
- Parse CSV string by splitting on commas, trimming whitespace
- For each algorithm name:
  - Call `r_muta_find(muta, algo)` to validate
  - If not found, free all created sessions and return NULL (fail-fast)
  - Call `r_muta_use(muta, algo)` to create session
- Store all sessions in RList
- Return heap-allocated RMutaBatch or NULL on failure

#### `r_muta_batch_update(RMutaBatch *batch, const ut8 *data, int len)`
- Iterate session list
- Call `r_muta_session_update(session, data, len)` for each session
- Return true if all succeeded, false on first failure
- No output management - pure data feeding

#### `r_muta_batch_sessions(RMutaBatch *batch)`
- Simple accessor: return the RList pointer
- Allows caller to iterate sessions for output retrieval

#### `r_muta_batch_free(RMutaBatch *batch)`
- Iterate sessions list, call `r_muta_session_free()` on each
- Free RList via `r_list_free()`
- Free batch struct
- Handle NULL input gracefully

---

## Files to Modify

### 2. `libr/include/r_muta.h`
**Changes**:

**Add after line 99 (after RMutaOptions)**:
```c
typedef struct r_muta_batch_t RMutaBatch;
```

**Add after line 135 (after r_muta_process declaration)**:
```c
// Batch processing API - process multiple algorithms in single pass
R_API RMutaBatch *r_muta_batch_new(RMuta *muta, const char *algo_csv);
R_API bool r_muta_batch_update(RMutaBatch *batch, const ut8 *data, int len);
R_API RList *r_muta_batch_sessions(RMutaBatch *batch);
R_API void r_muta_batch_free(RMutaBatch *batch);

// Metadata helpers for algorithm info
R_API const char *r_muta_algo_name(const char *algo);
R_API int r_muta_algo_size(const char *algo);
```

---

### 3. `libr/muta/meson.build`
**Changes**:

**Add 'muta_batch.c' to r_muta_sources list** (after 'muta_session.c'):
```meson
r_muta_sources = files(
  'charset.c',
  'muta.c',
  'muta_batch.c',      # <-- ADD THIS LINE
  'muta_bind.c',
  ...
)
```

---

### 4. `libr/muta/muta.c`
**Add after `r_muta_process()` function** (around line 70):

Implement two metadata helper functions:

#### `r_muta_algo_name(const char *algo)`
- Map algorithm name to display/canonical name
- Examples: "md5" → "MD5", "sha256" → "SHA256"
- Return const char* for canonical name, fallback to algo if unknown
- Handle both muta plugins and legacy r_hash algorithms

#### `r_muta_algo_size(const char *algo)`
- Return digest/output size in bytes for algorithm
- Examples: "sha256" → 32, "md5" → 16
- Return -1 if algorithm not found
- Query plugin metadata if available, else check r_hash_name for legacy algorithms

**Implementation approach**:
- Iterate muta->plugins to find matching plugin
- If found, check plugin->meta for size info, or use known defaults
- If not found, loop through r_hash bits to check legacy algorithms
- Use `r_hash_name()` to map bitmask → name and determine size

---

## Files NOT to Modify (Design Decision)

### r_hash.h / r_hash.c
- **Decision**: Keep r_hash internal to r_muta for now
- RHashSeed and r_hash_do_spice remain in r_hash.c only
- Legacy algorithm support stays via `r_hash_name()` in muta.c metadata functions
- Full r_hash deprecation is out-of-scope for this refactor

### rahash2.c
- **Decision**: NOT modified in this phase
- Refactor of rahash2.c to use batch API is separate task
- Batch API is standalone and usable immediately by any caller

---

## Code Style Requirements

**From AGENTS.md**:
- Indent with **TABS** for code, spaces for comment alignment
- Space before function parenthesis: `r_muta_batch_new ()`
- Always use `{}` braces, even for single-line conditionals
- Prefer `!strcmp ()` over `strcmp () == 0`
- Use `R_RETURN_*` macros in public functions
- Define and assign variables in same line when possible
- Use `r_str_newf`, `r_strbuf_new`, never `r_str_append`

**Example pattern from existing code**:
```c
R_API RMutaBatch *r_muta_batch_new(RMuta *muta, const char *algo_csv) {
	R_RETURN_VAL_IF_FAIL (muta && algo_csv, NULL);
	RMutaBatch *batch = R_NEW0 (RMutaBatch);
	batch->muta = muta;
	batch->sessions = r_list_newf (free);
	// ... implementation ...
	return batch;
}
```

---

## Implementation Order

1. **Create muta_batch.c** with 4 functions
2. **Update r_muta.h** with function declarations and RMutaBatch typedef
3. **Update meson.build** to add muta_batch.c to build
4. **Implement metadata helpers** in muta.c (r_muta_algo_name, r_muta_algo_size)
5. **Compile and verify** with `make -C libr/muta`
6. **Test** batch creation with valid/invalid algorithms

---

## Testing Strategy (Out of Scope for This Plan)

Test cases to verify:
- Batch creation with valid comma-separated algorithms
- Batch creation with invalid/unknown algorithms (should return NULL)
- Batch update with various data sizes
- Session iteration to retrieve results
- Memory cleanup on batch free
- Multiple batches in same process

---

## Trade-offs and Decisions

### CSV Parsing vs Array Interface
**Decision**: CSV string input (simpler for CLI tools)
- Easier to convert from bitmask in rahash2
- More flexible than fixed array size
- Can add array version later if needed

### No Built-in Iteration
**Decision**: Caller owns session iteration
- Batch is simple container with one responsibility
- Caller can order, filter, or process sessions as needed
- Matches "Do one thing well" philosophy
- rahash2.c maintains full control of output/iteration logic

### Metadata Functions in muta.c
**Decision**: Not in batch.c, shared utility
- Other code may need algo name/size info
- Avoids duplication
- Centralizes algorithm metadata logic

### No Plugin Callbacks for Iteration
**Decision**: Out of scope, manual loop in caller
- Acceptable for CLI tools (not hot path)
- Keeps batch API small and focused
- Can add plugin callbacks later if needed

---

## Success Criteria

✓ Code compiles without errors or warnings  
✓ Batch API accessible via r_muta.h  
✓ All 4 batch functions work correctly  
✓ Metadata helpers return correct values  
✓ Memory cleanup verified (no leaks)  
✓ Follows radare2 coding style (clang-format-radare2 passes)  
✓ No r_hash dependencies in batch.c  
✓ Radare2 codebase builds successfully  

---

## Future Work (Out of Scope)

- Refactor rahash2.c to use batch API
- Remove r_hash public API (keep internal use)
- Cache digests for iteration optimization
- Add plugin-level iteration callbacks
- Support for session-end/finalization in batch
