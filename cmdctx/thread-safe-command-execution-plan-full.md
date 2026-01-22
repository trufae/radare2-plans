# Thread-Safe Command Execution: Incremental PR Plan

## Compatibility Strategy

During migration, `core->default_ctx` bridges old and new code:
- `default_ctx->block` = `core->block` (same pointer)
- `default_ctx->addr` synced with `core->addr`
- `default_ctx->blocksize` synced with `core->blocksize`

Migrated code (`ctx->block`) and unmigrated code (`core->block`) see the same data. Each PR is independently mergeable without breaking anything.

## Isolation Levels

Tasks request an isolation level at creation time:

```c
typedef enum {
	R_CORE_TASK_ISOLATION_SHARED = 0,   // Legacy: shares core->block/addr directly
	R_CORE_TASK_ISOLATION_SNAPSHOT = 1,  // Private block copy, shared config (HTTP server)
	R_CORE_TASK_ISOLATION_ISOLATED = 2   // Private block + cloned config (full parallel)
} RCoreTaskIsolation;
```

- **SHARED**: default_ctx behavior during migration (no copies)
- **SNAPSHOT**: `r_core_task_ctx_new()` allocates private block, reads via `r_io_read_at()`
- **ISOLATED**: additionally clones config and console context

## Context Propagation

On task completion, contexts can propagate changes back to core:

```c
bool propagate_seek;    // Write ctx->addr back to core->addr
bool propagate_config;  // Merge config changes back to core->config
bool config_dirty;      // Track whether config was modified
```

Main-thread tasks propagate by default. Background/HTTP tasks don't.
Conflict resolution: last-to-commit wins (acceptable for seek; config merge is key-level).

## Success Criteria

1. HTTP server handles 10 concurrent requests without data corruption
2. Background tasks (`&` prefix) execute with correct isolated state
3. No memory leaks in context creation/destruction cycle
4. Performance within 10% of current single-threaded execution
5. All existing tests pass with new infrastructure

---

## PR 1: Add RCoreTaskCtx infrastructure (dead code)

**No behavioral change. New struct + functions, unused by existing code.**

### Files changed:

**`libr/include/r_core.h`** - Add before `struct r_core_t` (around line 326):
```c
typedef struct r_core_task_ctx_t {
	struct r_core_t *core;
	RCoreTaskIsolation isolation;
	// address/block state
	ut64 addr;
	ut8 *block;
	ut32 blocksize;
	bool block_valid;         // false after seek, lazy re-read on access
	// config state
	RConfig *config;
	bool config_dirty;        // whether config was modified in this ctx
	// propagation flags (set at creation based on isolation level)
	bool propagate_seek;      // write addr back to core on completion
	bool propagate_config;    // merge config changes back on completion
	// console state
	RConsContext *cons_ctx;
	// command state
	int cmd_depth;
	int cmd_depth_max;
	char *lastcmd;
	bool tmpseek;
	bool incomment;
	int cmdrepeat;
	// disasm state (was static globals)
	ut64 disasm_oaddr;
	char *disasm_section;
	// analysis state (was static globals)
	char *oldregread;
	RList *mymemxsr;
	RList *mymemxsw;
	ut64 esilbreak_last_read;
	ut64 esilbreak_last_data;
	ut64 esil_ntarget;
	// flag state
	int flagenum;
	// visual state
	int vmenu_level;
	st64 vmenu_delta;
	int vmenu_option;
	int vmenu_variable_option;
	int vmenu_printMode;
	bool vmenu_selectPanel;
	int vmenu_sortMode;
} RCoreTaskCtx;
```

Add to `struct r_core_t` (after line 335 `ut8 *block`):
```c
	RCoreTaskCtx *default_ctx;
```

Add API declarations at bottom of header:
```c
R_API RCoreTaskCtx *r_core_task_ctx_new(RCore *core, RCoreTaskIsolation level);
R_API void r_core_task_ctx_free(RCoreTaskCtx *ctx);
R_API void r_core_task_ctx_commit(RCoreTaskCtx *ctx);
R_API RCoreTaskCtx *r_core_ctx(RCore *core);
R_API ut8 *r_core_ctx_get_block(RCoreTaskCtx *ctx);
R_API bool r_core_ctx_seek(RCoreTaskCtx *ctx, ut64 addr, bool rb);
R_API bool r_core_ctx_block_size(RCoreTaskCtx *ctx, ut32 bsize);
```

**`libr/core/ctx.c`** (new file):
```c
/* radare - LGPL - Copyright 2025 - pancake */

#include <r_core.h>

R_API RCoreTaskCtx *r_core_task_ctx_new(RCore *core, RCoreTaskIsolation level) {
	R_RETURN_VAL_IF_FAIL (core, NULL);
	RCoreTaskCtx *ctx = R_NEW0 (RCoreTaskCtx);
	ctx->core = core;
	ctx->isolation = level;
	ctx->addr = core->addr;
	ctx->blocksize = core->blocksize;
	ctx->cmd_depth = core->max_cmd_depth;
	ctx->cmd_depth_max = core->max_cmd_depth;
	ctx->disasm_oaddr = UT64_MAX;
	ctx->esilbreak_last_read = UT64_MAX;
	ctx->esilbreak_last_data = UT64_MAX;
	ctx->esil_ntarget = UT64_MAX;
	switch (level) {
	case R_CORE_TASK_ISOLATION_SHARED:
		ctx->block = core->block;
		ctx->config = core->config;
		ctx->propagate_seek = true;
		break;
	case R_CORE_TASK_ISOLATION_SNAPSHOT:
		ctx->block = malloc (ctx->blocksize);
		r_io_read_at (core->io, ctx->addr, ctx->block, ctx->blocksize);
		ctx->block_valid = true;
		ctx->config = core->config;
		break;
	case R_CORE_TASK_ISOLATION_ISOLATED:
		ctx->block = malloc (ctx->blocksize);
		r_io_read_at (core->io, ctx->addr, ctx->block, ctx->blocksize);
		ctx->block_valid = true;
		ctx->config = r_config_clone (core->config);
		ctx->cons_ctx = r_cons_context_clone (core->cons->context);
		break;
	}
	return ctx;
}

R_API void r_core_task_ctx_free(RCoreTaskCtx *ctx) {
	if (!ctx) {
		return;
	}
	if (ctx->isolation != R_CORE_TASK_ISOLATION_SHARED) {
		free (ctx->block);
	}
	if (ctx->isolation == R_CORE_TASK_ISOLATION_ISOLATED) {
		r_config_free (ctx->config);
		r_cons_context_free (ctx->cons_ctx);
	}
	free (ctx->lastcmd);
	free (ctx->disasm_section);
	free (ctx->oldregread);
	r_list_free (ctx->mymemxsr);
	r_list_free (ctx->mymemxsw);
	free (ctx);
}

R_API void r_core_task_ctx_commit(RCoreTaskCtx *ctx) {
	R_RETURN_IF_FAIL (ctx && ctx->core);
	RCore *core = ctx->core;
	R_CRITICAL_ENTER (core);
	if (ctx->propagate_seek) {
		core->addr = ctx->addr;
	}
	if (ctx->propagate_config && ctx->config_dirty) {
		// TODO: key-level merge from ctx->config to core->config
	}
	R_CRITICAL_LEAVE (core);
}

R_API RCoreTaskCtx *r_core_ctx(RCore *core) {
	R_RETURN_VAL_IF_FAIL (core, NULL);
	RCoreTask *task = r_core_task_self (&core->tasks);
	if (task && task->ctx) {
		return task->ctx;
	}
	return core->default_ctx;
}

R_API ut8 *r_core_ctx_get_block(RCoreTaskCtx *ctx) {
	R_RETURN_VAL_IF_FAIL (ctx, NULL);
	if (!ctx->block_valid && ctx->block) {
		r_io_read_at (ctx->core->io, ctx->addr, ctx->block, ctx->blocksize);
		ctx->block_valid = true;
	}
	return ctx->block;
}

R_API bool r_core_ctx_seek(RCoreTaskCtx *ctx, ut64 addr, bool rb) {
	R_RETURN_VAL_IF_FAIL (ctx, false);
	ctx->addr = addr;
	ctx->block_valid = false;
	if (rb) {
		r_core_ctx_get_block (ctx);
	}
	return true;
}

R_API bool r_core_ctx_block_size(RCoreTaskCtx *ctx, ut32 bsize) {
	R_RETURN_VAL_IF_FAIL (ctx, false);
	if (bsize < 1 || bsize > ctx->core->blocksize_max) {
		return false;
	}
	ut8 *nb = realloc (ctx->block, bsize);
	if (!nb) {
		return false;
	}
	ctx->block = nb;
	ctx->blocksize = bsize;
	ctx->block_valid = false;
	r_core_ctx_get_block (ctx);
	return true;
}
```

**`libr/core/Makefile:9`** - add ctx.o:
```
OBJS=core.o cmd.o cfile.o cconfig.o visual.o cio.o yank.o libs.o agraph.o xpatch.o ctx.o
```

**`libr/core/meson.build`** - add to r_core_sources list:
```
'ctx.c',
```

**`libr/core/core.c`** - In `r_core_new()` (after line 2525 `core->block = calloc(...)`:
```c
	core->default_ctx = R_NEW0 (RCoreTaskCtx);
	if (core->default_ctx) {
		core->default_ctx->core = core;
		core->default_ctx->isolation = R_CORE_TASK_ISOLATION_SHARED;
		core->default_ctx->block = core->block;
		core->default_ctx->blocksize = core->blocksize;
		core->default_ctx->block_valid = true;
		core->default_ctx->addr = core->addr;
		core->default_ctx->config = core->config;
		core->default_ctx->propagate_seek = true;
		core->default_ctx->cmd_depth = core->max_cmd_depth;
		core->default_ctx->cmd_depth_max = core->max_cmd_depth;
		core->default_ctx->disasm_oaddr = UT64_MAX;
		core->default_ctx->esilbreak_last_read = UT64_MAX;
		core->default_ctx->esilbreak_last_data = UT64_MAX;
		core->default_ctx->esil_ntarget = UT64_MAX;
	}
```

In `r_core_fini()` (cleanup):
```c
	free (core->default_ctx);  // don't use ctx_free - block is owned by core
	core->default_ctx = NULL;
```

In `r_core_seek()` (cio.c:426, before return):
```c
	if (core->default_ctx) {
		core->default_ctx->addr = core->addr;
	}
```

In `r_core_seek_size()` (core.c:3160, after block assignment):
```c
	if (core->default_ctx) {
		core->default_ctx->block = core->block;
		core->default_ctx->blocksize = core->blocksize;
	}
```

Test: `make -j > /dev/null && r2r test/db/cmd/`

---

## PR 2: Enable R_CRITICAL_ENABLED

**One-line change. Activates existing lock infrastructure.**

Files:
- `libr/include/r_userconf.h:17` - change `#define R_CRITICAL_ENABLED 0` to `1`

This activates all existing R_CRITICAL_ENTER/LEAVE calls in:
- task.c (scheduler)
- cio.c (block_read)
- core.c (seek_size)
- cmd.c (cmd_subst_i partial)
- cfile.c (file ops)
- project.c (project load)
- cbin.c (bin info)

Test: `make -j > /dev/null && r2r test/db/cmd/`

---

## PR 3: Add R_CRITICAL to r_core_seek/seek_delta

**Protects core->addr mutations. No behavioral change in single-threaded use.**

Files:
- `libr/core/cio.c` - wrap r_core_seek() and r_core_seek_delta() bodies with R_CRITICAL_ENTER/LEAVE

Test: `make -j > /dev/null && r2r test/db/cmd/cmd_seek`

---

## PR 4: Wire ctx into task system

**Tasks create/destroy contexts. No change to command execution yet.**

Files:
- `libr/include/r_core_task.h` - add `RCoreTaskCtx *ctx` and `RCoreTaskIsolation isolation` fields to RCoreTask
- `libr/core/task.c` - in task_run():
  - Create ctx: `task->ctx = r_core_task_ctx_new (core, task->isolation);`
  - After cmd execution: `r_core_task_ctx_commit (task->ctx);`
  - Then: `r_core_task_ctx_free (task->ctx);`
  - Main task uses SHARED, background tasks (`&` prefix) use SNAPSHOT

Default isolation for task creation:
```c
// In r_core_task_new_cmd():
task->isolation = (task->id == 0)
    ? R_CORE_TASK_ISOLATION_SHARED
    : R_CORE_TASK_ISOLATION_SNAPSHOT;
```

Test: `make -j > /dev/null && r2r test/db/cmd/`

---

## PR 5: Remove HTTP block-swap, use SNAPSHOT isolation

**Removes unsafe block/addr swapping in HTTP server. HTTP commands execute in SNAPSHOT isolation via task context (private block, shared config).**

Files:
- `libr/core/rtr_http.inc.c`:
  - Remove origoff/origblk/origblksz/newoff/newblk/newblksz variables and swap logic (lines 201-232, 275-281)
  - HTTP command execution already goes through `r_core_cmd_str()` which uses the task system
  - Set `task->isolation = R_CORE_TASK_ISOLATION_SNAPSHOT` for HTTP-created tasks
  - The config clone (`newcfg`) can also be removed since ISOLATED level handles it

This is the first real validation that the context system works under concurrent load.

Test: `make -j > /dev/null && r2 -qc '=H 9090' /bin/ls` then concurrent: `for i in $(seq 10); do curl -s http://localhost:9090/cmd/pd%2010 & done; wait`

---

## PRs 6-20: Migrate core->block → ctx->block (one file per PR)

**Each PR migrates one file. Adds `RCoreTaskCtx *ctx = r_core_ctx(core);` at function entries, replaces core->block/blocksize/addr with ctx equivalents. Since default_ctx shares the pointer, behavior is identical.**

Pattern for each PR:
```c
// Before:
static void cmd_foo(RCore *core) {
    r_print_hexdump (core->print, core->addr, core->block, core->blocksize, ...);
}

// After (direct access - when block is known valid):
static void cmd_foo(RCore *core) {
    RCoreTaskCtx *ctx = r_core_ctx (core);
    r_print_hexdump (core->print, ctx->addr, ctx->block, ctx->blocksize, ...);
}

// After (lazy access - when block may be stale after seek):
static void cmd_bar(RCore *core) {
    RCoreTaskCtx *ctx = r_core_ctx (core);
    ut8 *block = r_core_ctx_get_block (ctx);  // re-reads if block_valid==false
    r_print_hexdump (core->print, ctx->addr, block, ctx->blocksize, ...);
}
```

Use `r_core_ctx_get_block()` when the function does seeks before reading.
Use `ctx->block` directly when the block is already current (entry point of command handlers).

### PR 6: cio.c (18 refs)

Functions to migrate:
| Function | Refs | Action |
|----------|------|--------|
| `r_core_dump` (line 68) | 1 | `core->blocksize` → `ctx->blocksize` |
| `r_core_transform_op` (line 113) | 11 | Replace all block/blocksize refs |
| `r_core_write_op` (line 344) | 1 | `core->blocksize` → `ctx->blocksize` |
| `r_core_write_at` (line 450) | 2 | Block overlap check uses ctx |
| `r_core_extend_at` (line 485) | 1 | Block range check uses ctx |
| `r_core_block_read` (line 571) | 2 | Read into `ctx->block` |

### PR 7: cmd.c (15 refs)

Functions to migrate:
| Function | Refs | Action |
|----------|------|--------|
| `r_core_readblock` (line 68) | 1 | `core->blocksize` → `ctx->blocksize` |
| `lastcmd_repeat` (~line 609) | 4 | Seek by `ctx->blocksize` increments |
| `cmd_bsize` (line 2732) | 8 | Get/set via ctx_block_size |
| `r_core_cmd_subst_i` (line 4265) | 3 | tmpbsz save/restore uses ctx |

Note: `cmd_bsize` implements the `b` command. It should call `r_core_ctx_block_size(ctx, newsize)` and also sync to core's blocksize_max.

### PR 8: disasm.c (23 refs + 2 globals)

Functions to migrate:
| Function | Refs | Action |
|----------|------|--------|
| `r_core_disassemble` (~line 3795) | 4 | Remove block swap pattern, use local tmpblock |
| `ds_disassemble` (~line 7239) | 1 | Use `ctx->block` |
| `r_core_disasm_bytes` (~line 7380) | 15+ | Replace all block/blocksize refs with ctx |

Globals to move into ctx:
- `static R_TH_LOCAL ut64 Goaddr` (line 25) → `ctx->disasm_oaddr`
- `static R_TH_LOCAL char *Gsection` (line 26) → `ctx->disasm_section`

All functions that read/write Goaddr/Gsection get `RCoreTaskCtx *ctx = r_core_ctx(core);` and use `ctx->disasm_oaddr`/`ctx->disasm_section` instead.

### PR 9: canal.c (25 refs + 3 globals)

Functions to migrate:
| Function | Refs | Action |
|----------|------|--------|
| `r_core_anal_refine_fcn_bb` (~line 750) | 4 | Replace block/blocksize |
| `r_core_seek_to_opcode_forward` (~line 1200) | 6 | Replace buffer range checks |
| `r_core_seek_to_opcode_backward` (~line 4369) | 13+ | Replace all buffer ops |
| `r_core_anal_type_match` (~line 4858) | 2 | Replace block access |

Globals to move into ctx:
- `static R_TH_LOCAL ut64 esilbreak_last_read` (line 5459) → `ctx->esilbreak_last_read`
- `static R_TH_LOCAL ut64 esilbreak_last_data` (line 5460) → `ctx->esilbreak_last_data`
- `static R_TH_LOCAL ut64 ntarget` (line 5461) → `ctx->esil_ntarget`

### PR 10: cmd_search.inc.c (23 refs)
- Replace block access with ctx

### PR 11: cmd_write.inc.c (33 refs)
- Replace block reads with ctx

### PR 12: visual.c (47 refs)
- Replace block access with ctx

### PR 13: cmd_print.inc.c - part 1 (~60 refs)
- Migrate functions a-m

### PR 14: cmd_print.inc.c - part 2 (~60 refs)
- Migrate functions n-z

### PR 15: cmd_print.inc.c - part 3 (~53 refs)
- Migrate remaining functions

### PR 16: cmd_anal.inc.c (15 refs)
- Move oldregread, mymemxsr/w globals into ctx

### PR 17: Small files batch 1 (cmd_seek, cmd_cmp, yank - 21 refs total)

### PR 18: Small files batch 2 (cmd_flag, cmd_meta, cmd_magic, carg - 13 refs total)

### PR 19: Small files batch 3 (casm, cbin, cfile, cmd_debug, cmd_open, cmd_type - 10 refs total)

### PR 20: vmenus.c (5 refs)
- Move visual state globals into RCoreVisualState sub-struct in ctx

Test for each: `make -j > /dev/null && r2r test/db/cmd/`

---

## PR 21: Remove core->block/blocksize fields

**After all consumers migrated, remove the fields. default_ctx owns the block.**

Files:
- `libr/include/r_core.h` - remove `ut8 *block` and `ut32 blocksize` from r_core_t
- `libr/core/core.c` - block allocation moves to default_ctx creation
- `libr/core/cio.c` - r_core_block_read/size operate on ctx directly
- Fix any remaining references (compile errors guide this)

The default_ctx now owns the block buffer. `r_core_ctx(core)->block` is the only way to access it.

Test: `make -j > /dev/null && r2r test/db/cmd/`

---

## PR 22: Remove core->addr usage in commands

**Replace core->addr with ctx->addr in command implementations.**

This is the same pattern as block migration but for the addr field. Many commands already use ctx->addr from the block PRs. This PR catches the remaining direct core->addr reads/writes in:
- cmd_print.inc.c (addr used without block)
- cmd_seek.inc.c (seek sets addr directly)
- disasm.c (addr for annotations)
- visual.c (addr for navigation)

---

## PRs 23-27: Global variable elimination (one PR per group)

### PR 23: Console globals → RCons fields
- `Dietline D` → `cons->dietline`
- `RConsEditor G` → `cons->editor`
- `Glock` → `cons->pal_lock`

### PR 24: Disasm globals → ctx (already done in PR 8 if combined)
- Verify Goaddr, Gsection moved

### PR 25: Canal/analysis globals → ctx
- esilbreak_last_read, esilbreak_last_data, ntarget

### PR 26: Visual menu globals → RCoreVisualState (already done in PR 20 if combined)

### PR 27: Remaining globals
- const_color → local variable
- oldcwd → ctx field
- cur_name → ctx field
- mydb/osymbols → remove (deprecated)
- Gcore → signal-safe alternative

---

## PRs 28-31: Subsystem rwlocks (one PR per subsystem)

### PR 28: RThreadRWLock implementation
- `libr/include/r_th.h` - add RThreadRWLock type
- `libr/util/thread_rwlock.c` (new) - implementation

### PR 29: IO rwlock
- `libr/include/r_io.h` - add lock field
- `libr/io/io.c` - rdlock in r_io_read_at(), wrlock in r_io_write_at()

### PR 30: Flags rwlock
- `libr/include/r_flag.h` - add lock field
- `libr/flag/flag.c` - rdlock in get, wrlock in set/unset

### PR 31: Analysis + Bin rwlocks + write queueing

RAnal contains linked lists (functions, xrefs) that aren't thread-safe for writes.
Strategy: reads use rdlock, writes queue mutations for main-thread application.

- `libr/include/r_anal.h` - add lock field + `RList *pending_writes`
- `libr/include/r_bin.h` - add lock field
- `libr/anal/anal.c`:
  - rdlock in read accessors (fcn_find, xref_get, etc.)
  - Write operations (fcn_add, xref_add) queue to `pending_writes` when called from non-main task
  - `r_anal_pending_flush(RAnal *anal)` applies queued writes under wrlock (called by main task)
- `libr/bin/bin.c` - rdlock in get, wrlock in set operations

This avoids holding write locks for long operations (analysis can take seconds) while keeping the database consistent.

---

## PR 32: RCons singleton elimination

**The largest console refactor. Make RCons passed as parameter instead of global `I`.**

- `libr/cons/cons.c` - remove `static RCons *I`, pass cons as parameter
- All internal cons functions get `RCons *cons` parameter
- `r_cons_singleton()` → deprecated compatibility wrapper
- Signal handler uses R_TH_LOCAL pointer (already partially done with s_cons_thread)

---

## Summary: 32 PRs, each independently mergeable

| PR | Description | Risk | Size |
|----|-------------|------|------|
| 1 | RCoreTaskCtx infrastructure | None (dead code) | S |
| 2 | R_CRITICAL_ENABLED=1 | Low | XS |
| 3 | Lock r_core_seek/delta | Low | S |
| 4 | Wire ctx into task system | Low | S |
| 5 | Remove HTTP block-swap | Medium | S |
| 6-20 | Migrate core->block (per file) | Low each | S-M |
| 21 | Remove core->block field | Medium | S |
| 22 | Migrate core->addr usage | Low | M |
| 23-27 | Eliminate globals (per group) | Low each | S |
| 28 | RThreadRWLock impl | None (additive) | S |
| 29-31 | Subsystem rwlocks | Low each | S |
| 32 | RCons singleton removal | High | L |

Each PR:
- Compiles cleanly (`make -j > /dev/null`)
- Passes tests (`r2r test/db/cmd/`)
- Can be merged independently
- Does not depend on later PRs (only on earlier ones in sequence)

---

## Testing Strategy

### Per-PR Verification
- `make -j > /dev/null` (clean compile)
- `r2r test/db/cmd/` (regression tests)
- `sys/lint.sh` (style check)

### Thread Safety Validation (after PR 5)
```sh
# Concurrent HTTP: verify no corruption
r2 -qc '=H 9090' /bin/ls &
for i in $(seq 10); do curl -s http://localhost:9090/cmd/pd%2010 > /tmp/out_$i & done
wait
# All outputs should be identical
md5sum /tmp/out_*
```

### Isolation Correctness (after PR 4)
```sh
# Background task doesn't corrupt main seek
r2 -qc 's 0x1000; &s 0x2000; sleep 1; s' /bin/ls
# Should print 0x1000 (background seek doesn't propagate)
```

### Stress Testing (after PR 21)
```sh
# Memory leak check
R2_DEBUG=1 r2 -qc '=H 9090' /bin/ls &
for i in $(seq 100); do curl -s http://localhost:9090/cmd/pd%2010 > /dev/null; done
kill %1
# Check no growth in RSS over iterations
```

### Config Isolation (after PR 31+)
```sh
# Parallel tasks with different configs
r2 -qc '&e asm.bytes=false; &pd 10; &e asm.bytes=true; &pd 10' /bin/ls
# Each output should reflect its own config

```

## Key Design Decisions

1. **TLS-based context lookup** (not core->task_ctx field): `r_core_ctx()` uses `r_core_task_self()` which looks up the calling thread's current task via `task_tls_current`. This avoids the race condition of two threads sharing the same core pointer.

2. **Lazy block reading** (`block_valid` flag): Seek only invalidates the block. The actual `r_io_read_at()` happens on first `r_core_ctx_get_block()` call. This avoids unnecessary I/O when commands seek multiple times before reading.

3. **Compatibility bridge** (`default_ctx->block = core->block`): During migration, SHARED isolation makes `ctx->block` point to the same buffer as `core->block`. Migrated code (`ctx->block`) and unmigrated code (`core->block`) see the same data with zero overhead.

4. **Write queueing for RAnal**: Background tasks queue analysis mutations instead of holding write locks during long operations. Main task flushes the queue periodically.

5. **r_cons_push/pop thread safety**: Not addressed by context isolation alone. The `ctx_stack` in RCons is a shared RList. ISOLATED mode clones the console context, sidestepping the shared stack issue for background tasks.
