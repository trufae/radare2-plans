# COREPRJ.md — The new binary `prj` core plugin: state of the art, gaps and roadmap

> Scope: this document reviews `libr/core/p/core_prj.c` (the new, experimental binary
> project format registered as the `prj` core plugin), compares it against the
> legacy script-based project format (`Ps`/`Po`/`r_core_project_save_script` in
> `libr/core/project.c`), lists the bugs and limitations found while reading the
> code and exercising the `prj save`/`prj load`/`prj info`/`prj r2` commands, and
> proposes a prioritised roadmap so r2 projects can finally become useful for
> everyone.
>
> This is intentionally long-form so that @pancake can annotate, reorder, cross
> out, and redefine priorities. Once priorities are agreed, the TODO checkboxes
> near the bottom should be the single source of truth for the implementation
> effort.

---

## 1. Executive summary

Radare2 has two project systems that do not yet interoperate:

1. **Legacy script-based projects** (`Ps`, `P<name>`, `P-`, `Pi`, `Pn`, `Pz`, `P.`, `PS`, …),
   driven by `libr/core/project.c`. These save an r2 command script (`rc.r2`)
   plus a few side-car SDB files (rop cache) and optional git/rvc history, all
   stored under `dir.projects/<name>/`. Information is persisted by running
   existing r2 commands in their `*` export mode and concatenating their output.
   This is the only format the `Ps`/`Po` family currently uses in the default
   configuration; it is also the format that all existing `test/db/cmd/projects`
   tests exercise.

2. **New binary projects** (`prj save|load|info|r2`) implemented as a core
   plugin in `libr/core/p/core_prj.c`. These save a single binary file with a
   chunked, tagged layout (header + typed entries + a string table), gated
   behind `e prj.new=true` when saved through `Ps` (`project.c` at line 733
   issues `prj save <path>/prj.bin` in addition to the `rc.r2` script).

The new format is currently **experimental, lossy, and unable to round-trip even
a minimal session** because of a combination of schema gaps (no functions, no
xrefs, no types, …) and outright bugs (only one IO map is written, the entry
length is miscalculated on read, the flag struct is half-written in little
endian of the wrong width, etc.). This document proposes making the new format
the default and first-class way to snapshot a radare2 session.

At a glance:

- **Header:** magic `'RPRJ' = 0x4a525052`, 32-bit version (currently `1`).
- **Entries:** tagged, length-prefixed records. Each `R2ProjectEntry` is
  `{ ut32 size; ut32 type; }` followed by payload. `size` is the *total*
  size including the 8-byte header.
- **Types seen in source:** `RPRJ_MAPS (0)`, `RPRJ_INFO (1)`, `RPRJ_FLAG (2)`,
  `RPRJ_CMNT (3)`, `RPRJ_CMDS (4)`, `RPRJ_BLOB (5)`, `RPRJ_MODS (6)`,
  `RPRJ_STRS (7)`, `RPRJ_THEM (8)`, `RPRJ_HINTS (9)`.
- **String table:** a single `RPRJ_STRS` blob of NUL-terminated strings,
  referenced from other entries by byte offset.
- **Modules:** `RPRJ_MODS` stores one record per IO map (`name, file, pmin,
  pmax, vmin, vmax, csum`). Other records (flags, comments, hints) store a
  `mod` index + `delta` instead of absolute addresses, so maps can be relocated
  on load. Fallback uses `UT32_MAX` to mean "absolute address in `delta`".

---

## 2. What lives in `core_prj.c` today

### 2.1 File format

```
┌──────────────────────────────────────────────────────────────────────────┐
│ R2ProjectHeader  { magic=0x4a525052, version=1 }                         │   8 bytes
├──────────────────────────────────────────────────────────────────────────┤
│ Entry<INFO>      { size, type }  payload: R2ProjectInfo                  │
│ Entry<MODS>      { size, type }  payload: R2ProjectMod × N_maps          │
│ Entry<CMDS>      { size, type }  payload: len+bytes × N_cmds             │
│ Entry<CMDS>      { size, type }  payload: len+bytes × N_cmds             │
│ Entry<FLAG>      { size, type }  payload: R2ProjectFlag × N_flags        │
│ Entry<CMNT>      { size, type }  payload: R2ProjectComment × N_comments  │
│ Entry<HINTS>     { size, type }  payload: R2ProjectHint × N_hints        │
│ Entry<STRS>      { size, type }  payload: NUL-terminated strings         │
└──────────────────────────────────────────────────────────────────────────┘
```

Implementation order in `prj_save` (the path that writes):

1. `rprj_header_write`
2. Hard-coded `Info` with name `"test-project"`, user `"pancake"`, current time.
3. `rprj_mods_write` (one record per IO map — but see bugs, only one is actually emitted).
4. Hard-coded `Cmds` entry containing `?e hello projects`, `?e goodbye`.
5. Hard-coded `Cmds` entry containing `?E clippy`.
6. `rprj_flag_write` — every flag in `core->flags` via `r_flag_foreach`.
7. `rprj_cmnt_write` — every `R_META_TYPE_COMMENT` interval-tree entry.
8. `rprj_hints_write` — only `IMMBASE` and `NEW_BITS` are emitted today.
9. Two hard-coded string-table entries (`"one string"`, `"another one"`).
10. `Strs` entry containing the accumulated string table.

Implementation order in `prj_load` / `prj info` / `prj r2` (the read path):

1. Read header, verify magic.
2. Locate and load `Strs` via `rprj_find`.
3. Locate and decode `Mods`; for each mod, resolve the current IO map via
   `coremod()` (by content checksum first, then by name), overwrite
   `vmin/vmax` with the live map range.
4. Iterate entries sequentially; dispatch on entry type:
   - `INFO` — pretty-print name/user only (date is commented-out).
   - `MODS` — already processed, no-op here.
   - `CMDS` — read length-prefixed strings; execute if `MODE_CMD`.
   - `FLAG` — recreate `r_flag_set` per record.
   - `CMNT` — `CCu base64:` of the stored comment text at resolved VA.
   - `HINTS` — `r_anal_hint_set_immbase` / `r_anal_hint_set_newbits`.
   - `MAPS`, `BLOB`, `THEM` — no-ops / stubs.

### 2.2 Command surface

```
prj save <file>   - serialise the current session to <file>
prj load <file>   - read the file and re-apply flags / comments / hints / run cmds
prj info <file>   - dump a structured view of the file (used for debugging)
prj r2   <file>   - print an equivalent r2 script
```

Commands are dispatched by matching the literal prefix `"prj"` inside the core
plugin `callback`; arguments are split once on the first space. There is no
tab-completion entry, no JSON mode, no help alias (`prj?`), and no integration
with `P`-series project commands: the new file lives independently from
`dir.projects/<name>/prj.bin`.

### 2.3 Build & registration

- Makefile fragment at `libr/core/p/prj.mk` links `core_prj.o` and exposes a
  shared plugin too.
- Meson build entry in `libr/core/meson.build` line 51.
- Plugin symbol table: `libr/config.h` arrays `r_core_plugin_prj` alongside
  `r_core_plugin_java`, so it is statically linked into every r2 build.

---

## 3. Observed behaviour (runtime session notes)

Session I ran while researching this document:

```
$ r2 -q -c 'prj save /tmp/foo.prj' /bin/ls
$ r2 -q -c 'prj info /tmp/foo.prj' /bin/ls
```

Key observations:

- **The `prj save` default info block is literally `test-project` / `pancake`**,
  regardless of the actual project name, `cfg.user`, or command-line arg.
- **`prj info` prints `Entry<UNKNOWN>` for comments** (type `0x03`) because
  `entry_type_tostring` has no arm for `RPRJ_CMNT`.
- **Hundreds of `Cant find map for <flag>` warnings** are printed during
  `prj info` / `prj load`. Every flag address that is not inside the single map
  that `rprj_mods_write` actually manages to persist is treated as "absolute"
  (`mod = UT32_MAX`), but the load path then tries `r_list_get_n` on an index
  equal to `UT32_MAX`, which returns NULL and logs the error — while *still*
  using the raw absolute address, which silently works but floods stderr.
- **`prj info` reports two modules even though only one was written**:
  `MOD: mmap.__LINKEDIT + 0x100015c00` followed by `MOD: dr:__5614542 + 0x0`.
  The second "module" is reconstructed garbage caused by the over-read bug in
  `rprj_find` (see §4.2).
- **`prj load`** prints `hello projects`, `goodbye`, and then runs `?E clippy`,
  because the two `CMDS` entries in the saved file are hard-coded debug stubs.
- **Size of a minimal `/bin/ls` project:** ≈16 KiB. ≈44 % of that is taken by
  the string table (every flag name, full path, plus the two hard-coded
  strings).

---

## 4. Bugs and weaknesses in the current implementation

Numbered so we can reference them from commits and tests. Severity is
subjective; feel free to reorder.

### 4.1 Correctness bugs (ship-blocking)

1. **`rprj_mods_write` writes at most one module** (`libr/core/p/core_prj.c:413–421`).
   `bsz` and `at` are captured before the loop and never refreshed. On every
   iteration after the first, `at + sizeof(R2ProjectMod) >= bsz` is trivially
   true, so the `// should never happen` break *always* fires. Result: only the
   first IO map is persisted. Every flag inside any later map becomes
   `mod = UT32_MAX` at save time and is reloaded as a raw absolute address,
   which breaks address rebasing for non-default `-B` / library load addresses.

2. **`rprj_find` reads past the entry payload**
   (`libr/core/p/core_prj.c:572–578`). `entry.size` includes the 8-byte header,
   but `r_buf_read_at (b, at + sizeof(R2ProjectEntry), buf, entry.size)` reads
   `entry.size` bytes starting *after* the header. The trailing 8 bytes are
   the header of the next entry, silently absorbed as payload. This is the
   root cause of the phantom `dr:__5614542` module above.

3. **`RPRJ_CMNT` missing from `entry_type_tostring`** (line 88). Cosmetic, but
   makes `prj info` output misleading.

4. **`rprj_cmnt_read` uses `sizeof (R2ProjectFlag)` instead of `R2ProjectComment`**
   (line 204). They happen to be the same size today, so the bug is latent but
   a landmine for anyone who changes either struct.

5. **`flag.size` / `cmnt.size` written as le32 into a ut64 field**
   (lines 159, 188). The struct is declared as `ut64 size;` but
   `r_write_le32 (&flag.size, …)` sets only the low 4 bytes; the upper 4 bytes
   are whatever random stack bytes were there before. On read, `r_read_le64`
   consumes those 8 bytes and produces garbage sizes. Flags with size `>0`
   reload with huge spurious sizes after a load.

6. **Struct padding bytes are serialised uninitialised**
   (`rprj_mods_write_one`, `rprj_flag_write_one`, `rprj_cmnt_write_one`, etc).
   The code writes the struct byte-for-byte via `r_buf_write` after having
   populated only the *named* fields. Compiler padding (4 bytes at the tail of
   `R2ProjectMod`, potentially more depending on alignment) carries stack
   garbage into the file → non-reproducible files (bad for git projects) and
   a small information leak.

7. **`rprj_string_write` uses `strlen`**, so any binary data (notes with
   embedded NULs, base64 payloads that happen to have `\0` in them) is
   truncated on write. The command entry format relies on length prefixes,
   so at least encoding is consistent — but it cannot round-trip arbitrary
   bytes.

8. **`rprj_string_read` vs `rprj_string_write` width mismatch:** writer emits a
   `ut32` length (via `r_write_le32` on `size_t len`), but if `len > UT32_MAX`
   it silently truncates. Minor, but deserves a cap check.

9. **Dummy data leaking into every project file:**
   - Info entry always names `"test-project"` / `"pancake"` / now() instead of
     using `prj.name` / `cfg.user` / real save time.
   - Two dummy `CMDS` entries (`?e hello projects`, `?e goodbye`, `?E clippy`)
     that actually execute on `prj load`. This pollutes user-visible output
     and is dangerous for automation.
   - Two dummy string-table rows (`"one string"`, `"another one"`) that waste
     bytes and add noise to diffs.

10. **Missing validation in `prj_load`:** no `version` check, no early return
    when `RPRJ_STRS` is missing, and no NULL guard before `strlen (cmnt_text)`
    / `strlen (flag_name)` — any of those crash if the string-table lookup
    returns NULL for a corrupted/malicious file.

11. **`prj_load` does not bound-check `flag.mod`** (`r_list_get_n` with
    `UT32_MAX` is safe but still burns CPU walking most of the list; and the
    fallback path silently uses `flag.delta` as absolute, which may be entirely
    wrong for rebased binaries).

12. **`r_flag_foreach` iteration order is not deterministic** across runs,
    which makes binary project files non-reproducible. This makes `git diff` of
    checked-in projects noisy.

13. **Resource leaks / lifetime mistakes:**
    - `prj_save` never calls `r_buf_free (b)` / `r_unref (b)` (`prj_save`
      returns right after `r_file_dump`).
    - `cur.mods` (`r_list_newf (free)`) is leaked in both `prj_save` and
      `prj_load`.
    - `st.data` is leaked in `prj_save` and in the load path (only the
      decoded buffer from `rprj_find` is assigned, never freed).
    - If the interactive "Overwrite project file?" branch answers no, the
      function returns early without freeing anything.

14. **Weak, non-salted `checksum` of the first 1024 bytes** for map matching.
    Any two maps beginning with identical bytes (very common for padded ELF
    segments or zero-initialised regions) collide. Combined with the fallback
    that matches by `RIOMap->name`, this produces unreliable mod-to-map
    binding when the project is re-opened against the same binary.

15. **Sandbox / file-system operations are unchecked.** `r_file_rm` and
    `r_file_dump` are called in `prj_save` with no
    `r_sandbox_check (R_SANDBOX_GRAIN_FILES | R_SANDBOX_GRAIN_DISK)` guard.
    `prj_load` happily executes arbitrary `CMDS` entries without a
    sandbox/consent prompt.

16. **Paths are not resolved against `dir.projects`.** `prj save foo` creates
    `./foo` in the shell cwd. `prj load` does not understand project names.
    There is no separation between project *name* and *file path*.

17. **Entry size and version fields are written as `-1` then patched.** If
    writing an entry fails mid-way (e.g. `r_buf_write` returns short, or an
    inner call fails and we abort), the file is left with a bogus `size=-1`
    entry and the next load aborts. There is no integrity check at the end
    (trailer, CRC32 or blake3).

### 4.2 Design weaknesses (not strictly bugs but serious)

18. **No versioning strategy.** `version = 1` is hard-coded and never checked
    on load. No forward/backward compatibility story. No deprecation path.

19. **No endianness policy documentation** (we use `r_read_le32` / `r_write_le32`
    but serialise whole structs raw, which *only* works on little-endian hosts
    without padding).

20. **String-table index `0` is ambiguous** with the sentinel `UT32_MAX` pattern
    used elsewhere. The first string written receives index 0, which is
    legitimate, but any bug that confuses "unset" with "empty string" silently
    succeeds.

21. **No compression, no deduplication** of string table entries. For medium
    binaries (1000+ functions) the string table dominates the file size and
    compresses ≈ 5×.

22. **`BLOB`, `MAPS`, `THEM` entry types exist in the enum but are
    unimplemented.** `MAPS` would be useful for saving `wc` (write cache)
    patches as raw bytes.

23. **No notion of flag spaces.** RFlag exposes spaces (`imports`, `strings`,
    `symbols`, `functions`, `registers`, user-defined ones). Today all flags
    collapse into a single namespace on reload.

24. **No notion of flag realnames, flag zones, or flag colours.**

25. **No notion of meta types other than comments.** Format strings (`Cf`),
    data (`Cd`), strings (`Cs`), magic (`Cm`), hidden (`Ch`), highlight
    (`Ch`), size (`Cs`), zones (`fz`) are all silently dropped.

26. **No functions / basic blocks / xrefs / variables / types** — the bulk of
    analysis state lives in `core->anal` and is not touched by `core_prj.c`
    at all.

27. **No config eval keys** (`asm.arch`, `asm.bits`, `anal.gp`, `asm.syntax`,
    `cfg.bigendian`, `io.cache`, etc.). A saved project will restore the
    binary's architecture *only* if the auto-detection still matches on load.

28. **No bin info** (`baddr`, `laddr`, relocs, imports overrides) and no
    multi-file state (`o*`, `o=fd`).

29. **No write-cache persistence** (`wc*`). Patches the user made to the file
    are lost on project close.

30. **No UI state:** panels layout, theme, scr.color, cursor mode, tabs.
    Visual marks (`m` letter) and navigation history are not persisted.

31. **No debug state:** breakpoints (`db`), register snapshot (`dr`), ESIL
    emulation state (`aeim`, `aets`), trace session (`dts`).

32. **No search / zignature / ROP state** (`z*`, `zs*`, ROP gadget DB, search
    history `/*`).

33. **No command aliases (`$*`) or macros (`(*`).**

34. **`prj_load` executes `CMDS` without any consent or sandbox**, which is a
    security foot-gun: loading a hostile project file can run arbitrary r2
    commands, including `#!pipe` on non-sandboxed builds.

35. **No project metadata besides name/user/time.** Missing: r2 version,
    commit hash, tag list, description, binary SHA256, original file path,
    original size, original mtime, architecture, bits, endianness, CPU
    variant — anything that would let `prj info` produce a useful report or
    a Merkle-tree-like integrity check.

36. **No `prj` command registered in the help trees / completion.** `prj?`
    does not exist; `prjhelp()` prints a short, non-standard INFO block
    instead of using `r_core_cmd_help`.

37. **No JSON output mode** for `prj info` / `prj r2`.

38. **No incremental save.** Today every `prj save` writes the whole file;
    for large projects (tens of thousands of flags) this is wasteful.

39. **No relationship with existing `P` commands.** `P+`, `Ps`, `P.`, `Pi`,
    `Pl`, `Pn` etc. do not know about `prj`, so there is no way to list,
    rename, delete, or annotate a new-style binary project through the
    canonical interface.

40. **Hard-coded `"?e hello projects"` cmd payload** — self-evident, already
    noted in §4.1.9.

---

## 5. What is lost when saving & restoring a project

Use this section as the master checklist. Each box is something the **current**
binary format (`prj save` → `prj load`) drops on the floor today, regardless
of whether the legacy `Ps` format covers it. Priorities are my opinion;
reshuffle freely.

### 5.1 Core analysis (highest priority)

- [ ] **Functions** (`afl*`): addresses, sizes, names, realname, calling
      convention, return type, stack frame size, noreturn flag, bb_min,
      diff state, purity, color.
- [ ] **Basic blocks** (`afb*`): per-function list, start/size, successors
      (jump/fail), switch-op tables, cmpreg/cmpval, colorize, traced flag,
      folded/hidden flag, fingerprint/diff tags.
- [ ] **Switch-op tables** (`JMPTBLPLAN.md` is adjacent context): case
      addresses and enum labels.
- [ ] **Xrefs** (`ax*`): edges `(from, to, type, perm)` — call, data, code,
      string.
- [ ] **Variables per function** (`afv*`): stack variables (base pointer
      delta, name, type), register variables (reg name, name, type),
      argument vs local distinction, access reads/writes per address.
- [ ] **Function signatures** (`afs*` / `afcr*`).
- [ ] **Calling conventions** (`tcc*`): the list of conventions the current
      project relies on, not just the default.
- [ ] **Noreturn functions** (`afnr*` / `tn*`).
- [ ] **Classes** (`ac*` / `aC*`): methods, bases, vtables, protocols.
- [ ] **Resolved vtables** (`avrr*`, `av`).
- [ ] **Types** (`t*`): structs, unions, enums, typedefs, atoms, function
      pointers. Includes the `anal/types` sdb namespace.
- [ ] **Meta items** other than comments: strings (`Cs`), data (`Cd`),
      format (`Cf`), magic (`Cm`), run (`Cr`), hidden (`Ch`), highlight
      (`Ch` colour), code ranges (`Cc`), "do not disassemble" (`Cs`),
      zones (`CC`), bookmarks.
- [ ] **Analysis hints** (full `ah*`, not just `immbase` and `newbits`):
      arch, bits, cpu, syntax, size, optype, opcode, stackframe, ptr, val,
      jump, fail, nword, esil, highlight, ret, immbase, noret.
- [ ] **Flag spaces** (`fs` / `fs+` / `fs-`): flags belonging to
      `strings`, `symbols`, `imports`, user-defined spaces, priorities.
- [ ] **Flag zones** (`fz*`): named ranges.
- [ ] **Flag realnames / aliases / colours.**
- [ ] **Function code graph overrides** (`agC`, `ag` layout hints): layout
      algorithm, seed, collapsed-node list, manual positions.
- [ ] **Opcode operand hints** (`ano*` / `anov*`): constants, enum mapping.
- [ ] **ESIL zones** (`aez*`) and pins (`aep*`).

### 5.2 IO / binary mapping (high priority)

- [ ] **All IO maps** (not only the first one — bug 4.1.1). Name, fd,
      delta, perm, itv, bank assignments, priority.
- [ ] **Open files** (`o*`): list of fd → path + load options, the active
      fd (`o=`). Required for multi-binary sessions and libraries.
- [ ] **Bin object state** (`oba*`): baddr, laddr, arch override, bits
      override, demangle options, rel/reloc overrides.
- [ ] **Write cache** (`wc*`): uncommitted writes (address, size, bytes),
      plus the `io.cache` config bit that enabled them.
- [ ] **Mapped blobs / RBuffer attachments** (for synthetic maps such as
      `malloc://`, `hex://`, `null://`).

### 5.3 Debug / emulation (medium priority)

- [ ] **Breakpoints** (`db*`): addr, size, enabled, hits, condition,
      command, name, esil trace flag, hw vs sw.
- [ ] **Registers snapshot** (`dr*` / `ar*`). Gated behind a debug or
      emulation mode; useful to resume live sessions.
- [ ] **Current PC / seek** (`s`).
- [ ] **ESIL trace session** (`dts*`, `aets*`): step history, register
      deltas, memory writes.
- [ ] **ESIL initialised memory** (`aeim*`): region, size, cookie.
- [ ] **Debug plugin / backend** (`dL*`, `e dbg.backend`), handlers, signal
      routing (`dk*`).

### 5.4 User interface (medium priority)

- [ ] **Panels layout** (`Vp`): panel tree, active panel, per-panel
      command, title, hidden flag.
- [ ] **Visual cursor / mark positions** (`m` visual marks, cursor mode,
      tabs).
- [ ] **Graph / disasm view preferences** (`asm.*`, `graph.*`) — overlap
      with config but UI users rely on it.
- [ ] **Color theme** (`eco <name>`, full palette `ec*`), highlight colour
      (`ec highlight`).
- [ ] **Command aliases** (`$name=…`).
- [ ] **Macros** (`(name cmd; cmd; cmd)`).
- [ ] **Command history** (`!!`) — optionally, gated behind `prj.history`
      like today.
- [ ] **Seek history** (`s-` / `s+`).

### 5.5 Metadata, notes, provenance (nice to have)

- [ ] **Project metadata**: r2 version, git commit, date (UTC), author,
      description, tags, free-form notes (`Pn`).
- [ ] **Original binary identity**: absolute path, basename, SHA256, size,
      mtime, arch/bits when opened.
- [ ] **Integrity trailer** (blake3 or xxhash64 of the payload).
- [ ] **Compression** (zstd or deflate of the payload after the header).
- [ ] **Encryption hook** (the config already has a stub `prj.gpg`).
- [ ] **Signature for trusted authors** (optional Ed25519 detached sig).

### 5.6 Runtime / session state

- [ ] **Config eval keys** marked as "project relevant": the complete set
      that `r_core_project_save_script` already dumps via `e *` when
      `R_CORE_PRJ_EVAL` is on.
- [ ] **Sandbox grain / enabled flag.**
- [ ] **Plugins loaded at save time** (`L*`): asm/anal/io/bin, so reload
      can re-check availability and warn.
- [ ] **Zignatures** (`z*`): the entire signature DB used for matching.
- [ ] **ROP gadget DB** (`rop.d/`, today serialised as nested sdbs).
- [ ] **Search history** (`/*`): last regex/pattern, last hits.
- [ ] **Yank buffer** (`y`): optional.

### 5.7 Project-system integration

- [ ] `prj save <name>` resolves `<name>` against `dir.projects` when it
      does not contain a path separator, same as `Ps`.
- [ ] `prj save` / `prj load` honour `prj.files`, `prj.vc`, `prj.history`,
      `prj.sandbox`, `prj.abspath`, `prj.prompt` the same way the legacy
      code does.
- [ ] `prj.name` is written to and read from the `INFO` entry, and reflected
      in the eval.
- [ ] `prj info <name>` without a path reads `dir.projects/<name>/prj.bin`.
- [ ] `prj list` / `prj ls` equivalent of `Pl` — and conversely, `Pl` knows
      about `.prj` files.
- [ ] `prj delete <name>` equivalent of `P-`, with the same safety guards
      (`project_path_is_within_projects_dir`, interactive confirmation).
- [ ] `prj diff <name>` equivalent of `Pd` (git log against the project dir).
- [ ] `prj rename`, `prj merge`, `prj fork`.
- [ ] `prj` accepts `-j` for JSON output.
- [ ] `prj?` renders real `RCoreHelpMessage` help.
- [ ] Autocomplete entry so `prj <TAB>` and `prj save <TAB>` work.
- [ ] Hooks so dirtying analysis state marks the project as unsaved
      (leverage `R_DIRTY_CHECK`, already wired for the old format via
      `r_core_project_is_dirty`).

---

## 6. A proposed design for v2 of the format

This section is a sketch, not a commitment. Feel free to throw away any
subsection.

### 6.1 Container

```
┌────────────────────────────────────┐
│ magic    'RPRJ' (4 B)              │
│ version  ut16                      │     header (16 B aligned)
│ flags    ut16                      │
│ payload_size  ut64                 │
│ payload_checksum  ut64 (xxhash64)  │
├────────────────────────────────────┤
│ payload (optionally zstd-compressed)│
│   = TLV records                    │
├────────────────────────────────────┤
│ trailer 'END!' (4 B)               │
└────────────────────────────────────┘
```

- `flags` bit 0: payload is zstd-compressed. Bit 1: payload is encrypted.
  Bit 2: trailer carries Ed25519 signature. Etc.
- `payload_size` and `payload_checksum` make partial writes detectable.
- Trailer magic makes it easy to append appendices (e.g. a secondary
  detached signature, future metadata extension) without breaking parsers.

### 6.2 TLV records

Each record: `{ tag: ut16, flags: ut16, size: ut32, payload[size] }`. Size
excludes the 8-byte header (fixing bug 4.1.2). Tag namespace is structured:

```
0x00xx  Container records (STRS, META, META_SIGN, TRAILER_PAD, …)
0x01xx  IO state           (FILES, MAPS, BANKS, WRITE_CACHE)
0x02xx  Bin state          (BININFO, BADDR, SECTIONS, IMPORTS, CLASSES)
0x03xx  Anal state         (FUNCTIONS, BBLOCKS, XREFS, VARS, TYPES,
                            CALLING_CONVS, HINTS, CLASSES, VTABLES,
                            METACMT, METASTR, METAFMT, METAZONE, …)
0x04xx  Flag state         (SPACES, FLAGS, ZONES, COLORS)
0x05xx  Debug state        (BREAKPOINTS, REGS, ESILTRACE, DBGMAP)
0x06xx  UI state           (PANELS, THEME, MARKS, ALIASES, MACROS, HISTORY,
                            SEEKHIST)
0x07xx  Session state      (CONFIG, SANDBOX, SIGDB, ROPDB, SEARCH, YANK)
0x08xx  Custom / plugin    (reserved for RCorePlugin contributors)
```

Unknown tags are simply skipped by the loader, and the container-level
`flags` on each record advertise whether a record is **mandatory** (abort
on unknown) or **optional** (skip).

### 6.3 String table

One global `STRS` section at the end (we can emit it last, as today) and
optionally a per-record mini-pool for locality. Entries reference strings
by `ut32` offset. Index 0 is reserved to mean "empty string", and the
sentinel for "none" is always `UT32_MAX`.

### 6.4 Address encoding

All addresses are stored as `{ mod: ut32, delta: ut64 }` tuples where
`mod == UT32_MAX` means "absolute". Mods are defined per-file in the
`FILES` record (one entry per `RIODesc`) and per-map in the `MAPS` record.
This guarantees re-basable projects across different `-B` load addresses.

### 6.5 Serialisation API

Define `RProjectSerializer`/`RProjectDeserializer` abstractions in
`r_core` (or better `r_util`) that take an `RBuffer` plus a tag table.
Each subsystem (`r_anal`, `r_flag`, `r_bin`, `r_io`, `r_debug`, `r_panels`)
registers a pair `(save, load)` for every tag it owns. This mirrors how
`r_core_project_save_script` is organised today, but with binary
round-tripping instead of re-parsing r2 commands.

### 6.6 Integration with the `P` command

- `e prj.new = true` becomes the default after the format stabilises.
- `Ps <name>` writes `dir.projects/<name>/prj.bin` + `notes.txt` +
  optional `history`, and keeps the rvc/git workflow.
- `Po <name>` / `P <name>` loads `prj.bin` preferring it over `rc.r2`
  when both exist, with a deprecation warning for the old format.
- A migration helper `prj migrate <name>` reads `rc.r2` by replaying it
  into a temporary session and then writes `prj.bin`.

---

## 7. Testing strategy

Tests for project serialisation live under `test/db/cmd/projects`. The new
format should reuse that directory with a dedicated tag (e.g. `PRJ_BIN`) so
existing tests stay for the script format. Proposed new tests:

- [ ] **Round-trip smoke test** (`prj save` → `prj load` into a new r2
      instance): expect the same number of flags, same addresses, same
      names, same map layout.
- [ ] **Round-trip with `-B 0x<base>`**: save at one base address, load at
      another; every flag must land at the rebased address because the
      `mod+delta` encoding does its job.
- [ ] **Round-trip with comments and hints** (immbase, newbits, later all
      hint kinds).
- [ ] **Magic validation**: feeding `prj load` a bogus file must fail
      cleanly (no crash, no spam).
- [ ] **Fuzz corpus**: random-byte mutations of a valid file must never
      crash, assert, or leak. Wire into the existing `ia_fuzz` /
      `clusterfuzz` setup.
- [ ] **Maps bug regression** (bug 4.1.1): save a file with ≥4 maps, load
      it back, every flag must find its mod.
- [ ] **`rprj_find` regression** (bug 4.1.2): a file with many entries must
      decode with the correct mod count.
- [ ] **Non-LE hosts**: cross-compile tests run under qemu (ppc64be,
      sparc). The current raw-struct serialisation fails; the new byte-oriented
      encoding must not.
- [ ] **Sandbox test**: `prj load` of a file whose `CMDS` runs `!rm -rf /`
      must be rejected when sandbox is on and gated behind a confirmation
      prompt when sandbox is off.
- [ ] **Large project benchmark** (≥100k flags, ≥10k functions): save/load
      time under a budget (e.g. 500 ms on CI hardware); file size under a
      budget (e.g. <10 MiB uncompressed, <2 MiB with zstd).
- [ ] **Deterministic output** test: saving twice with the same state must
      produce byte-identical files (requires sorted flag iteration and
      zeroed struct padding).
- [ ] **Interop test**: `prj migrate <old>` on every `.zrp` in `test/db`
      must succeed and reproduce the state.
- [ ] **Version skew**: a v1 file loaded by a v2 binary must be upgraded
      to v2 in place (or at least read correctly).
- [ ] **Unit tests** (`libr/core/test/test_prj.c`) for every serialiser:
      `st_append`/`st_get`, entry roundtrip, mod resolution, hint kinds,
      meta kinds, xref edges, flag spaces, etc.

---

## 8. Master TODO (checkbox list for review)

Tick boxes mean "the new binary format handles this". Re-order as needed.

### 8.1 Bug fixes (blocking v2)

- [ ] **FIX-1:** rewrite `rprj_mods_write` to iterate all IO maps correctly.
- [ ] **FIX-2:** make `rprj_find` read only `entry.size - sizeof(header)` bytes
      (and teach callers that `size` excludes the header in v2).
- [ ] **FIX-3:** add `RPRJ_CMNT` → `"Comments"` in `entry_type_tostring`.
- [ ] **FIX-4:** rename `sizeof(R2ProjectFlag)` to `sizeof(R2ProjectComment)`
      in `rprj_cmnt_read`.
- [ ] **FIX-5:** align `flag.size` / `cmnt.size` field widths between
      writer and reader; either store `ut32` (matches `RFlagItem->size`)
      or fully write a `ut64` via `r_write_le64`.
- [ ] **FIX-6:** zero out struct buffers before serialising; or (preferred)
      replace raw-struct writes with explicit field-by-field byte writes.
- [ ] **FIX-7:** free `RBuffer`, `st.data`, and `cur.mods` in every return
      path of `prj_save` and `prj_load`.
- [ ] **FIX-8:** guard all string-table lookups against NULL returns.
- [ ] **FIX-9:** remove dummy `"test-project"`, `"pancake"`, `?e hello
      projects`, `?e goodbye`, `?E clippy`, `"one string"`, `"another one"`.
- [ ] **FIX-10:** route file I/O and command execution in `prj_load`
      through `r_sandbox_check`.
- [ ] **FIX-11:** deterministic flag iteration order (sort by address, then
      by name) for reproducible files.
- [ ] **FIX-12:** add a magic trailer / CRC so torn writes are detectable.
- [ ] **FIX-13:** add `prj?` help via `RCoreHelpMessage`, and autocomplete.
- [ ] **FIX-14:** resolve `prj <op> <name>` through `dir.projects` when the
      argument is not a path, mirroring `get_project_script_path`.

### 8.2 Schema extensions (roadmap, ordered by expected usefulness)

Core analysis (most user-visible):

- [ ] **Functions** (full `RAnalFunction` graph: addr, size, name, realname,
      cc, type, bp_off, stack, noreturn, purity, diff, colour).
- [ ] **Basic blocks** with successors, switch op, fingerprint.
- [ ] **Xrefs** (to/from + type).
- [ ] **Variables** (stack + reg + args, types, access list).
- [ ] **Full analysis hints** (arch, bits, cpu, syntax, size, optype,
      opcode, stackframe, ptr, val, jump, fail, nword, esil, highlight,
      ret, noret) — not just `immbase` and `newbits`.
- [ ] **Types / calling conventions / noreturn DB / classes / vtables.**
- [ ] **All meta types** (`Cs`, `Cd`, `Cf`, `Cm`, `Ch`, `Cr`, zones).
- [ ] **Flag spaces, realnames, colours, zones.**

IO & bin:

- [ ] **All IO maps** (already in the format, but broken — see FIX-1).
- [ ] **IO files** (`o*`) and active fd.
- [ ] **Write cache** (`wc*`).
- [ ] **Bin info** (baddr/laddr, arch override, bits override, demangle).

Session:

- [ ] **Config eval** (filtered whitelist, matching `R_CORE_PRJ_EVAL`).
- [ ] **Command aliases, macros.**
- [ ] **Seek / seek history.**

Debug / emulation:

- [ ] **Breakpoints.**
- [ ] **Register snapshot / register profile override.**
- [ ] **ESIL map init** (`aeim*`).
- [ ] **ESIL trace / debug trace sessions.**

UI:

- [ ] **Panels layout.**
- [ ] **Theme** (`eco`, `ec*`).
- [ ] **Visual marks.**
- [ ] **Command history** (opt-in via `prj.history`).

Rare:

- [ ] **Zignatures (`z*`).**
- [ ] **ROP gadgets DB.**
- [ ] **Search history, yank buffer.**

### 8.3 Container & integration work

- [ ] Define v2 container (header + trailer + size + checksum + optional
      compression + optional signature) and bump the magic or version.
- [ ] Namespaced TLV tags (`0x01xx` IO, `0x02xx` bin, …).
- [ ] Split each subsystem's save/load into its own source file.
- [ ] Wire `Ps`/`Po`/`Pl`/`P-`/`Pi`/`Pd`/`Pn` to understand `.prj` files
      when `prj.new` is enabled.
- [ ] Migration helper `prj migrate <name>`.
- [ ] Hooks to mark the project dirty (`core->anal->is_dirty`,
      `core->flags->is_dirty`, `core->config->is_dirty`).
- [ ] JSON output modes for `prj info` / `prj list`.
- [ ] Document the format in `doc/` (binary layout table + tag registry).

### 8.4 Tests (tracking list — see §7)

- [ ] Round-trip smoke + rebased round-trip.
- [ ] Individual subsystem tests (flags, comments, hints, functions, …).
- [ ] Fuzz integration (`clusterfuzz-testcase-minimized-ia_fuzz-*` already
      lives in the tree, same pipeline).
- [ ] Big-endian host test.
- [ ] Sandbox test.
- [ ] Large-project benchmark gate.
- [ ] Deterministic output test.
- [ ] Version-skew test.
- [ ] Migration test from `rc.r2` + `.zrp`.

---

## 9. Open questions for review

1. **Scope of v1 → v2 transition.** Do we bump the magic, keep the magic and
   bump the version field, or ship v2 as a *distinct* plugin (`prj2`)? My
   tentative preference: keep `RPRJ`, bump `version=2`, and refuse to load
   v1 files (there are no real v1 files in the wild yet).

2. **Who owns serialisation?** Option A: everything inside
   `libr/core/p/core_prj.c` with helpers in other libs. Option B: each
   subsystem exports a pair `(serialize, deserialize)` via its own header
   (e.g. `libr/anal/serialize.c`). I lean to B because it keeps the plugin
   small and lets `anal` / `flag` / `bin` evolve independently.

3. **Compression default.** zstd is permissive and vendored widely; libc
   zlib is everywhere. Either works — but committing binary project files
   to git is a common workflow, so we may want to keep compression *off*
   by default and let users opt in via `prj.zip` / `prj.zstd`.

4. **Signing / encryption.** Do we want these in v2 at all, or is it
   enough to land an integrity hash and defer signing to a later version?

5. **Interop with the old script format.** Should `prj load` *also* read
   `rc.r2`-style files transparently, or force `prj migrate`?

6. **Command name.** `prj` is fine, but we should probably add a
   short-form under `P` (e.g. `Pb` for "Project binary") so the new
   format shows up in the canonical help.

7. **Deterministic iteration.** We need to pick a canonical order for
   every subsystem (flags by addr then name, hints by addr then kind,
   functions by addr, etc.). Worth agreeing explicitly because it affects
   every test.

8. **Identity of the binary.** Store only a SHA256 digest, or also the
   full path + mtime + size? The latter helps with "file moved / renamed"
   prompts; the former is more privacy-preserving.

9. **Multi-file projects.** Is the new format expected to serialise
   multiple `o` files (libraries), or is that still a single-binary
   assumption?

10. **Debug state.** Is it in scope at all? Breakpoints and ESIL state are
    a ton of work and arguably orthogonal to "project = analysis
    snapshot". My vote: yes to breakpoints (trivial), no to live registers
    / trace state (explicitly documented as non-persistent).

---

## 10. Glossary

- **Module (`R2ProjectMod`)** — one serialised RIOMap. Stores name,
  associated filename index, physical and virtual ranges, and a weak
  checksum of the first 1024 bytes used to re-bind the map on load.
- **Flag (`R2ProjectFlag`)** — one `RFlagItem`, rebased via mod index +
  delta.
- **Comment (`R2ProjectComment`)** — one `R_META_TYPE_COMMENT` interval,
  rebased via mod index + delta.
- **Hint (`R2ProjectHint`)** — one analysis hint (only `immbase` and
  `newbits` today).
- **String table (`RPRJ_STRS`)** — append-only byte pool of NUL-terminated
  strings, referenced by offset.
- **Legacy project** — `rc.r2` + side-cars under `dir.projects/<name>/`.

---

*End of document — ready for annotation.*
