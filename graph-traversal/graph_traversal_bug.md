# Graph Traversal Bug in ESIL Analysis

**Date:** 2026-03-04  
**Files Involved:** `libr/core/canal.c`, `libr/core/cmd_anal.inc.c`  
**Functions:** `get_next_i()`, `r_core_anal_esil()`, `cmd_aaef()`

## Executive Summary

The ESIL analysis iteration in `r_core_anal_esil()` uses a graph traversal algorithm (`get_next_i()`) that fails to visit all basic blocks in functions with complex control flow graphs. This causes missed xrefs, incomplete type propagation, and inconsistent analysis results between `aae` and `aaef`.

## Reproducer

```bash
# Download test binary (Apple kernel)
# Expected: xref to string at 0xfffffff00708b9fd from instruction at 0xfffffff0085e513c

# This FAILS (xref not found):
r2 -Qc 'aaa; axtj 0xfffffff00708b9fd' tmp/com.apple.kernel
# Returns: []

# This WORKS (with workaround):
r2 -Qc 'aaa; aae; axtj 0xfffffff00708b9fd' tmp/com.apple.kernel  
# Returns: [{"from":18446744005130473788,"type":"STRN",...}]
```

The instruction at `0xfffffff0085e513c` is:
```
0xfffffff0085e5138      adrp x0, 0xfffffff00708b000
0xfffffff0085e513c      add x0, x0, 0x9fd    ; constructs 0xfffffff00708b9fd
0xfffffff0085e5140      bl fcn.fffffff00890f0f4
```

This `adrp` + `add` pattern constructs the string address, but the xref is never created because the ESIL analysis never reaches this instruction.

## Root Cause Analysis

### The `get_next_i()` Algorithm

Location: `libr/core/canal.c:5670-5760`

The algorithm attempts to traverse the function's CFG following control flow:

```c
static bool get_next_i(IterCtx *ctx, size_t *next_i) {
    (*next_i)++;
    ut64 cur_addr = *next_i + ctx->start_addr;
    if (ctx->fcn) {
        // Initialize on first call
        if (!ctx->cur_bb) {
            ctx->bbl = r_list_clone (ctx->fcn->bbs, NULL);
            ctx->cur_bb = r_anal_get_block_at (ctx->fcn->anal, ctx->fcn->addr);
            r_list_push (ctx->path, ctx->cur_bb);
        }
        RAnalBlock *bb = ctx->cur_bb;
        if (cur_addr >= bb->addr + bb->size) {
            // Try to find next block via bb->jump
            bbit = r_list_find (ctx->bbl, &bb->jump, ...);
            if (!bbit && bb->fail != UT64_MAX) {
                // Try bb->fail
                bbit = r_list_find (ctx->bbl, &bb->fail, ...);
            }
            if (!bbit) {
                // Backtrack through path trying prev_bb->fail
                do {
                    prev_bb = r_list_pop (ctx->path);
                    bbit = r_list_find (ctx->bbl, &prev_bb->fail, ...);
                } while (!bbit && !r_list_empty (ctx->path));
            }
            // ... move to found block or return false
        }
    }
}
```

### Bug #1: Incomplete DFS Traversal

The algorithm uses depth-first search following `jump` then `fail` edges:

1. Start at function entry block
2. When block ends, follow `bb->jump` to find next block
3. If not found, try `bb->fail`
4. If still not found, backtrack and try `prev_bb->fail`

**Problem:** This misses blocks that are only reachable via `jump` from blocks that were already visited. The backtracking only tries `fail` paths, never revisiting `jump` alternatives.

**Example from Apple kernel:**

```
Function: sym.syscall.315.aio_suspend (0xfffffff0085e4d94)
Block 0x5110: size=16, jump=0x50a8 (backward loop)
Block 0x5120: size=100, contains target instruction 0x513c
```

The traversal:
1. Reaches block `0x5110`
2. Follows `jump` to `0x50a8` (backward jump, creates a loop)
3. Block `0x5120` is NEVER visited because:
   - It's not in the `jump`/`fail` chain from the current traversal path
   - Backtracking only tries `fail`, never `jump`

### Bug #2: Basic Block Sharing Between Functions

Debug output revealed:
```
DEBUG SWITCH: from bb=0xfffffff0085e5144 to bb=0xfffffff0085e5184 next_i=64
DEBUG SWITCH: from bb=0xfffffff0085e510c to bb=0xfffffff0085e5110 next_i=1172
DEBUG END: after bb=0xfffffff0085e5110 no more blocks
```

The blocks `0x510c` and `0x5110` are being analyzed as part of function `fcn.fffffff0085e5144` (starting at `0x5144`), NOT `sym.syscall.315.aio_suspend` (starting at `0x4d94`).

**Problem:** Multiple functions can have basic blocks in overlapping address ranges. When `IterCtx.fcn` is set, the iteration only considers blocks in that specific function's `fcn->bbs` list, missing blocks that belong to other functions but fall within the address range.

### Bug #3: Missing ESIL Memory Initialization

The original broken code in commit `4ab9a1c5b5` changed `aaef` from:
```c
r_core_cmd0 (core, "aeim");  // Initialize ESIL memory
// ... iterate with afla order ...
r_core_anal_esil (core, "f", NULL);
```

To:
```c
r_core_cmd0 (core, "aef@@@F");  // No aeim!
```

Without `aeim`, the ESIL virtual memory is not properly initialized, leading to incorrect emulation results.

### Bug #4: Function List Mutation During Analysis

The original `aef@@@F` approach iterates directly over `core->anal->fcns`. However, ESIL emulation can discover and create new functions during analysis, mutating the list and potentially invalidating iterators.

The `afla` approach captures function addresses BEFORE analysis begins, avoiding this issue.

## Function Address Range vs Basic Block List

When `r_core_anal_esil(core, "f", NULL)` is called:

```c
RAnalFunction *fcn = r_anal_get_fcn_in (core->anal, core->addr, 0);
if (fcn) {
    start = fcn->addr;
    end = r_anal_function_max_addr (fcn);
}
```

The address range `[start, end]` may contain:
1. Gaps (unreachable code, data, padding)
2. Blocks from OTHER functions (due to tail calls, shared code)
3. Blocks that ARE in `fcn->bbs` but not reachable via DFS from entry

When `IterCtx.fcn` is set, the iteration ONLY visits blocks in `fcn->bbs`, but this list may be incomplete or the traversal algorithm may fail to visit all blocks.

## Why Linear Iteration Works

Setting `IterCtx.fcn = NULL` forces the simple path:

```c
} else if (cur_addr >= ctx->end_addr) {
    return false;
}
```

This just increments `next_i` until we reach `end_addr`, analyzing EVERY instruction in the range regardless of control flow. This is slower but correct.

## Potential Proper Fixes

### Option 1: Fix the DFS Algorithm

Modify `get_next_i()` to use proper DFS that tracks visited blocks and explores ALL edges:

```c
// Pseudo-code
if (!ctx->visited) {
    ctx->visited = ht_uu_new0 ();
    ctx->stack = r_list_new ();
    r_list_push (ctx->stack, entry_block);
}

while (!r_list_empty (ctx->stack)) {
    RAnalBlock *bb = r_list_pop (ctx->stack);
    if (ht_uu_find (ctx->visited, bb->addr, NULL)) {
        continue;  // Already visited
    }
    ht_uu_insert (ctx->visited, bb->addr, 1);
    
    // Add successors to stack (both jump AND fail)
    if (bb->jump != UT64_MAX) {
        push_if_in_function (ctx->stack, bb->jump);
    }
    if (bb->fail != UT64_MAX) {
        push_if_in_function (ctx->stack, bb->fail);
    }
    // Handle switch cases too
    
    // Analyze this block
    *next_i = bb->addr - ctx->start_addr;
    return true;
}
return false;  // No more blocks
```

### Option 2: Sort Basic Blocks by Address

Instead of following CFG edges, sort all basic blocks by address and iterate:

```c
if (!ctx->sorted_bbs) {
    ctx->sorted_bbs = r_list_clone (ctx->fcn->bbs, NULL);
    r_list_sort (ctx->sorted_bbs, cmp_bb_addr);
    ctx->bb_iter = r_list_head (ctx->sorted_bbs);
}

// When current block ends, move to next in sorted order
if (cur_addr >= bb->addr + bb->size) {
    ctx->bb_iter = ctx->bb_iter->n;
    if (!ctx->bb_iter) return false;
    ctx->cur_bb = ctx->bb_iter->data;
    *next_i = ctx->cur_bb->addr - ctx->start_addr;
}
```

**Caveat:** This still only visits blocks in `fcn->bbs`, which may be incomplete.

### Option 3: Hybrid Approach

1. Use sorted basic block iteration for the function
2. Also do linear scan of gaps between blocks
3. Handle overlapping functions properly

### Option 4: Improve Basic Block Discovery

The root issue may be that `fcn->bbs` is incomplete. Improving the initial analysis (`af`, `aa`) to discover all reachable blocks would help.

## What's Missing

1. **Complete CFG reconstruction**: Some basic blocks may not be discovered during initial analysis due to indirect jumps, computed gotos, or obfuscation.

2. **Shared block handling**: When functions share basic blocks (tail call optimization, code deduplication), the analysis should handle this properly.

3. **Register state management**: The original algorithm used `r_reg_arena_push/pop` to track register state across blocks, but this was tied to the broken traversal. A proper fix needs to maintain this for accurate emulation.

4. **Performance**: Linear iteration is O(n) where n is the address range. For sparse functions with few blocks spread across large ranges, this is wasteful.

## What's Left to Do

1. **Root cause the incomplete `fcn->bbs`**: Investigate why block `0x5120` exists but the traversal doesn't find it. Is it in the function's bbs list but unreachable via jump/fail? Or is the bbs list incomplete?

2. **Add test cases**: Create unit tests for the graph traversal with known complex CFGs.

3. **Profile performance**: Compare linear vs sorted block vs fixed DFS approaches on various binaries.

4. **Handle overlapping functions**: Decide policy for blocks that belong to multiple functions.

5. **Register state preservation**: If using block-order iteration instead of CFG traversal, determine if register state tracking is still needed/useful.

## Debug Commands Used

```bash
# Check function info
r2 -Qc 'aa; afl~aio_suspend' tmp/com.apple.kernel

# Check basic blocks of a function
r2 -Qc 'aa; s sym.syscall.315.aio_suspend; afbq' tmp/com.apple.kernel

# Check specific basic block
r2 -Qc 'aa; s 0xfffffff0085e5120; abi' tmp/com.apple.kernel

# Check what jumps to a block
r2 -Qc 'aa; s sym.syscall.315.aio_suspend; afbj' tmp/com.apple.kernel | grep '"jump"'

# Check function sizes
r2 -Qc 'aa; s sym.syscall.315.aio_suspend; afi~size' tmp/com.apple.kernel
```

## Current Workaround

The current fix uses:
1. `aeim` for ESIL memory initialization
2. `afla` to get function addresses in dependency order before analysis
3. `IterCtx.fcn = NULL` to force linear iteration

This is correct but potentially slower than a properly fixed graph traversal.

## Related Files

- `libr/core/canal.c`: `r_core_anal_esil()`, `get_next_i()`, `IterCtx`
- `libr/core/cmd_anal.inc.c`: `cmd_aaef()`, `cmd_anal_all()`
- `libr/anal/fcn.c`: Basic block discovery
- `libr/anal/bb.c`: Basic block management

## Commits Reference

- `4ab9a1c5b5`: Original breaking change (replaced afla loop with aef@@@F)
- `34d9a31281^`: Last known working state before regression
