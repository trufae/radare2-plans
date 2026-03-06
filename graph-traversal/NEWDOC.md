# Correct Graph Traversal and Path-Sensitive Emulation Plan

**Date:** 2026-03-06

## Goal

Define a correct algorithm for whole-program ESIL-based emulation that:

- follows all feasible execution paths,
- keeps register and memory state per path and per basic block,
- handles shared basic blocks and overlapping function ownership,
- reaches a stable fixpoint instead of depending on block order,
- resolves code/data references and feeds type propagation with correct values,
- removes the need for workaround-style behavior such as the one introduced in PR `#25496`.

This document focuses on correctness first. Performance matters, but only after the traversal and state model are formally correct.

## What is broken today

From the existing notes in `gt/graph_traversal_bug.md`, `gt/newreport.md`, and the current code in `libr/core/canal.c` and `libr/anal/p/anal_tp.c`, the current analysis suffers from structural issues:

1. `get_next_i()` in `r_core_anal_esil()` is not a complete CFG traversal.
   - It mixes linear instruction iteration with ad-hoc DFS-like block switching.
   - It prefers `jump`, then `fail`, and backtracks mostly through `fail`.
   - It can miss blocks reachable through alternate paths, loops, or switch edges.

2. Register state is coupled to traversal order.
   - `r_reg_arena_push/pop` is used as if the walk order itself were the state model.
   - If a block is reached from another predecessor later, the previous emulation order may have already destroyed the correct incoming state.

3. Basic block ownership is treated as function-local when it is not.
   - Shared blocks, overlapping functions, tail-merged code, noreturn heuristics, split functions, and delayed discovery all violate the assumption that `fcn->bbs` is a complete and unique universe.
   - A block may be semantically reachable from multiple functions or contexts.

4. Type propagation in `libr/anal/p/anal_tp.c` currently sorts blocks by address.
   - The code itself already contains a TODO saying the algorithm would be more accurate if blocks were followed by control flow instead of address order.
   - Address order is deterministic, but not semantically correct.

5. The current flow is not path-sensitive enough.
   - Branch conditions, switch cases, loop-carried values, and call-return effects need separate incoming states until they can be safely merged.
   - A single flat register arena per function is not enough.

6. Sparse-function fallback is still a workaround.
   - Emulating each basic block independently helps coverage, but destroys predecessor state and therefore loses the actual values needed for computed refs and type information.

7. Global correctness depends on side conditions.
   - `aeim` must be initialized.
   - Function lists can mutate during analysis.
   - Address-range iteration is unsafe when code ownership and CFG reconstruction are incomplete.

## Root cause

The root problem is that the current implementation does not model emulation as a dataflow problem over a graph.

Instead, it tries to emulate code by choosing one next instruction stream, while using the emulator state itself as temporary traversal storage. That makes correctness depend on visitation order. As soon as multiple predecessors, loops, switches, or shared blocks exist, order-dependent emulation loses information.

For type propagation and computed references, the correct question is not:

- "Which block do I visit next?"

but:

- "What are all possible abstract states at the entry of this block, and what states leave it through each outgoing edge?"

That requires a fixpoint engine, not a one-shot walk.

## Required correctness model

The algorithm must operate on these principles.

### 1) Canonical graph, not function-local walk order

The primary input is a canonical program graph:

- nodes: basic blocks,
- edges: `jump`, `fail`, switch cases, fallthrough, optional call/return summaries,
- ownership: blocks are not assumed to belong to exactly one function,
- lookup: traversal is keyed by block identity/address, not by list position inside `fcn->bbs`.

Function boundaries remain useful as entry roots and for reporting, but they must not define the reachable state universe.

### 2) State lives on edges and block entries

For each reachable block we track one or more abstract input states.

Each state must minimally include:

- register file,
- stack pointer and frame-relative knowledge,
- sparse memory view for emulated writes,
- known constants and pointer-like values,
- known references already discovered,
- type hints derived from calls, loads, stores, and var accesses,
- control constraints that make a branch feasible,
- context information for recursion/calls.

The block itself is only a transfer function. The state is the real analysis object.

### 3) Path sensitivity until merge is safe

Different predecessors into the same block must remain separate until they are proven equivalent or safely joinable.

Examples:

- one predecessor sets `x0 = str.hello`, another sets `x0 = str.world`,
- one predecessor sets `SP += 0x20`, another does not,
- one predecessor reaches a shared tail block from function A, another from function B.

Joining these too early causes lost refs and bad types.

### 4) Monotone convergence

The engine must iterate until no block receives a new meaningful state.

That means:

- transfer functions are monotone,
- joins never become more precise than their inputs,
- widening is applied where loops would otherwise grow indefinitely,
- the analysis terminates predictably.

## Proposed solution

## A. Make emulation a worklist-based abstract interpreter

Implement a new analysis engine that uses a worklist of pending `(context, block, input-state)` items.

High-level flow:

1. Build or query the canonical block graph.
2. Seed the worklist with program roots.
   - function entrypoints,
   - discovered call targets,
   - optional explicit analysis roots from the user.
3. Pop one item from the worklist.
4. Merge it into the block-entry state table.
5. If it adds no new information, stop processing that item.
6. Otherwise emulate the block from that input state.
7. Produce successor states for each outgoing edge.
8. Enqueue those successor states.
9. Repeat until fixpoint.

This replaces both broken DFS-like behavior and address-sorted block emulation.

## B. Use a real abstract state, not the live register arena as traversal state

The current ESIL machinery can still be reused, but the state must be externalized.

Two layers are needed:

1. **Concrete ESIL snapshot layer**
   - serialization/deserialization of registers,
   - sparse VM overlay for writes,
   - PC/SP and architecture-specific flags.

2. **Abstract lattice layer**
   - constant value,
   - pointer/base+offset,
   - small set of possible values,
   - unknown/top,
   - impossible/bottom.

The engine should restore an input state into ESIL, emulate one block, then extract an output state back into the abstract representation.

This keeps ESIL as the execution mechanism, but stops using the traversal stack as the state model.

## C. Key the analysis by context, not only by block

The same block can be reached in different semantic contexts. The minimum key should be:

- block address or block id,
- root function or caller context id,
- optional call-string depth,
- optional memory/stack domain class.

Suggested key:

`StateKey = { block_id, context_id }`

where `context_id` is derived from:

- entry root,
- bounded call-string,
- flags describing whether the path is interprocedural, speculative, or shared-tail.

This is what prevents shared blocks from collapsing states from unrelated callers.

## D. Introduce entry-state partitions

A block should not have only one merged entry state if that destroys useful precision.

Instead, store a small partitioned set of incoming states per `StateKey`.

Recommended strategy:

- keep up to `N` distinct states per block/context,
- hash by a reduced signature of the values that matter for refs and types,
- join only states with compatible signatures,
- if the partition count exceeds `N`, widen into a conservative merged state.

Example signature fields:

- argument registers used by the block,
- stack delta class,
- known pointer-bearing registers,
- branch discriminator values,
- memory cells used for indirect refs.

This gives path sensitivity without unbounded explosion.

## E. Emulate whole basic blocks as transfer functions

For each input state:

1. Restore the state into ESIL.
2. Start at `bb->addr`.
3. Step instructions until:
   - end of block,
   - explicit control transfer,
   - invalid decode with stop policy,
   - impossible state,
   - iteration/widening limit.
4. Record side effects during emulation:
   - computed code refs,
   - computed data refs,
   - string refs,
   - stack/var access patterns,
   - inferred argument and return value types,
   - load/store base types.
5. Split the output by outgoing edge semantics.

Each outgoing edge receives its own successor state.

For conditional branches:

- true-edge state gets the condition constraint,
- false-edge state gets the negated constraint,
- if the condition is decidable, only emit the feasible edge.

For switches:

- emit one state per concrete case target when known,
- emit a conservative default summary when the index is unknown.

## F. Add explicit join and widening rules

The analysis must define what it means to merge states.

### Registers

For each register:

- same constant + same provenance => keep constant,
- different small constants => keep small value-set up to threshold,
- different pointers to same base => keep base + joined offset domain,
- unrelated values => top/unknown.

### Stack and memory

Use sparse memory cells keyed by address class:

- stack slot,
- absolute address,
- register-relative address,
- unknown alias class.

Join memory cell values with the same lattice used for registers.

### Types

Track types as derived facts, not as primary execution state.

Examples:

- register `x0` may hold pointer to `char`,
- stack slot `[sp+0x10]` may hold `int *`,
- call target summary says arg0 is `const char *`.

When states merge, type facts are intersected or weakened conservatively.

### Widening

Loops and recursion need widening to terminate.

Recommended widening rules:

- constant sequence becomes interval or top after threshold,
- growing pointer offset sets collapse to base+unknown-offset,
- memory cells beyond per-block limit are summarized,
- repeated visits with no new ref/type facts stop.

## G. Handle loops by fixpoint, not by traversal hacks

Loops are expected and should not be treated as traversal failures.

For a back edge:

1. the successor state is enqueued back to the loop header,
2. it is joined with the current header entry state,
3. if the join adds information, the header is reprocessed,
4. after the widening threshold, precision is reduced to guarantee termination.

This solves the exact class of bugs where a backward jump prevented later blocks from being analyzed correctly.

## H. Shared basic blocks must be first-class citizens

Shared blocks are not an anomaly. They must be modeled explicitly.

Rules:

- never assume `fcn->bbs` is the unique block set for emulation,
- never reject a block just because it is also reachable from another function,
- never let one function's path overwrite another function's incoming state for a shared block,
- report refs discovered in shared blocks to all owning contexts that reach them.

The graph should support:

- one block with multiple predecessor functions,
- one block with multiple owning symbols,
- one block reachable from a range walk even if not yet attached to the expected function list.

## I. Interprocedural model

To resolve references and propagate types across the whole program, interprocedural analysis is needed, but it should be bounded.

Recommended call handling:

### Direct calls

- create or reuse a callee summary keyed by callee entry + context class,
- analyze the callee recursively,
- return a summary describing:
  - clobbered registers,
  - preserved registers,
  - return register value domain,
  - produced refs,
  - argument type effects,
  - memory write classes.

### Indirect calls

- if target value-set is small, fork to each candidate callee,
- otherwise apply an unknown-call summary that kills volatile regs and weakens memory conservatively.

### Recursion

- use SCC-aware scheduling or bounded call-string recursion,
- widen recursive summaries until stable.

This keeps whole-program traversal correct without requiring unbounded inlining.

## J. Separate discovery from reporting

The engine should accumulate facts during traversal and commit them after stabilization checkpoints.

Facts include:

- xrefs,
- string refs,
- call targets,
- var accesses,
- inferred parameter/return types,
- pointer provenance.

Why separate them:

- prevents transient states from emitting contradictory data,
- avoids duplicate refs from repeated visits,
- allows fact provenance to be stored with the contributing path/context.

## Concrete implementation plan

## Phase 1: add a generic path-state engine

Introduce a new internal module, for example:

- `libr/anal/emu_flow.c`
- `libr/anal/emu_state.c`
- `libr/anal/emu_summary.c`

Responsibilities:

- canonical successor iteration,
- worklist scheduling,
- state keying and joins,
- ESIL snapshot restore/extract,
- loop widening,
- fact collection.

This should become the single source of truth for graph traversal in emulation-sensitive analyses.

## Phase 2: make `r_core_anal_esil()` use the new engine

Replace `get_next_i()`-driven function walking with:

- root selection,
- graph-state worklist execution,
- final fact commit.

Important:

- keep `aeim` initialization mandatory,
- snapshot function roots before the analysis starts to avoid list mutation hazards,
- do not use address-range iteration as the semantic traversal mechanism.

## Phase 3: make type propagation consume path states

`libr/anal/p/anal_tp.c` should stop sorting blocks by address as its primary strategy.

Instead it should:

- query entry states produced by the traversal engine,
- reuse the same block transfer facts,
- propagate type information per incoming path partition,
- join types only at explicit merge points.

This is the point where the current TODO in `anal_tp.c` is finally resolved correctly.

## Phase 4: introduce block and function summaries

Cache:

- block transfer summaries for stable input signatures,
- function summaries for interprocedural reuse,
- invalidation hooks when analysis modifies CFG or block contents.

This improves performance without changing semantics.

## Phase 5: unify traversal semantics elsewhere

The traversal unification proposed in `gt/newreport.md` should be aligned with this work.

Even if shortest-path code and type-propagation code have different goals, they should share:

- one canonical successor policy,
- one address-to-block normalization policy,
- one understanding of switch/call/ref edges.

## Data structures

Recommended internal structures:

```c
typedef struct {
	ut64 block_addr;
	ut32 context_id;
	ut16 partition_id;
} REmuStateKey;

typedef struct {
	RHashTable *regs;
	RHashTable *mem;
	RVector refs;
	RVector type_facts;
	ut64 sp;
	ut64 pc;
	ut32 flags;
} REmuAbstractState;

typedef struct {
	REmuStateKey key;
	REmuAbstractState *in_state;
} REmuWorkItem;
```

The exact containers can differ, but the semantics should be the same:

- immutable or copy-on-write states,
- block-entry table,
- stable worklist,
- deduplicated fact store.

## State domains that matter for refs and types

The implementation does not need a fully symbolic executor. It needs the smallest useful abstract domains that preserve ref/type correctness.

Minimum useful domains:

1. **Const**
   - exact scalar or address.

2. **Pointer(base, offset)**
   - image base + offset,
   - stack base + offset,
   - heap/unknown base + offset.

3. **ValueSet(N)**
   - small set of constants/pointers for branch-sensitive paths.

4. **Unknown**
   - conservative top.

5. **Unreachable**
   - bottom.

This is enough to model many computed refs such as:

- `adrp + add`,
- stack-based string/struct access,
- switch tables,
- jump tables,
- tail-merged argument forwarding,
- pointer-carrying shared epilogues.

## Scheduling strategy

Use a deterministic worklist order so results are reproducible, but do not depend on that order for correctness.

Recommended policy:

- priority by function root,
- then reverse postorder inside an SCC when possible,
- then stable address order as a tie-breaker only.

Reverse postorder helps convergence, but the fixpoint rules provide correctness.

## Failure policy

When the engine cannot decode or cannot resolve a target:

- preserve already learned facts,
- emit conservative unknown-state successors when appropriate,
- do not abort the whole function unless explicitly configured.

The current all-or-nothing behavior makes diagnosis harder and hides partial progress.

## Why this fixes PR `#25496` class regressions

The workaround in that PR tried to avoid missed refs caused by bad block visitation order. The new algorithm fixes the actual cause:

- refs are derived from stabilized path states, not from the order of a one-shot walk,
- shared blocks keep separate incoming states per context,
- loops revisit headers until stable,
- alternate paths are not lost just because one branch was visited first,
- type propagation reads the same path-sensitive facts as xref discovery.

This means a block shared by multiple functions can be analyzed once per relevant context instead of being accidentally "claimed" by whichever traversal touched it first.

## Test plan

Add dedicated tests for these cases:

1. diamond CFG where both branches set different constant pointer values,
2. loop with backward jump and post-loop block using a carried register,
3. switch with multiple case targets and shared exit block,
4. shared tail block reached from two functions with different argument registers,
5. overlapping function ranges with one shared block,
6. sparse function with gaps and noncontiguous reachable blocks,
7. indirect call with small target set,
8. recursive function with bounded summary convergence,
9. `adrp + add` string/materialized pointer sequence,
10. regression case from PR `#25496`.

Success criteria:

- `aae` and `aaef` do not disagree on reachable computed refs,
- type propagation is invariant under block ordering,
- shared-block refs are preserved for all callers,
- rerunning the analysis yields stable results,
- debug logs show fixpoint convergence instead of traversal dead-ends.

## Non-goals

This plan does not require:

- full SMT solving,
- perfect alias analysis,
- path explosion without bounds,
- exact modeling of every unknown indirect edge.

The objective is a conservative, terminating, path-sensitive analysis that is strong enough to resolve computed references and propagate types correctly in real binaries.

## Recommended next steps

1. implement the generic state/worklist engine,
2. migrate `r_core_anal_esil()` to it,
3. teach `anal_tp.c` to consume path-partitioned entry states,
4. add shared-block and loop regression tests,
5. remove the workaround logic once the new engine reaches parity.

## One-line summary

The correct fix is to stop treating ESIL analysis as a block visitation problem and reimplement it as a path-sensitive abstract interpretation over the canonical CFG, with explicit register/memory state at block entries, stable joins, bounded partitions, and interprocedural summaries.
