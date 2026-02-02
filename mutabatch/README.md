# RMutaBatch: Minimal Multi-Algorithm Processing API

## Overview

A minimal API for processing the same input data through multiple hashing/crypto algorithms in a single pass. This is required for rahash2.c to work without r_hash after r_hash deprecation.

The design philosophy: **Do one thing well** - just batch multiple sessions. Output retrieval and iteration logic stays in the caller (rahash2.c).

## Problem Statement

Currently in rahash2.c (lines 237-310), there's a loop that processes one input file through multiple algorithms:

```
for each algorithm in requested_algorithms:
    create/initialize context for algorithm
    read file in blocks
    process each block through algorithm
    apply iterations/stretching if needed
    print results
```

Without r_hash, this must use r_muta. But creating N independent sessions for N algorithms and feeding them the same data N times (once per loop iteration) is inefficient. We need a way to:
1. Create multiple sessions once
2. Feed the same data to all simultaneously
3. Use the sessions directly for output and iteration

## API Design (Minimal)

### Core Functions

```c
// Create a batch of sessions for multiple algorithms (CSV-separated names)
// Example: r_muta_batch_new(cry, "md5,sha256,crc32")
// Returns NULL if any algorithm not found/invalid
typedef struct r_muta_batch_t RMutaBatch;
R_API RMutaBatch *r_muta_batch_new(RMuta *muta, const char *algo_csv);

// Feed data to all sessions in the batch simultaneously
// Called once per block read - internally dispatches to all sessions
R_API bool r_muta_batch_update(RMutaBatch *batch, const ut8 *data, int len);

// Access the underlying sessions for output retrieval and iteration
// Caller iterates this list to handle per-algorithm operations
R_API RList *r_muta_batch_sessions(RMutaBatch *batch);

// Cleanup all sessions and batch context
R_API void r_muta_batch_free(RMutaBatch *batch);
```

### Header Changes

**File: libr/include/r_muta.h**

Add these declarations in the public API section:

```c
typedef struct r_muta_batch_t RMutaBatch;

R_API RMutaBatch *r_muta_batch_new(RMuta *muta, const char *algo_csv);
R_API bool r_muta_batch_update(RMutaBatch *batch, const ut8 *data, int len);
R_API RList *r_muta_batch_sessions(RMutaBatch *batch);
R_API void r_muta_batch_free(RMutaBatch *batch);
```

## Implementation (libr/muta/batch.c)

### Internal Structure

```c
struct r_muta_batch_t {
    RList *sessions;  // RMutaSession* items - one per algorithm
    RMuta *muta;      // Reference to parent (for potential future use)
};
```

### Function Behaviors

**r_muta_batch_new()**
- Parse `algo_csv` by splitting on commas
- Validate each algorithm name via `r_muta_find(muta, algo)`
- Create one `RMutaSession` per algorithm via `r_muta_use()`
- If ANY algorithm is invalid, free what was created and return NULL (fail-fast)
- Store sessions in an RList
- Return heap-allocated RMutaBatch

**r_muta_batch_update()**
- Iterate the session list
- Call `r_muta_session_update(session, data, len)` for each
- Return true if all succeeded, false if any failed
- No output handling - just feeding data

**r_muta_batch_sessions()**
- Simple accessor: return the RList pointer
- Caller owns the iteration - batch doesn't hide the sessions

**r_muta_batch_free()**
- Iterate sessions list and call `r_muta_session_free()` on each
- Free the RList itself
- Free the batch struct

## Iteration/Stretching: Handled in rahash2.c

### Why Move Iteration to rahash2?

The old `r_hash_do_spice()` feature (iterative hashing with seed prefix/suffix) is:
- Only used by rahash2 (via `-i` flag)
- Fundamentally a rahash2 workflow choice, not a generic crypto operation
- Can be implemented as a manual loop in rahash2

### How rahash2 Will Handle It

Current code (lines 180-255 in rahash2.c) does:
```c
r_hash_do_spice(ctx, hash, ro->iterations, ro->_s);
```

New approach:
```c
if (ro->iterations > 0) {
    // Manually apply iterations per session
    RListIter *iter;
    RMutaSession *session;
    r_list_foreach(batch->sessions, iter, session) {
        // Seed handling (prefix or suffix)
        if (ro->s.buf && ro->s.prefix) {
            r_muta_session_update(session, ro->s.buf, ro->s.len);
        }
        // Loop N-1 times (first hash already done in update loop)
        for (int i = 1; i < ro->iterations; i++) {
            ut8 *digest = r_muta_session_get_output(session, &len);
            r_muta_session_update(session, digest, len);
            free(digest);
        }
        if (ro->s.buf && !ro->s.prefix) {
            r_muta_session_update(session, ro->s.buf, ro->s.len);
        }
    }
}
```

**Caveat:** This recomputes the digest N-1 times for each session (via get_output). Not as efficient as the old r_hash approach, but correct and simple. If performance becomes critical, plugins can later add an optional `iterate` callback.

### RHashSeed Removal

The `RHashSeed` struct (from r_hash.h) is no longer needed:
- Seed data is already a buffer in rahash2 options (`ro->s.buf`, `ro->s.len`)
- Prefix flag is already in options (`ro->s.prefix`)
- rahash2 can handle the logic directly without a special struct

Delete from r_hash.h:
```c
typedef struct r_hash_seed_t {
    int prefix;
    ut8 *buf;
    int len;
} RHashSeed;
```

Remove r_hash_do_spice declaration (it becomes internal to libr/muta or doesn't exist).

## Changes to rahash2.c

### Replace Algorithm Loop (Lines 237-310)

**Before:**
```c
RHash *ctx = r_hash_new(true, algobit);
if (ro->incremental) {
    for (i = 1; i < R_HASH_ALL; i <<= 1) {
        if (algobit & i) {
            // ... per-algorithm setup ...
            r_hash_do_begin(ctx, i);
            // ... process blocks ...
            r_hash_do_end(ctx, i);
            r_hash_do_spice(ctx, i, ro->iterations, ro->_s);
            // ... print ...
        }
    }
}
```

**After:**
```c
// Build CSV of requested algorithms
char *algo_csv = build_algo_csv(algobit);  // Helper: convert bitmask to "md5,sha256,..."
RMutaBatch *batch = r_muta_batch_new(cry, algo_csv);
free(algo_csv);

if (!batch) {
    R_LOG_ERROR("Invalid algorithm in batch");
    return 1;
}

// Single pass through file with all algorithms
for (j = ro->from; j < ro->to; j += bsize) {
    int len = ((j + bsize) > ro->to) ? (ro->to - j) : bsize;
    r_io_pread_at(io, j, buf, len);
    r_muta_batch_update(batch, buf, len);
}

// Iteration/stretching
if (ro->iterations > 0) {
    // ... iteration logic as shown above ...
}

// Get outputs and print
RListIter *iter;
RMutaSession *session;
r_list_foreach(batch->sessions, iter, session) {
    int result_size = 0;
    ut8 *result = r_muta_session_get_output(session, &result_size);
    // ... existing print logic ...
    free(result);
}

r_muta_batch_free(batch);
```

### Helper Function: Build Algorithm CSV

rahash2 needs a helper to convert from the bitmask format used internally to CSV:

```c
// Convert R_HASH_* bitmask to comma-separated algorithm names
// Caller must free returned string
static char *build_algo_csv(ut64 algobit) {
    // Iterate through bit positions
    // Use r_muta_algo_name() (or similar) to get name for each set bit
    // Build "md5,sha256,crc32" etc.
    // Return allocated string
}
```

Note: This requires either:
1. A mapping function in r_muta (e.g., `r_muta_name_from_bit(ut64 bit)`)
2. Or a parallel mapping in rahash2 itself

### Remove r_hash Direct Usage

In rahash2.c, replace all:
- `r_hash_new()` → gone (use batch instead)
- `r_hash_free()` → gone (use r_muta_batch_free instead)
- `r_hash_calculate()` → replaced by batch update
- `r_hash_do_begin/end()` → gone (handled in sessions)
- `r_hash_do_spice()` → replaced by manual loop in rahash2
- `r_hash_name()` → use `r_muta_algo_name()` or existing metadata
- `r_hash_size()` → use session output length or metadata API

**Encryption/Decryption paths (lines 408-500)** - Already use r_muta, no changes needed.

## New r_muta API Additions (Besides Batch)

To support rahash2, we may need:

```c
// Get canonical/display name for an algorithm
// "md5" → "MD5", "sha256" → "SHA256", "crc32" → "CRC32"
R_API const char *r_muta_algo_name(const char *algo);

// Get size of output in bytes
// "sha256" → 32, "md5" → 16
R_API int r_muta_algo_size(const char *algo);
```

These are metadata helpers to replace r_hash_name() and r_hash_size() functionality.

## File Structure

**New file:**
- `libr/muta/batch.c` - Implementation of batch API

**Modified files:**
- `libr/include/r_muta.h` - Add batch declarations and new metadata functions
- `libr/muta/meson.build` or `Makefile` - Add batch.c to build
- `libr/main/rahash2.c` - Refactor to use batch API instead of r_hash
- `libr/include/r_hash.h` - Remove RHashSeed, r_hash_do_spice declaration

**Removed (from public API):**
- `r_hash.h` declarations for: `r_hash_do_begin`, `r_hash_do_end`, `r_hash_do_spice`, `RHashSeed` (they can stay internal if libr/muta needs them)

## Benefits

1. **Minimal API surface** - 4 functions, very focused
2. **No output management** - Caller uses standard session output APIs
3. **No iteration callbacks** - Iteration logic stays in rahash2 (where it's used)
4. **No r_hash dependency** - Pure r_muta
5. **Efficient data flow** - One read loop, all algorithms updated together
6. **Simple implementation** - Batch is just a container with one operation

## Trade-offs

1. **Session count overhead** - Still creates N sessions (memory cost unavoidable)
2. **Iteration inefficiency** - Manual loop in rahash2 re-fetches digest N times (acceptable for a CLI tool)
3. **Caller complexity** - rahash2 handles iteration logic (but it's isolated to one place)

## Testing Strategy

1. Test batch creation with valid/invalid algorithms
2. Test batch update with various data sizes
3. Test iteration/stretching behavior matches old r_hash behavior
4. Verify output correctness for all algorithms
5. Check that per-block hashing mode still works
6. Validate seed prefix/suffix handling

## Future Optimizations (Out of Scope)

If performance becomes critical:
- Add optional plugin callback for batch-aware iteration
- Cache digests across iterations instead of re-fetching
- Batch session end operations
- Special handling for algorithms that can work with the same context (future architecture change)
