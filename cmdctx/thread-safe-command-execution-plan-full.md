# Thread-Safe Command Execution: Vision & Roadmap

## Philosophy

This is a vision document for discussion. Individual PRs need their own careful planning and code review. The goal is NOT mechanical find-replace but code quality improvement at each step.

**Priority order** (per pancake's guidance):
1. Eliminate `core->block` usage (commands use local buffers + `r_io_read_at()`)
2. Eliminate `R_TH_LOCAL` globals (they're per-thread but not per-task)
3. Reduce `r_cons` calls (build `RStrBuf`, write once)
4. THEN introduce `RCoreTaskCtx` on top of clean code

Each change should make the code better independently, not just move tech debt around.

## Success Criteria

1. HTTP server handles 10 concurrent requests without data corruption
2. Background tasks (`&` prefix) execute with correct isolated state
3. No memory leaks in context creation/destruction cycle
4. Performance within 10% of current single-threaded execution
5. All existing tests pass throughout

---

## Phase A: Eliminate core->block (FIRST PRIORITY)

### Why first?

`core->block` is a shared mutable cache that makes parallel execution impossible. It must disappear entirely - not be wrapped in a ctx. Each command should own its own buffer via:

```c
// Before (shared mutable state):
r_print_hexdump (core->print, core->addr, core->block, core->blocksize, ...);

// After (local buffer, no shared state):
ut8 *buf = malloc (len);
r_io_read_at (core->io, addr, buf, len);
r_print_hexdump (core->print, addr, buf, len, ...);
free (buf);

// Or for small reads, stack buffer:
ut8 buf[256];
r_io_read_at (core->io, addr, buf, sizeof (buf));
```

### Guidelines for each file migration

Each file's migration is its own PR requiring individual planning. When migrating:

- Replace `core->block` reads with `r_io_read_at()` + local buffer
- Replace `core->blocksize` with explicit size from command args or config
- Identify and eliminate the seek-read-restore anti-pattern:
  ```c
  // REMOVE THIS PATTERN:
  ut64 orig = core->addr;
  r_core_seek (core, target, true);
  // use core->block
  r_core_seek (core, orig, true);

  // REPLACE WITH:
  ut8 *buf = malloc (len);
  r_io_read_at (core->io, target, buf, len);
  // use buf
  free (buf);
  ```
- Where commands build output incrementally, use `RStrBuf`:
  ```c
  RStrBuf *sb = r_strbuf_new ("");
  for (i = 0; i < n; i++) {
      r_strbuf_appendf (sb, "%02x", buf[i]);
  }
  r_cons_print (r_strbuf_get (sb));
  r_strbuf_free (sb);
  ```
- Use `r_str_newf()` instead of malloc+snprintf for string building
- Don't add new defensive checks that aren't needed

### Why each PR is safe (app works throughout)

- `r_io_read_at(core->io, addr, buf, len)` reads the same data as `core->block` (they both call into the same IO layer)
- `core->blocksize` still exists as a field until PR 12 - migrated commands can still reference it for default read size
- Seek still calls `r_core_block_read()` which refreshes `core->block` for unmigrated commands
- No command depends on another command's block content (each uses it as a cache of data at `core->addr`)
- PR 12 removal produces compile errors for any missed consumers - acts as a safety net

### PRs: core->block elimination

Each PR replaces `core->block` usage with local buffers + `r_io_read_at()`.
Each PR also converts r_cons_printf loops to RStrBuf where applicable.

#### PR 1: Small files (carg, casm, cbin, cfile, yank, cmd_cmp - 24 refs total)
- Simple replacements across small files. Low risk warmup.

#### PR 2: Small command files (cmd_seek, cmd_flag, cmd_meta, cmd_magic, cmd_debug, cmd_open, cmd_type - 18 refs total)
- Blocksize refs → config-based default size
- Small command file batch

#### PR 3: cmd_anal.inc.c + cmd.c (30 refs)
- Analysis commands + command infrastructure use local buffers
- `cmd_bsize` (`b` command) keeps blocksize as preference (core field or config)

#### PR 4: cio.c (18 refs)
- r_core_transform_op, r_core_write_op use local buffers
- r_core_block_read → deprecate (callers should use r_io_read_at directly)

#### PR 5: cmd_search.inc.c (23 refs)
- Search reads into local buffers

#### PR 6: disasm.c (23 refs + globals)
- Core disasm engine: careful migration, eliminate block swap patterns
- Remove Goaddr/Gsection globals (pass as params or make local)

#### PR 7: canal.c (25 refs + globals)
- Analysis reads → local buffers
- Remove esilbreak_last_read/data, ntarget globals

#### PR 8: cmd_write.inc.c (33 refs)
- Write operations: read into local buf, transform, r_io_write_at

#### PR 9: visual.c + vmenus.c + panels.c (62 refs total)
- Visual mode keeps using local buffers. Stays tied to core (not per-task)

#### PR 10: cmd_print.inc.c - part 1 (~85 refs)
- First half of print commands. Heavy RStrBuf conversion opportunity.

#### PR 11: cmd_print.inc.c - part 2 (~88 refs)
- Second half of print commands.

#### PR 12: Remove core->block/blocksize fields
- Remove `ut8 *block` and `ut32 blocksize` from `r_core_t`
- Remove `r_core_block_read()` and `r_core_block_size()` (or make them no-ops initially)
- Keep blocksize as user preference (config var `io.blocksize` or core field for default read size)
- Compile errors guide any remaining fixups

---

## Phase B: Eliminate R_TH_LOCAL globals

`R_TH_LOCAL` makes vars thread-local but NOT task-local. A thread running multiple tasks sequentially leaks state between them. These must become function-local, parameter-passed, or part of existing structs.

Note: disasm.c and canal.c globals are handled in Phase A PRs 6-7.

#### PR 13: Remaining core globals (cmd_anal.inc.c + misc)
- `oldregread`, `mymemxsr/w` → make function-local
- `const_color` → local variable
- `oldcwd` → function-local or core field
- `cur_name` → function-local or pass as param
- `mydb`/`osymbols` → remove (deprecated)
- `Gcore` → signal-safe alternative

#### PR 14: Console globals → RCons fields
- `Dietline D` → `cons->dietline`
- `RConsEditor G` → `cons->editor`
- `Glock` → `cons->pal_lock`

Note: `visual.c` vmenu globals stay in RCoreVisual (single instance, not per-task).

---

## Phase C: RCoreTaskCtx (on clean code)

After core->block is gone and globals eliminated, the ctx struct is simpler:

```c
typedef enum {
	R_CORE_TASK_ISOLATION_SHARED = 0,   // Legacy single-threaded
	R_CORE_TASK_ISOLATION_SNAPSHOT = 1,  // RConfigHold save/restore (HTTP server)
	R_CORE_TASK_ISOLATION_ISOLATED = 2   // Full config clone (parallel tasks)
} RCoreTaskIsolation;

typedef struct r_core_task_ctx_t {
	struct r_core_t *core;
	RCoreTaskIsolation isolation;
	// per-task seek
	ut64 addr;
	ut32 blocksize;          // default read size preference
	// config isolation
	RConfigHold *config_hold; // SNAPSHOT: save/restore on same RConfig
	RConfig *config;          // ISOLATED: full clone
	bool config_dirty;
	// propagation
	bool propagate_seek;
	bool propagate_config;
	// console
	RConsContext *cons_ctx;
	// command state
	int cmd_depth;
	int cmd_depth_max;
	char *lastcmd;
	bool tmpseek;
	bool incomment;
	int cmdrepeat;
} RCoreTaskCtx;
```

Note: No `ut8 *block` field. Commands allocate their own buffers.
Note: No visual mode state. Visual is tied to core (only one visual mode).

### Config isolation via RConfigHold

RConfigHold (in `libr/config/hold.c`) already implements per-key snapshot/restore:
- `r_config_hold_new(cfg)` → create hold
- `r_config_hold(h, "key1", "key2", ..., NULL)` → snapshot values
- `r_config_hold_restore(h)` → restore values (triggers setter callbacks)
- `r_config_hold_free(h)`

For SNAPSHOT isolation: snapshot all asm.*/scr.*/cfg.* keys at task entry, restore at exit.
For ISOLATED: use `r_config_clone()` for full independence (true parallel).

**Potential RConfigHold improvements needed:**
- Currently requires explicit key names. May need `r_config_hold_all()` or pattern-matching (`r_config_hold_prefix(h, "asm.")`) to snapshot relevant keys without listing each one.
- Not thread-safe - fine for SNAPSHOT (sequential) but can't be used for true parallel.
- Setter callbacks on restore may have side effects - need audit of which callbacks are safe to call from non-main thread.

### Context lookup (TLS-based)

```c
R_API RCoreTaskCtx *r_core_ctx(RCore *core) {
	R_RETURN_VAL_IF_FAIL (core, NULL);
	RCoreTask *task = r_core_task_self (&core->tasks);
	if (task && task->ctx) {
		return task->ctx;
	}
	return core->default_ctx;
}
```

Uses existing `task_tls_current` (task.c:6) for per-thread task lookup.
NOT `core->task_ctx` (race condition: two threads sharing same core).

#### PR 15: RCoreTaskCtx infrastructure + task wiring
- Define struct + enum in `libr/include/r_core.h`
- Implement `r_core_task_ctx_new()`, `_free()`, `_commit()`, `r_core_ctx()` in new `libr/core/ctx.c`
- Add `RCoreTaskCtx *default_ctx` to `r_core_t`, init in `r_core_new()`
- Add `RCoreTaskCtx *ctx` and `RCoreTaskIsolation isolation` to RCoreTask
- In `task_run()`: create ctx before cmd, commit+free after
- Main task: SHARED, background tasks (`&`): SNAPSHOT
- Build system: add `ctx.o` to Makefile, `'ctx.c'` to meson.build

---

## Phase D: Locking

#### PR 16: Enable R_CRITICAL + lock seek
- `libr/include/r_userconf.h:17` - change `#define R_CRITICAL_ENABLED 0` to `1`
- `libr/core/cio.c` - wrap `r_core_seek()` and `r_core_seek_delta()` with R_CRITICAL

#### PR 17: RThreadRWLock + IO/Flags rwlocks
- `libr/include/r_th.h` - add RThreadRWLock type
- `libr/util/thread_rwlock.c` - implementation
- `libr/io/io.c` - rdlock in `r_io_read_at()`, wrlock in `r_io_write_at()`
- `libr/flag/flag.c` - rdlock in get, wrlock in set/unset

#### PR 18: Analysis + Bin rwlocks + write queueing
- `libr/include/r_anal.h` - add lock + `RList *pending_writes`
- `libr/anal/anal.c` - rdlock reads, queue writes from non-main tasks
- `r_anal_pending_flush()` applies queued writes under wrlock
- `libr/bin/bin.c` - rdlock in get, wrlock in set operations

---

## Phase E: Console isolation

r_cons coupling should already be reduced during Phase A (RStrBuf conversions).

#### PR 19: RCons singleton removal
- `libr/cons/cons.c` - remove `static RCons *I`
- Internal functions get `RCons *cons` parameter
- `r_cons_singleton()` → deprecated wrapper
- Fix `r_cons_push()`/`r_cons_pop()` thread safety (shared RList ctx_stack has no locking)
- Largest single PR - may need splitting

---

## Phase F: HTTP server migration

After phases A-D, the HTTP server cleanup is straightforward:

#### PR 20: Remove HTTP block-swap, use task isolation
- Remove manual block/addr save/restore in `rtr_http.inc.c` (lines 201-232, 275-281)
- HTTP requests run as SNAPSHOT tasks (config save/restore, per-task seek)
- No block swap needed (commands allocate own buffers)
- Remove config clone (`newcfg`) - handled by task isolation

Test: `for i in $(seq 10); do curl -s http://localhost:9090/cmd/pd%2010 & done; wait`

---

## Summary: 20 PRs across 6 phases

| Phase | PRs | Description | Risk |
|-------|-----|-------------|------|
| A | 1-12 | Eliminate core->block (per file, code quality improvements) | Low each |
| B | 13-14 | Eliminate R_TH_LOCAL globals | Low each |
| C | 15 | RCoreTaskCtx + task wiring | Medium |
| D | 16-18 | Locking (R_CRITICAL, rwlocks, write queue) | Medium |
| E | 19 | RCons singleton removal | High |
| F | 20 | HTTP server migration | Low (after A-D) |

Each PR: compiles cleanly, passes tests, mergeable independently.

---

## Key Design Decisions

1. **Eliminate before abstracting**: Remove core->block entirely first. Don't wrap it in a ctx struct - that's moving tech debt. Each command should own its memory.

2. **RConfigHold for snapshots**: Existing mechanism handles save/restore elegantly. Only clone for full parallel isolation.

3. **TLS-based context lookup** (not core->task_ctx field): Uses `r_core_task_self()` to find calling thread's task. No race between threads sharing same core.

4. **Visual mode stays on core**: Only one visual mode instance. No per-task visual state.

5. **RStrBuf for output**: Commands build strings, write once. Reduces r_cons coupling, makes per-task output collection trivial.

6. **Write queueing for RAnal**: Background tasks queue mutations. Main task flushes. No long write-lock holds.

7. **Individual PR planning**: This document is the vision. Each PR needs its own analysis of the specific file, code patterns to fix, and quality improvements to make.

---

## Testing Strategy

### Per-PR
- `make -j > /dev/null` (clean compile)
- `r2r test/db/cmd/` (regression)
- `sys/lint.sh` (style)

### After PR 12 (Phase A complete)
```sh
# Verify no core->block references remain
grep -r 'core->block' libr/core/ --include='*.c' | grep -v '//' | wc -l
# Should be 0
```

### After PR 16-18 (Phase D locking)
```sh
# Concurrent HTTP
r2 -qc '=H 9090' /bin/ls &
for i in $(seq 10); do curl -s http://localhost:9090/cmd/pd%2010 > /tmp/out_$i & done
wait
md5sum /tmp/out_*  # all identical
```

### After PR 15+16 (Phase C+D: ctx + locking)
```sh
# Background task isolation
r2 -qc 's 0x1000; &s 0x2000; sleep 1; s' /bin/ls
# Should print 0x1000 (background seek doesn't propagate)
```
