# ESIL rstrs follow-up — pending simplifications

## History

- `093f6b1` "Avoid copies!": introduced upfront buffer rewrite so every parse
  pre-NUL-separates the input, letting `r_esil_push_strs` become a single
  store for parse-time tokens.
- `7388ac9a20` "simplify": collapsed `parm_*_strs` into static-inline wrappers.
  Regressed perf at `-O0` (default build) because `static inline` doesn't
  inline at `-O0` — wrapper indirection turned into real function calls.
- `f94e26b46a` "Optimize down 4.8" / `7f2f7be08a` "less code less time":
  - Restored the slice-native `parm_*_strs` as real R_API functions.
  - Extracted `r_strs_u64hex` into `libr/util/strs.c` (clean, reusable).
  - Restored open-coded brace checks in `runword_strs`.
  - Dropped redundant `!r_strs_empty()` in op handlers where
    `r_esil_get_parm_strs` already handles empty slices.
  - **Removed the upfront buffer rewrite**: parse now slices in-place off the
    caller's input, `push_strs` copies to arena on the fly. This is the same
    architecture as the pre-`093f6b1` "Heapless strings" design — the rewrite
    in `093f6b1` turned out to be net slower for the common aae workload.
  - Inlined `stack_arena_reserve` into push/pushnum.
  - Collapsed `r_esil_get_parm_strs` to skip the `parm_size_strs` frame.
  - Dropped `R_RETURN_VAL_IF_FAIL` from `r_esil_pop_strs` hot path.

## Current perf (`r2 -q -c '?t aae' /usr/bin/vim`, pinned CPU, 25 runs each)

- **master** (strdup-based): 5.380s (5.304-5.519)
- **enod branch, current**: 5.278s (5.176-5.442)

Net vs. `093f6b1` baseline: **-48 LOC** (147 ins / 195 del) AND faster than
master by ~100ms on the measured machine.

## Pending ideas — ordered by expected impact

### 1. Stack slot: `char*[]` instead of `RStrs[]`

**Biggest remaining win candidate.** Today `RStrs stack[stacksize]` uses 16
bytes/slot (two pointers). Master's `char *stack[]` uses 8 bytes/slot.
Every pop/push reads/writes twice as much data; at stacksize=4096 the stack
is 64KB vs master's 32KB.

Because `r_esil_push_strs` always NUL-terminates copied slices (the arena
copy pads with `\0`), any pop can use the `a` pointer directly as a
`const char *`. The `b` pointer is only used for 7 `r_strs_empty` checks in
`esil_ops.c`, all of which could check `*s == '\0'` instead.

Proposal: change `REsil.stack` from `RStrs*` to `char**`. Push stores `dst`.
Pop returns `char*`. Op handlers that need length call `strlen(s)` on demand
(rare). `r_esil_pop_strs` becomes a wrapper: `RStrs{s, s + strlen(s)}`.

Expected: cuts the slot load/store in half, and drops the struct-return ABI
cost which at `-O0` costs register pair rax:rdx bookkeeping.

Risk: any caller using `r_strs_len(popped)` without re-checking would get
strlen-on-demand. Not a correctness issue but repeated strlens could undo
the win. Audit the 7 `r_strs_len` call sites first.

### 2. Avoid the double `r_reg_get` per register parm

`r_esil_get_parm_type_strs` calls `not_a_number` → `r_reg_get` + `r_unref`
just to classify the parm as REG. Then `r_esil_reg_read` internally does
another `r_reg_get` to actually read. Two hashtable lookups per register
parm access; both master and this branch share the defect.

Fix: cache the `RRegItem *` in the classifier path. Return PARM_REG *with*
the `RRegItem` already held; `r_esil_get_parm_strs` reads the value from
the cached item and drops the ref once.

Requires refactoring `r_esil_reg_read` to accept a pre-resolved item, or a
side-channel (`esil->last_reg_item`) — the latter keeps the public API
stable. Expected: noticeable since register parms dominate.

### 3. Arena placement

`stack_buf` is `malloc`'d. At stacksize=4096, cap = 4096 × 32 = 128KB.
That's well beyond L1 (32KB) — arena accesses can miss into L2/L3 for
long expressions. Each `r_esil_parse` resets `stack_buf_len=0` so short
parses reuse the first few bytes — cache-warm.

For consistency with master's fastbin behavior, consider capping the arena
smaller (e.g. stacksize × 8 = 32KB) and falling back to heap-alloc per slot
only on overflow. Typical aae ESIL expressions push <200 bytes total.

### 4. Binop macro in `esil_ops.c`

`esil_and`/`or`/`xor`/`add`/`sub`/`mul` are structurally identical.
Collapse with a macro; ~50 LOC saved, zero perf impact. Same for the
`*eq` family once the strs migration reaches them.

```c
#define ESIL_BINOP(NAME, OP) \
	static bool esil_##NAME(REsil *esil) { \
		ut64 s, d; \
		const RStrs dst = r_esil_pop_strs (esil); \
		const RStrs src = r_esil_pop_strs (esil); \
		if (r_esil_get_parm_strs (esil, src, &s) \
				&& r_esil_get_parm_strs (esil, dst, &d)) { \
			return r_esil_pushnum (esil, d OP s); \
		} \
		R_LOG_DEBUG ("esil_" #NAME ": invalid parameters"); \
		return false; \
	}
ESIL_BINOP (and, &) ESIL_BINOP (or, |) ESIL_BINOP (xor, ^)
ESIL_BINOP (add, +) ESIL_BINOP (sub, -) ESIL_BINOP (mul, *)
```

### 5. `r_esil_pop` migration

The AITODO at `libr/esil/esil.c:651` remains: ~25 `r_esil_pop` call sites
in `esil_ops.c` still use the NUL-term API. Migrating them to
`r_esil_pop_strs` unblocks idea #1 (char* stack) and lets
`r_esil_pop` retire. Mostly mechanical, similar to what was done for the
binops.

### 6. Op lookup fast path

`r_esil_get_op_strs` uses a custom slice hash (DJB-like) + memcmp via HtPP.
For typical ESIL ops (1-3 chars), a perfect hash or tiny switch on
`wlen + w.a[0]` could skip HT traversal entirely for the top 20 ops. Worth
profiling first — the current HT is probably ~10ns/op at -O0, and there are
millions of op lookups per aae.

## Design notes for future `rstrs` work (-O0 gotchas)

- `static inline` is NOT inlined at `-O0` (default build). A `static inline`
  wrapper whose body calls another function costs **two** call frames at
  `-O0`. Prefer real out-of-line functions or use `__attribute__((always_inline))`
  for trivially-bodied helpers that must inline.
- For short literal tokens on hot paths (ESIL braces `}`, `}{`, `?{`),
  direct byte compares beat any helper at `-O0`.
- When extracting reusable primitives to `libr/util/strs.c`, keep the body
  small (one function, shallow) — `r_strs_u64hex` is a good shape.
- Struct return of 16-byte `RStrs` uses register pair (rax:rdx) on SysV
  AMD64. At `-O0`, gcc still emits extra moves. If slot storage can be
  8-byte (idea #1), do it — saves ABI overhead on every pop.
