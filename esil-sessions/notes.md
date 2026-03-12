# ESIL sessionization plan (towards thread-safe execution contexts)

## Objective

Refactor ESIL integration so execution uses explicit **sessions/contexts** instead of mutable global state. Keep one shared, reusable global ESIL metadata state (ops/plugins/instruction semantics cache) that can be held with `r_ref`, while per-execution mutable state (register arena, trap/trace state, callbacks/user payload, mem/reg hook wiring) is isolated per session.

This is aligned with existing session patterns used in `rbinfile`, `r_arch` sessions, and `rmuta` sessions.

## Current `core->esil` usage inventory

### Direct `core->esil` access

`core->esil` references are concentrated in:

- `libr/core/core_esil.c` (37 matches): owns initialization/finalization, runtime stepping, trap revert, custom ops (`TODO`, `$`), and temporary architecture callback wiring through `core->anal->arch->esil`.
- `libr/core/cconfig.c` (6 matches): toggles `core->esil.cfg` bits (`RO`, `NONULL`, `TRAP_REVERT_CONFIG`).
- `man/3/r_arch.3` (2 matches): documentation examples using `core->esil.reg`.
- `libr/core/canal.c` matches are `core->esil_anal_stop` (separate flag, not `RCoreEsil`).

### Adjacent global ESIL coupling (important for migration scope)

Even where `core->esil` is not directly touched, many call sites depend on global `core->anal->esil` mutation or singleton assumptions:

- `libr/core/cmd_anal.inc.c` (42 matches)
- `libr/core/cconfig.c` (40 matches)
- `libr/core/disasm.c` (17 matches)
- `libr/core/cmd_debug.inc.c` (16 matches)
- `libr/core/cmd_search.inc.c` (7 matches)
- `libr/core/cmd_search_rop.inc.c` (5 matches)
- `libr/core/canal.c` (3 matches)

These are the main risk areas for cross-session interference.

## Proposed target architecture

Introduce 2 levels:

1. `REsilShared` (refcounted, mostly immutable after init)
   - ESIL operation registry and plugin activation template
   - decode/instruction semantics caches
   - architecture-specific callbacks registration baseline
   - managed via `r_ref`

2. `REsilSession` (short/medium-lived execution context)
   - per-session register state and reg interface binding
   - mem interface + IO bank selection and policy flags
   - trap state, trace buffers, stats db handle, user pointer
   - callback hooks and temporary command hooks
   - optional overlay to shared defaults

`RCore` should hold a default shared ESIL object and create/reuse explicit sessions per task (analysis pass, disasm emulation, debug emulation, search/rop emulation).

## Refactor phases

### Phase 0 — Contract and compatibility shell

- Add new public/private types and APIs:
  - `r_esil_shared_new/free/ref/unref`
  - `r_esil_session_new/free`
  - `r_esil_session_reset`
  - bridge helpers from legacy `REsil *` to session-backed object.
- Keep old entry points as thin wrappers to avoid large one-shot migration.
- Add docs defining what is shared vs per-session mutable.

Deliverable: compiles with no behavior changes.

### Phase 1 — Isolate `RCoreEsil` into shared + default session

- Split `RCoreEsil` responsibilities:
  - shared template object in `RCore`
  - one default session used by current commands.
- Move mutable fields out of shared state (`trap_revert`, `old_pc`, traps, user hooks, command strings where needed).
- Keep `core_esil_mem_if`/`core_esil_reg_if` session-bound (not static-global mutable references to `core`).

Deliverable: `r_core_esil_single_step` and init/fini use explicit session object.

### Phase 2 — Replace temporary global swaps (`arch->esil`)

- Remove pattern:
  - save `arch->esil`
  - set `arch->esil = &core->esil.esil`
  - callback
  - restore
- Add `r_arch_esilcb_session (arch, session, action)` (or equivalent API) so callback targets a supplied session explicitly.

Deliverable: no temporary global pointer rewrites in `core_esil.c`.

### Phase 3 — Migrate high-interference callers

Prioritize callers that mutate `esil->user`, `esil->cb.*`, trace/trap fields:

1. `cmd_anal.inc.c`
2. `disasm.c`
3. `cmd_debug.inc.c`
4. `cmd_search*.inc.c`

Pattern:

- Acquire shared ESIL template (`r_ref`).
- Create local session for the operation scope.
- Apply scope-specific hooks/flags.
- Execute.
- Destroy session (`r_unref`/free).

Deliverable: these paths no longer rely on singleton `core->anal->esil` mutable state.

### Phase 4 — Config model split (global defaults vs session overrides)

- Keep config keys (e.g. `esil.romem`, `esil.nonull`, trap policies) as defaults in shared/template config.
- At session creation, snapshot defaults into session runtime options.
- Add optional per-command/session overrides without mutating global singleton.

Deliverable: `cconfig` writes defaults; active sessions are explicitly updated/created from defaults.

### Phase 5 — Thread-safety hardening and tests

- Audit/remove remaining static/thread-local cross-talk where feasible.
- Add targeted tests:
  - two concurrent ESIL sessions with different register/memory hooks
  - trap/trace isolation
  - deterministic results under parallel task runner
- Add stress harness for repeated create/run/free cycles.

Deliverable: CI coverage for session isolation and regression prevention.

## Migration strategy details

- Use adapter shims to keep existing `r_esil_*` call sites compiling while progressively swapping to session-aware APIs.
- Migrate by feature slices, not by file-only changes, to keep behavior testable.
- Keep shared metadata immutable after warmup; mutable fields must be session-owned.
- Ensure shared state lifetime is `r_ref`-managed, enabling pooling and safe cross-task reuse.

## Risks and mitigations

- **Risk:** hidden writes through `core->anal->esil` aliases.
  - **Mitigation:** temporary instrumentation macro to log writes to `esil->user`, `esil->cb`, trap/trace fields during migration.
- **Risk:** performance regressions from per-op setup.
  - **Mitigation:** pre-warmed shared object + lightweight session reset/recycle pool.
- **Risk:** plugin API breakage.
  - **Mitigation:** keep legacy callbacks as wrappers; deprecate gradually.

## Definition of done

- No runtime path requires mutating a global singleton ESIL object for temporary operation-local behavior.
- Shared ESIL metadata is refcounted and reusable (`r_ref`).
- Multiple sessions can execute concurrently without register/memory/trap/trace interference.
- Existing user-facing behavior and commands remain compatible.

