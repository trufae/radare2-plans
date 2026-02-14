# Task System Race Condition Analysis and Fix Plan

## Problem Summary

The radare2 task system suffers from race conditions when running background tasks (`&` command) that cause heap-use-after-free and double-free errors detected by AddressSanitizer.

## Root Cause Analysis

### The Fundamental Issue

The core problem is that `core->cons` (RCons) is a **global shared resource** accessed by all tasks. When background tasks run commands via `r_core_cmd_str()`, they modify `core->cons->context` through `r_cons_push()`/`r_cons_pop()` operations, which races with the main thread accessing the same console.

### Crash Patterns Identified

From analyzing 5 crash logs, two distinct patterns emerged:

#### Pattern 1: heap-use-after-free in `r_cons_context_clone`
```
Thread T2: r_cons_context_clone() reads ctx->break_stack
Thread T1: already freed context via r_cons_pop() -> r_cons_context_free_internal()
```

#### Pattern 2: double-free in `cons_palloc`
```
Thread T0 (main): r_cons_printf() -> cons_palloc() -> realloc(C->buffer)
Thread T1/T2: r_cons_pop() already freed C->buffer
```

### Call Flow Analysis

When a background task runs:
```
r_core_task_submit()
  -> r_core_task_enqueue()
    -> r_th_new(task_run_thread)
      -> task_run()
        -> r_core_cmd_str()  // Uses core->cons (SHARED!)
          -> r_cons_push()   // Modifies cons->context
          -> r_core_cmd()    // Executes command
          -> r_cons_pop()    // Frees cloned context, restores parent
```

Meanwhile, the main thread might be:
```
r_core_task_list()
  -> r_cons_printf()  // Accesses cons->context
    -> cons_palloc()  // Uses cons->context->buffer
```

The race occurs because `cons->context` pointer can change (and the old context be freed) between when a thread reads it and when it uses the data.

## Solutions Evaluated

### Solution 1: Global Locking (Quick Fix - Implemented)

**Approach**: Make `cons->lock` recursive and protect all functions that access `cons->context`:
- `r_cons_push()` / `r_cons_pop()`
- `r_cons_printf_list()`
- `r_cons_break_push()` / `r_cons_break_pop()`

**Changes made**:
1. Changed `cons->lock = r_th_lock_new(false)` to `r_th_lock_new(true)` (recursive)
2. Added `r_th_lock_enter/leave` around critical sections in:
   - `r_cons_push()`
   - `r_cons_pop()`
   - `r_cons_printf_list()`
   - `r_cons_break_push()`
   - `r_cons_break_pop()`

**Problems with this approach**:
- Locking is a band-aid, not a proper fix
- Hard to ensure ALL code paths are protected
- Performance impact from lock contention
- The shared mutable state (`cons->context`) remains fundamentally problematic
- Context can still change between operations even with per-function locking
- Does not scale well as more functions need protection

### Solution 2: Per-Task Console Context (Proper Fix - Recommended)

**Approach**: Each task should have its own isolated console state, not share `core->cons`.

The task system already has infrastructure for this:
```c
typedef struct r_core_task_t {
    RConsContext *cons_context;  // Already exists but underutilized!
    // ...
} RCoreTask;
```

And there's a disabled `CUSTOMCORE` option in task.c:
```c
#define CUSTOMCORE 0

static RCore *mycore_new(RCore *core) {
#if CUSTOMCORE
    RCore *c = R_NEW (RCore);
    memcpy (c, core, sizeof (RCore));
    c->cons = r_cons_new ();  // Each task gets its own console!
    return c;
#else
    return core;  // Currently just returns shared core
#endif
}
```

## Recommended Implementation Plan

### Phase 1: Task-Local Console Context API

Create new APIs that accept a task context parameter instead of using global `core->cons`:

```c
// New task-aware command execution
typedef struct r_core_task_ctx_t {
    RCore *core;
    RConsContext *cons_ctx;  // Task-local console context
    RStrbuf *output;         // Task-local output buffer
} RCoreTaskCtx;

// Alternative to r_core_cmd_str that uses task context
R_API char *r_core_task_cmd_str(RCoreTaskCtx *ctx, const char *cmd);

// Alternative to r_cons_printf that writes to task context
R_API void r_core_task_printf(RCoreTaskCtx *ctx, const char *fmt, ...);
```

### Phase 2: Modify task_run() to Use Task Context

```c
static RThreadFunctionRet task_run(RCoreTask *task) {
    RCoreTaskCtx ctx = {
        .core = task->core,
        .cons_ctx = task->cons_context,  // Use task's own context
        .output = r_strbuf_new("")
    };
    
    // Execute command with isolated context
    char *res = r_core_task_cmd_str(&ctx, task->cmd);
    task->res = res;
    
    // No shared state modified!
}
```

### Phase 3: Gradual Migration

For commands that need to work in background tasks:

1. Identify which `r_cons_*` calls they make
2. Create task-aware variants that use `RCoreTaskCtx`
3. Route background task execution through new APIs
4. Keep old APIs for main thread / interactive use

### Phase 4: Output Aggregation

When a background task completes, its output needs to be presented:

```c
// Task stores output in its own buffer
task->res = r_strbuf_drain(ctx.output);

// Main thread can safely access task->res after task completes
// via &=<tid> command
```

## Files That Need Modification

### Core Changes
- `libr/core/task.c` - Task execution and context management
- `libr/core/cmd.c` - `r_core_cmd_str()` and related functions
- `libr/include/r_core.h` - New API declarations

### Console Changes
- `libr/cons/cons.c` - Task-aware console functions
- `libr/include/r_cons.h` - New API declarations

### Command Handlers (as needed)
- `libr/core/cmd_*.inc.c` - Commands that need to work in background

## Minimal Fix for Current Tests

For the immediate goal of making `r2r test/db/cmd/task` pass consistently, the locking approach provides stability. However, it's not a complete solution.

The test expects specific output format that assumes tasks are queued but not yet running when `&` command returns. The current implementation starts tasks immediately, leading to race conditions in output ordering.

A minimal semantic fix would be:
1. Keep the locking for safety
2. Ensure `&` command waits for task to be fully initialized before returning
3. Or update test expectations to handle async behavior

## Conclusion

The proper fix requires architectural changes to isolate task state. The locking approach is a temporary measure that reduces crash frequency but doesn't eliminate the fundamental shared-state problem.

The recommended path forward:
1. **Short-term**: Keep locking fixes for stability
2. **Medium-term**: Implement `RCoreTaskCtx` API for critical paths
3. **Long-term**: Migrate all task-executed commands to use isolated context

This ensures:
- No shared mutable state between tasks
- Each task owns its console context completely
- Main thread and background tasks never race on console state
- Clean separation of concerns
