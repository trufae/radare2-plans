# Graph Traversal Analysis Reimplementation Report (Correctness + Performance + LOC Reduction)

## Scope and method

This report is based on a direct code audit of the current traversal stack in:

- `libr/util/graph.c` (`RGraph` traversal + dominator/post-dominator)
- `libr/anal/block.c` (CFG successor iteration and shortest-path traversal)
- `libr/anal/p/anal_path.c` (second shortest-path implementation)
- `libr/anal/function.c` (function CFG materialization)
- `libr/core/p/core_agD.c` (dominance visualization entrypoint)

I could not fetch the external historical report URL from this environment (HTTP tunnel denied), so this document is intentionally self-contained.

---

## Executive summary

The current traversal behavior is incorrect or unstable for non-trivial CFGs because the implementation mixes:

1. **A non-standard dominator algorithm** in `r_graph_dom_tree`.
2. **In-place graph mutation tricks** for post-dominators (`_invert_edges`) that only partially invert node state.
3. **Two divergent BFS/path engines** (`block.c` and `anal_path.c`) with different edge semantics and different early-stop behavior.
4. **List-based adjacency + duplicate edges** with no canonical edge model, causing order-dependent traversal and poor asymptotics.

The fastest path to correctness and lower LOC is to **unify traversal into one small generic engine** and replace dominance with a standard algorithm (Lengauer-Tarjan or iterative bitset solver) over a compact index-based graph.

---

## Code map (where bugs and drift originate)

### 1) Graph core (`libr/util/graph.c`)

- DFS uses `RStack` of heap-allocated `RGraphEdge` objects for each push.
- Graph nodes keep three lists (`out_nodes`, `in_nodes`, `all_neighbours`) that must stay synchronized manually.
- Dominator tree is built with custom heuristics over DFS-inserted graph copy (`r_graph_dom_tree`).
- Post-dominator (`r_graph_pdom_tree`) inverts edges by swapping `in_nodes`/`out_nodes` pointers with XOR swaps.

### 2) Analysis traversal (`libr/anal/block.c`)

- `r_anal_block_successor_addrs_foreach` enumerates `jump`, `fail`, and switch-case jumps.
- `r_anal_block_shortest_path` performs BFS over successors plus refs (CODE/CALL/DATA).
- DFS variant (`r_anal_block_recurse_depth_first`) has dedicated logic and branch ordering semantics.

### 3) Plugin traversal (`libr/anal/p/anal_path.c`)

- Reimplements shortest-path BFS again.
- Traverses successors plus CALL refs only.
- Near duplicate of `block.c` path logic, but behavior is not identical.

### 4) CFG builder (`libr/anal/function.c`)

- `r_anal_function_get_graph` builds `RGraph` from BB list and links jump/fail/switch edges.
- Any missing successor node returns "Broken fcn" and aborts graph creation.

---

## Deep problem list

## A. Dominator and post-dominator correctness risks

### A1) Dominator implementation is heuristic, not a known correct algorithm

`r_graph_dom_tree` reconstructs a DFS tree and then patches parents for multi-in nodes (`mi`) using min/max DFS index heuristics. This is not equivalent to Lengauer-Tarjan, Cooper-Harvey-Kennedy, or classical iterative dataflow; therefore correctness is graph-shape dependent.

Impact:

- Incorrect immediate dominators on irreducible CFGs, dense switch CFGs, and certain cross-edge patterns.
- Order-dependent output (depends on list insertion order and DFS path).

### A2) Post-dominator builds by in-place graph inversion with partial state updates

`r_graph_pdom_tree` uses `_invert_edges(graph)` before/after calling dominator. `_invert_edges` swaps only `in_nodes` and `out_nodes` pointers per node; `all_neighbours` is not recomputed and graph-level edge metadata stays untouched.

Impact:

- Any logic touching `all_neighbours` during this window observes stale semantics.
- Shared graph object mutation makes reasoning fragile and non-thread-safe.
- Easy to break invariants if new traversal APIs are added.

### A3) Entry/root assumptions are implicit and under-validated

Dominance is called with a caller-provided `root`, but there is no explicit validation for reachability coverage, unique entry semantics, synthetic exit handling for post-dominance, or multi-exit normalization.

Impact:

- Inconsistent results across functions with multiple returns/unreachable blocks.
- Subtle mismatch between expected dominance theory and produced tree.

---

## B. BFS/path correctness drift (duplicated engines)

### B1) Two shortest-path implementations diverged semantically

- `libr/anal/block.c:r_anal_block_shortest_path` follows CFG successors + CODE/CALL/DATA refs.
- `libr/anal/p/anal_path.c:shortest_path_blocks` follows CFG successors + CALL refs only.

Impact:

- Same conceptual command can produce different paths depending on API path/plugin path.
- Traversal “correctness” is ambiguous because edge policy is duplicated and inconsistent.

### B2) Early-stop callback result is ignored in one path

`r_anal_block_shortest_path` calls `r_anal_block_successor_addrs_foreach(cur, shortest_path_successor_cb, &ctx)` without checking the returned bool. The callback can request stop when destination is found, but loop continues.

Impact:

- Wasted work (performance).
- Parent map can be overwritten less predictably in future edits (correctness fragility).

### B3) Address-to-block normalization is duplicated and minimal

Both implementations use a local `bb_addr_for` helper that picks the first block containing an address. This policy is duplicated and implicit.

Impact:

- Hidden ambiguity when overlapping/multiple blocks exist.
- Different modules may diverge further on future edits.

---

## C. Data-structure choices causing both bugs and cost

### C1) List-based adjacency everywhere (O(deg) membership, O(E) deletions)

`RGraph` uses linked lists for adjacency and uses repeated `r_list_contains` / `r_list_delete_data`.

Impact:

- High constant factors and poor cache locality on large CFGs.
- Duplicate-edge handling is implicit (not prevented), which distorts traversal classification and dominance inputs.

### C2) Three adjacency containers per node (`out`, `in`, `all`)

Each edge mutation manually updates multiple lists.

Impact:

- Invariant drift risk (a single missed update corrupts traversal semantics).
- Extra LOC and maintenance burden.

### C3) DFS allocates an edge object per stack push

`dfs_node` heap-allocates `RGraphEdge` for traversal stack events.

Impact:

- Allocation-heavy traversal path.
- Preventable overhead and fragmentation pressure.

---

## D. Failure behavior and diagnosability

### D1) "Broken fcn" in graph build is all-or-nothing

`r_anal_function_get_graph` aborts if one successor target node is missing.

Impact:

- Consumers cannot inspect partial graph.
- Hard to diagnose malformed CFG roots vs sparse analysis states.

### D2) No dedicated dominator/post-dominator unit tests

Current unit coverage exercises generic graph primitives (`test/unit/test_graph.c`) but does not pin dominance correctness on canonical CFG fixtures.

Impact:

- Regressions in dominance logic can pass CI silently.

---

## Reimplementation blueprint (small core, higher confidence)

## 1) Introduce a compact traversal kernel (single source of truth)

Create one internal traversal module (e.g. `libr/anal/traversal.c`) with:

- index-based nodes (`0..N-1`)
- adjacency vectors
- optional reverse adjacency
- policy flags: follow_jump, follow_fail, follow_switch, follow_refs(mask)

Then route both `r_anal_block_shortest_path` and `anal_path` through it.

Result: remove duplicated BFS logic and guarantee identical semantics by configuration.

## 2) Replace dominator with standard algorithm

Pick one:

- **Lengauer-Tarjan** (fast, classic)
- or **iterative bitset dominance** (very small code, acceptable for moderate N)

Given the objective (small code + performance), a compact Lengauer-Tarjan implementation over integer indices is the best fit.

## 3) Stop mutating graphs for pdom

Compute post-dominators by:

- building reverse CFG view (or passing reverse edge accessor)
- using synthetic exit if multi-exit
- running same dominator engine on reverse graph

No in-place pointer swaps.

## 4) Remove `all_neighbours` from algorithmic paths

Keep it only for legacy UI if needed; algorithmic code should use explicit out/in adjacency generated from one canonical edge list.

## 5) Consolidate edge policy in one enum

Define one public/internal enum for traversal edge classes:

- CFG_JUMP
- CFG_FAIL
- CFG_SWITCH
- REF_CODE
- REF_CALL
- REF_DATA

APIs and commands should pass the desired mask explicitly.

---

## LOC-reduction opportunities (high-value)

1. **Merge duplicated BFS implementations** (`block.c` + `anal_path.c`) into one engine.
2. **Remove local duplicated helpers** (`bb_addr_for`, callback glue variants).
3. **Delete `_invert_edges` trick** and associated cognitive load.
4. **Share one predecessor-map reconstruction helper** for all path APIs.
5. **Unify traversal order contract** (document once, test once).

Expected outcome: less traversal code, fewer corner-case deltas, easier performance tuning.

---

## Suggested phased plan

### Phase 0: lock behavior with tests

- Add CFG fixture-based tests for:
  - reducible branch/loop CFG
  - irreducible CFG
  - switch-heavy CFG
  - multi-exit CFG (pdom)
- Add path tests with configurable edge mask.

### Phase 1: unify shortest path

- Implement one BFS routine + edge iterator callback.
- Make both API/plugin call into it.
- Keep old wrappers for compatibility.

### Phase 2: replace dominance

- Add new dominator implementation behind internal feature switch.
- Run both old/new in debug mode and diff outputs on tests.
- Remove old heuristic logic after parity on expected fixtures.

### Phase 3: simplify graph infra

- Minimize adjacency containers touched by traversal algorithms.
- Avoid per-edge heap allocations in DFS hot path.

---

## Performance notes (practical)

- Index-based arrays + vectors will significantly outperform linked-list adjacency in analysis workloads.
- Dominator runtime should become predictable and less sensitive to node insertion order.
- Unified traversal will reduce instruction footprint and branch complexity while improving cache behavior.

---

## Acceptance criteria for "fixed"

1. Dominator/pdom output matches expected trees for canonical CFG fixtures.
2. `a:path` plugin and block shortest-path APIs return identical results for same edge mask.
3. No in-place graph mutation to compute reversed traversals.
4. Traversal LOC in analysis path is lower than current combined duplicated code.
5. Benchmarks on large functions show reduced time and allocation count.

---

## One-line commit message proposal

Unify and harden graph traversal analysis design for correctness and speed ##analysis

