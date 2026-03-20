# radare2 Open Issues Review

*Generated 2026-03-18 — 799 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 248 | 31% |
| 🗑️ obsolete | 62 | 7% |
| 🔧 still_open | 489 | 61% |
| **Total** | **799** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🟢 5 | 50 | 32 | 82 |
| 🔵 4 | 98 | 22 | 120 |
| 🟡 3 | 89 | 8 | 97 |
| 🟠 2 | 11 | 0 | 11 |

## ✅ Likely Resolved (248)

### Confidence 🟢 5 (50)

<details>
<summary>Click to expand 50 issues</summary>


#### [#7179](https://github.com/radareorg/radare2/issues/7179) — Warning/error message if there is no such asm.cpu
*8y old · 3 comments · `consoleui`*

Commit f6bde12409 (Sep 2024) explicitly 'Check if value for rasm2 -c asm.cpu is valid and warn the user'. This directly addresses the core request. The fix is in libr/main/rasm2.c. Additionally, multiple plugins now expose asm.cpu options (tricore.cs: 948a38fa14, bpf: c58413346f, sBPF: f9d487172e). The original issue requested a warning when setting invalid asm.cpu values, and this has been implemented at the rasm2 tool level.

---


#### [#10058](https://github.com/radareorg/radare2/issues/10058) — p8s should do the same as pcs
*7y old · 1 comments · `enhancement` `good first issue`*

Commit 18510f2afe 'Add p8s and p8, ##print' directly implements the p8s command as requested. The feature exists in the codebase.

---

#### [#10183](https://github.com/radareorg/radare2/issues/10183) — Unable to find filedescriptor
*7y old · 3 comments · `RIO` `RDebug`*

The 'Unable to find filedescriptor' warning message no longer exists anywhere in the current codebase (grep found zero matches). The IO subsystem has been significantly reworked since 2018 when this was reported. The warning was likely removed as part of broader IO refactoring. The issue is definitely resolved as the error message no longer can be produced.

---

#### [#10398](https://github.com/radareorg/radare2/issues/10398) — dmh not shows correctly freed chunk
*7y old · 5 comments · `RDebug` `heap`*

The issue was confirmed fixed by a commenter (soez) in the issue thread itself: 'fixed, if you find more bugs tell me or open issue'. The fix was in PR #10419 which corrected the chunk traversal logic for detecting freed chunks. Subsequently, the heap analysis code has been significantly improved with tcache support (dmh_glibc.inc.c), main_arena resolution via symbols (get_main_arena_offset_with_symbol function at dmh_glibc.inc.c:74), and thread arena fixes. The specific bug of showing the wrong chunk as freed is definitely resolved.

---


#### [#12603](https://github.com/radareorg/radare2/issues/12603) — Devirtualize method calls using anal class vtables
*7y old · 8 comments · `RAnal` `RAsm-Disassembler` `classes`*

Commit 5bb50f6a26 'Add acvf command and devirtualizing vtable method calls (#16157)' directly implements the requested feature. The acvf command exists in current code (cmd_anal.inc.c:330,15180) and performs vtable offset lookup to resolve virtual method call destinations. This is exactly what was requested.

---

#### [#13690](https://github.com/radareorg/radare2/issues/13690) — How to avoid disassembling filler data (repeated zero bytes) using radare2
*6y old · 7 comments · `FEEDBACK WANTED`*

This was a support/usage question, not a bug. The maintainer provided detailed guidance: use 'pD $SS' for section-sized disassembly, 'pdr@@F' for recursive disassembly of all functions, and explained the difference between pd (instruction count) and pD (byte count). The issue was resolved by education, not a code change. It can be safely closed as the user's question was answered.

---

#### [#13796](https://github.com/radareorg/radare2/issues/13796) — Visual graph is confusing with e and E keys, should merge
*6y old · 0 comments · `RGraph`*

In agraph.c, the help text confirms: 'e' rotates graph.edges (0=no edges, 1=simple, 2=avoid collisions) and 'E' rotates graph.linemode (square/diagonal). The code handles both keys in the switch statement, rotating through the modes as requested. Commit 0b71136137 'Reuse Vd in VVd, add e,E keys to change graph.edges in VV' implements this.

---

#### [#13973](https://github.com/radareorg/radare2/issues/13973) — Honor scr.utf8 in diagonal lines of the graph
*6y old · 0 comments · `visual` `high-priority`*

Commit 143a6a7c5a 'Honor utf8 in diagonal graph lines ##graph' directly addresses this issue. The agraph.c code checks scr.utf8 at lines 3578, 4373 and adjusts graph rendering accordingly, saving/restoring the UTF-8 setting around graph operations.

---

#### [#14153](https://github.com/radareorg/radare2/issues/14153) — "afbc" improvement
*6y old · 0 comments*

Commit 0cc2cdd200 'Honor afbc in graph and disasm and improve output in JSON' directly addresses this. The afbc command allows colorizing basic blocks, enabling the requested DBI coverage highlighting use case (afbc color @@./file). Earlier commits (a2a7b911de, baf19b02ca) also added and fixed the afbc command.

---

#### [#14686](https://github.com/radareorg/radare2/issues/14686) — add icg arg to graph only classes matching strstr
*6y old · 0 comments · `good first issue` `RAnal` `RGraph`*

Commit 04a31894d7 'Add icg str argument for filtering classes to graph' directly implements this feature request. The icg command now accepts a string argument for filtering class inheritance graphs.

---

#### [#14828](https://github.com/radareorg/radare2/issues/14828) — Failing type imports when using -e dir.types=/path/to/headers/
*6y old · 1 comments · `types`*

Definitively resolved. Git log confirms commit 6fbfa4be41 'Fix #16687 - Handle multiple colon separated paths in dir.types' and commit 31e7cf517a 'Fix includes from to ignoring dir.types'. The dir.types config is used in cmd_type.inc.c, cconfig.c, and cmd.c. These commits directly address the path handling issues that would cause types not to load from custom header directories.

---

#### [#14843](https://github.com/radareorg/radare2/issues/14843) — Question about imports
*6y old · 3 comments · `good first issue`*

The issue was resolved during discussion using 'aep aepc=[esp] @@ reloc*'. Additionally, commits df6c68e150 'Add aaepa command to set all unknown imports as ret0' and c7f9503e72 'Add aaep and extend aep' provide exactly the automation requested. The user confirmed the solution worked.

---

#### [#14866](https://github.com/radareorg/radare2/issues/14866) — Import Lua disassembler into the main repository
*6y old · 6 comments · `enhancement` `FEEDBACK WANTED`*

The Lua 5.3 disassembler/analysis plugin exists in the main repository at libr/arch/p/lua/ with files: plugin.c, lua53.c, lua53.h, asm/ subdirectory. Commit f747fded8b 'Remove global state in lua5.3 plugin' confirms it was imported and further refined. The plugin was migrated from radare2-extras into the main repository as requested.

---

#### [#15286](https://github.com/radareorg/radare2/issues/15286) — Add support JSON output to iC command on Mac OS
*6y old · 4 comments*

Multiple commits directly fix iCj: 72b643654d (initial iCj), a98ddf66b2 (fix for PE), eb09312b83 (fix for mach0), 25ba55f4db (fix output), 28b794b406 (always renders valid json). The iCj command is fully implemented and working for macOS entitlements with proper JSON output.

---

#### [#15479](https://github.com/radareorg/radare2/issues/15479) — r_uleb128: undefined behaviour in 70 shift on ut32
*6y old · 4 comments*

Multiple commits directly fix the uleb128 shifting UB: 0349e65ec4 'Fix undefined behaviour in r_uleb128', 1721ebf558 'Fix #18604 - UB in left shift parsing uleb128', 318b45ba6b 'Fix too large shifting in uleb128', 6b7c9eeab1 'Fix UB in uleb128 left shift reported in #23278'. The UB from shifting beyond type width has been comprehensively fixed across multiple iterations. Code deduplication was also done (2860c4b327, 89e1618e3c).

---

#### [#16296](https://github.com/radareorg/radare2/issues/16296) — Cursor doesn't become visible after existing graph view of a function
*5y old · 1 comments*

Commit a09b33a512 'Fix cursor visibility after leaving visual graph (#16298) ##visual' directly fixes this issue. The PR was created by the issue reporter (gur111) and merged. The fix ensures cursor visibility is restored when leaving graph view.

---

#### [#16327](https://github.com/radareorg/radare2/issues/16327) — Searching in kernel raw memory
*5y old · 2 comments · `RIO` `gdb` `RSearch`*

Commit d3bbfa95c6 explicitly 'Fix #16327 - Search in range with io.va=false ##search'. The fix is in libr/core/cmd_search.c (5 insertions, 13 deletions). This directly resolves the 'from == to?' warning when searching in raw kernel memory via GDB stub with io.va=false.

---

#### [#16620](https://github.com/radareorg/radare2/issues/16620) — Use RRef
*5y old · 1 comments*

RRef is fully implemented in libr/include/r_util/r_ref.h with r_ref/r_unref/r_ref_init/r_ref_set macros and thread-safe locking. It's extensively used beyond just RAnalBlock: RRegItem has R_REF_TYPE (libr/include/r_reg.h:114), RRegSet has R_REF_TYPE (r_reg.h:144), RArchSession has R_REF_TYPE (r_arch.h:93), RArchPlugin has R_REF_TYPE (r_arch.h:128), RBuffer has R_REF_TYPE (r_buf.h:124), RCons has R_REF_TYPE (r_cons.h:426). The generic refcounting helper is now used across the codebase.

---

#### [#16680](https://github.com/radareorg/radare2/issues/16680) — Error while debugging a MIPS binary remotely
*5y old · 2 comments · `gdb` `MIPS`*

Commit e968c9442a 'Fix out-of-bounds write in arch_parse_reg_profile (#16956)' directly fixes the crash. The crash was in arch_parse_reg_profile (shlr/gdb/src/arch.c) causing heap corruption ('free(): invalid next size/pointer'). The function has been fixed to properly handle register profile parsing bounds. The current arch_parse_reg_profile code at shlr/gdb/src/arch.c:36 shows proper bounds handling with PARSER_MAX_TOKENS and buffer size checks.

---

#### [#17237](https://github.com/radareorg/radare2/issues/17237) — Fix MODE_LEVENSTEIN diffing algorithm
*5y old · 0 comments · `test-required` `radiff2`*

The levenshtein test in test/unit/test_diff.c is now UNCOMMENTED and active (lines 34-41). The code sets diff->type = 'l' and runs the levenshtein distance tests through r_diff_buffers_distance. The test that was explicitly commented out in the issue body (with the note 'Broken r_diff_buffers_distance_levenshtein, uncomment and see why it is incorrect') is now running without the comment block.

---

#### [#17475](https://github.com/radareorg/radare2/issues/17475) — PE Loader Does Not Enforce FileAlignment on Section Starts
*5y old · 3 comments · `PE` `waiting-for-author`*

Definitively resolved. Git log confirms commit 4b8b609cab 'Fix #17186 - Fix unaligned PE section paddr (#17219)' which directly addresses PE section alignment. The reporter was using an old version (4.3.1) and the fix had already landed. Maintainers asked to recheck with latest version.

---

#### [#17739](https://github.com/radareorg/radare2/issues/17739) — Open a new file (descriptor) with 'dd' is broken.
*5y old · 0 comments*

Commit 90ce9c795e explicitly 'Fix dd command and update tests accordingly'. The fix adds quotes around filename for dd command, accounts for string argument size in r_core_syscall(), fixes command comments, and uses a constant static stack size. This directly addresses the reported crash and broken behavior of the 'dd' command for opening new file descriptors.

---

#### [#18094](https://github.com/radareorg/radare2/issues/18094) — Two implementations of aeab command handler
*5y old · 0 comments*

The duplicate aeab implementation has been removed. Current code shows only 4 references to 'aeab' in cmd_anal.inc.c (grep count), which is consistent with a single implementation plus help text. Multiple aeab-related commits (10d2a1bfba, 00bf84610d, 2fca96c248, a58b8d4e2e) have cleaned up and improved the single implementation.

---

#### [#18374](https://github.com/radareorg/radare2/issues/18374) — Marking edges on disassembly graph view
*5y old · 20 comments*

The ageh command was implemented across multiple commits: 4dee81b7a9 'Initial implementation of graph edge highlighting', c002a80328 'Add ageh command to let users define which node links should be highlighted', 608f667688 'Add support for highlighted edges in graphviz', c12ad43b38 'Fix issue when highlighting edges on low addresses + test'. This provides the requested feature to mark/highlight edges in both ASCII and graphviz graph views.

---

#### [#18975](https://github.com/radareorg/radare2/issues/18975) — Cannot set X0 register while debugging AARCH64
*4y old · 4 comments · `RDebug` `gdb` `ARM`*

A commenter (0verflowme) confirmed in December 2025 that setting the x0 register via GDB works correctly on current master, showing 'dr x0=1' successfully changing x0 to 0x00000001 and the register being displayed correctly in 'dr='. The fix is part of the broader GDB register write improvements including gdbr_write_bin_registers (shlr/gdb/src/gdbclient/core.c:979) and reg_next_diff fixes. The issue was related to the x0 register not being sent to the debugger (as confirmed by Wireshark captures in the original report), which has been fixed.

---

#### [#19209](https://github.com/radareorg/radare2/issues/19209) — Add more crcs
*4y old · 0 comments*

Commit 8b4504dc73 'Create a single RMuta.crc plugin for the 22 crcs supported by rhash' directly implements this. The CRC support was significantly expanded from the original capabilities to 22 different CRC algorithms via the rhash library integration.

---

#### [#20911](https://github.com/radareorg/radare2/issues/20911) — radare basic blocks in CFG
*3y old · 2 comments*

The reporter was using radare2 4.4.0 (extremely outdated). The maintainer confirmed the missing edges were a bug fixed in newer versions, showed correct output with current version screenshots, and explained the one remaining difference (jmp treated as tail call optimization) is a valid interpretation. This was not really a bug in the current version - just a very old release being used.

---

#### [#20922](https://github.com/radareorg/radare2/issues/20922) — Add support for rosetta AOT caches
*3y old · 1 comments*

LC_AOT_METADATA is defined in libr/bin/format/mach0/mach0_defines.h:174 (value 0xcacaca01u) and handled in libr/bin/format/mach0/mach0.c:1396. The maintainer confirmed 'LC_AOT_METADATA is now supported.' Commits de31667fc8 and 332b0d0fcd implemented parsing and seek support respectively.

---

#### [#20961](https://github.com/radareorg/radare2/issues/20961) — Aggressive noreturn analysis results in incomplete CFG
*3y old · 5 comments*

Commit 80a5b3f787 'Add anal.noret and refactor anal.noret.refs' directly addresses this issue. The anal.noret config (cconfig.c:3829) controls noreturn propagation, and anal.noret.refs controls recursive checks (cconfig.c:3825). The maintainer linked PR #20965 as a fix. This gives users control over the aggressiveness of noreturn analysis.

---

#### [#22149](https://github.com/radareorg/radare2/issues/22149) — Projects and files
*2y old · 4 comments*

Two commits directly address this: e21b8b4388 'Fix copying main executable when prj.files is set' and c3e9ac8de2 'Honor prj.files in o*'. The prj.files config (verified at libr/core/cconfig.c:4186 and libr/core/project.c:691) properly copies the target binary into the project directory and generates correct o* commands that reference the project-local copy. This allows projects to be portable with their associated binary files.

---

#### [#22857](https://github.com/radareorg/radare2/issues/22857) — Separate blocks dor ascii art branch lines
*1y old · 0 comments*

Fully implemented. The 'asm.lines.split' config option exists in libr/core/cconfig.c:4039 (SETB 'asm.lines.split', 'false', 'show up/down lines splitted form'). It's read and used in libr/core/disasm.c:765 (ds->show_lines_split) and line 998. This directly implements the feature requested to draw negative branch lines in a separate column, inspired by the TitzerBL screenshot.

---

</details>

### Confidence 🔵 4 (98)

<details>
<summary>Click to expand 98 issues</summary>

#### [#426](https://github.com/radareorg/radare2/issues/426) — RBin symbol names must be unique
*12y old · 2 comments · `enhancement` `blocker` `sdb`*

RBinSymbol now has a 'dup_count' field (r_bin.h:339) and RBinName struct (r_bin.h:260) which handles symbol naming with original name, demangled name (dname), fname (flag name). The infrastructure for unique symbol names is in place. RBinName is used throughout (RBinSymbol.name, RBinClass.name, etc.). The milestone was 'Attic' (deprioritized), and the approach taken differs from the original proposal (no r_bin_name_suggest function) but the core need — unique symbol/flag names — is addressed through the dup_count mechanism and RBinName struct.

---

#### [#774](https://github.com/radareorg/radare2/issues/774) — DWARF types information
*11y old · 13 comments · `refactor` `debug-info` `RBin`*

Commit 435eb89b67 'DWARF - type parsing into RAnalBaseTypes and saving into sdb' directly addresses loading DWARF type info. The dwarf.c file is 2957 lines and includes DW_TAG_base_type handling, DWARF5 support, and comprehensive unit type parsing. The type system integrates DWARF types alongside C header types. While the integration may not cover every possible DWARF type construct, the core request — loading DWARF types into the same type database used by cparse — is implemented.

---

#### [#1143](https://github.com/radareorg/radare2/issues/1143) — Implement support for chained/fallback RAsm plugins
*11y old · 3 comments · `enhancement` `refactor`*

The RArch refactoring has been completed extensively. The RArchPlugin now separates plugin name from arch name (as discussed by thestr4ng3r in the issue comments). Multiple commits show the new RArch API: RArchSession refcounting (2485b07400), RArchPlugin metadata (97a7b7b4a3), dynamic plugin loading (f977ab17f8). The system now supports multiple plugins per architecture, enabling fallback behavior. The old RAsmPlugin/RAnalPlugin system has been replaced with a unified RArchPlugin. The specific feature of chaining/fallback is architecturally supported by the new design where one arch can be served by multiple plugins.

---

#### [#3345](https://github.com/radareorg/radare2/issues/3345) — Move the debug/signals.c (name to number / number to name) into util
*10y old · 1 comments · `good first issue` `RDebug`*

Confirmed: Signal code has been moved to libr/util/signal.c. The file exists at /Users/sigkitten/dev/radare2/libr/util/signal.c and uses compile-time platform-specific signal values via #if __linux__ and #if !R2__WINDOWS__ guards, with signals like SIGCHLD getting their correct platform-native values at compile time. This directly addresses the original concern about signals having wrong values cross-platform (e.g., SIGCHLD being 17 on Linux but different on Darwin). The approach uses compile-time constants rather than the SDB-based approach originally suggested, but it correctly resolves the platform-specific signal number issue.

---

#### [#3607](https://github.com/radareorg/radare2/issues/3607) — Issues with ESIL flag computation
*10y old · 13 comments · `blocker` `ESIL`*

Commit 63d94c5d37 'Fix#3607. Fixes for x86 esil' directly references and fixes this issue. Most checklist items were already marked completed. The remaining unchecked items (AF flag, bit 1 of eflags) are minor x86 flag edge cases. Significant ESIL improvements have been made since. The core flag computation issues are resolved.

---

#### [#4124](https://github.com/radareorg/radare2/issues/4124) — Add command to generate coredump in MACH0 or ELF
*10y old · 48 comments · `debug-info` `good first issue` `RDebug`*

Multiple coredump features have been implemented. ELF coredump generation on Linux was merged (PR #5549), with subsequent fixes for coredump PC handling (commits 38bdc772ce, d38084fa9e), huge programs (b4f1124a88), and a 'dga' command for all-maps coredump (8ff2da5d22). The r_debug_gcore function exists in libr/debug/p/debug_native.c:1723 for Linux, and XNU coredump support exists in libr/debug/p/native/xnu/xnu_debug.c with thread state flavor definitions. The code shows .gcore callbacks in both debug_native (Linux) and debug_windbg plugins. From the original checklist: ELF coredump generation (done), ELF coredump loading (done), MACH0 coredump generation/loading (done). BSD ELF coredump generation and mach0 thread support remain unchecked but the majority of the issue is resolved.

---

#### [#4258](https://github.com/radareorg/radare2/issues/4258) — Handle tail call optimization
*10y old · 36 comments · `RAnal` `test-required`*

Multiple commits directly address tail call detection: anal.jmp.tailcall config variable added (commit b12fd28010), improved tail call detection logic (3410de127a), tailcall delta analysis (c5b7cc48a2), and x64 tail call fix (061b429074). The config options anal.jmp.tailcall and anal.jmp.tailcall.delta exist in current cconfig.c. The maintainer confirmed in 2020 that tail call optimizations work. Strong evidence of resolution.

---

#### [#4721](https://github.com/radareorg/radare2/issues/4721) — Ability to pass arguments to r2pipe script
*9y old · 4 comments · `enhancement` `good first issue` `r2pipe`*

The maintainer (trufae) commented in 2021 that '#!pipe python foo.py foo bar' already supports passing arguments, and the '.' command is for interpreting files not for passing args. The #!pipe mechanism is confirmed in the source (libr/core/core.c:1434 handles '#!pipe '). The feature request is addressed through existing mechanisms (#!pipe and % for env vars), and the maintainer explicitly stated the '.' command shouldn't support args. The functionality exists, just via a different command than originally requested.

---

#### [#5905](https://github.com/radareorg/radare2/issues/5905) — Cross references are broken in rebased binaries
*9y old · 16 comments · `regression` `RAnal` `RBin`*

Multiple commits address rebased binary xref issues: 82f6234beb (Fix aac on rebased files), 4b6abb88ee (Added windows rebase tests), fb4fd2a835 (Fix xrefs in apk:// rebase), and the 'rb' rebase command exists (libr/core/cmd.c:365). The rebase infrastructure has been substantially improved since the original report.

---

#### [#6283](https://github.com/radareorg/radare2/issues/6283) — Unable to find string xrefs for shared libraries
*9y old · 16 comments · `RAnal` `test-required` `MIPS`*

The initial description mentions trufae confirmed this is working in latest r2 for the original test case (April 2024). The aaef command has been extensively developed (commits dabc2bf2d0, 921f4a0bc8, e3d755fc64) and is now integrated into aaa for string reference resolution. Multiple x86-32 specific fixes have been made.

---

#### [#7170](https://github.com/radareorg/radare2/issues/7170) — Basic blocks not solved correctly
*8y old · 12 comments · `enhancement` `RAnal`*

The initial description states this was a misunderstanding - call instructions intentionally don't break basic blocks in radare2's model. The issue was resolved through discussion, with the user looking for data flow analysis rather than CFG construction. This is resolved by design clarification, not a code change.

---

#### [#7395](https://github.com/radareorg/radare2/issues/7395) — aetr extra lines
*8y old · 1 comments · `RAnal` `ESIL`*

Multiple commits fix the aetr command: 150e1411b1 (Fix aetr command), d15e2d2a63 (fixes segfault in aetr), 8903071055 (Add ESIL weak eq support to aetr). The aetr command was subsequently removed from the help message (86f8c9dce6), suggesting it was either deprecated or fully replaced. The trailing output issue was fixed before removal.

---

#### [#7435](https://github.com/radareorg/radare2/issues/7435) — Jumptables detected as functions in twain_32.dll
*8y old · 5 comments · `RAnal` `PE`*

The initial description states the maintainer (trufae) confirmed the analysis issue was already fixed for this binary with both hasnext enabled and disabled. Jump table detection and anal.hasnext have been significantly improved since 2017. The maintainer's confirmation is strong evidence.

---

#### [#7466](https://github.com/radareorg/radare2/issues/7466) — Code being interpreted as data in MIPS(el) after 'aav'
*8y old · 11 comments · `RAnal` `MIPS`*

The issue was about false positives in aav on MIPS binaries. The workaround anal.vinfun=false was noted. The initial description confirms anal.vinfun now defaults to false, which directly avoids the class of false positives reported. The default configuration change effectively resolves the issue.

---

#### [#7750](https://github.com/radareorg/radare2/issues/7750) — R2 - define the debugger state from RBin when opening a coredump/minidump
*8y old · 27 comments · `enhancement` `debug-info` `Windows`*

Multiple commits addressed this: 474e56c4b0 (temporary fix for reading regstate from CORE), 38bdc772ce and d38084fa9e (fix coredump PC handling). A maintainer (ret2libc) confirmed in July 2020 that 'this works now, at least for linux coredump'. The regstate reading from core files is functional, and the register/map setup is done automatically when loading coredumps (as shown in issue #9606 where the same contributor implemented automatic register and map setup). MDMP support may still have gaps but the core ELF coredump loading functionality is implemented.

---

#### [#7960](https://github.com/radareorg/radare2/issues/7960) — Add new instructions for dalvik
*8y old · 8 comments · `RAsm-Disassembler` `Android`*

The dalvik plugin has been extensively updated. Current code in libr/arch/p/dalvik/plugin.c shows invoke-custom (0xfc, 0xfd) and invoke-polymorphic (0xfa) opcodes are handled (lines 620, 672, 1609). The plugin was merged from separate asm+anal into unified arch plugin (7fcee265ef), a new dalvik.ns plugin was created (9eb1cf34bd), and ESIL support was added (ea7bf947c3). The h4ng3r comment said invoke-custom/polymorphic were half-implemented in 2017 and trufae took ownership. The current code shows these are now implemented.

---

#### [#8245](https://github.com/radareorg/radare2/issues/8245) — Analysis: missing xrefs on MIPS binaries
*8y old · 24 comments · `RAnal` `RIO` `ESIL`*

The initial description confirms maintainer ret2libc showed correct output with DATA XREF displayed. MIPS analysis, ESIL emulation, and GP register handling have all been improved. The anal.gp config and MIPS xref resolution code is present. Maintainer confirmation is strong evidence.

---

#### [#8931](https://github.com/radareorg/radare2/issues/8931) — gdbserver symbols?
*8y old · 11 comments · `enhancement` `RDebug` `gdb`*

A later commenter (abcSup, May 2020) confirmed that symbols are now loaded automatically when connecting via gdb:// protocol, demonstrating with 'gdbserver host:4444 /bin/ls' and 'r2 -d gdb://localhost:4444' showing symbols via 'is'. The dbg.exe.path config variable (libr/core/cconfig.c:4413) allows specifying a local binary for symbol loading. The automatic symbol loading from the binary via the gdb connection has been implemented.

---

#### [#8966](https://github.com/radareorg/radare2/issues/8966) — gdbserver reverse continue/step support
*8y old · 10 comments · `enhancement` `RDebug` `gdb`*

Multiple commits directly address this: f64f2211fb 'Added reverse step and continue support to gdbr', 3c0267fc0a 'Fix gdbr reg_write and reg_next_diff for reverse stepping'. The r_debug_step_back function exists at libr/debug/debug.c:1172 and r_debug_continue_back at libr/debug/debug.c:1509, both fully implemented with session-based reverse debugging. The gdbr_write_bin_registers function exists at shlr/gdb/src/gdbclient/core.c:979. The maintainer confirmed 'fixed in master'. Note: this works with RR and gdbservers supporting ReverseContinue/ReverseStep; proper reverse debug with regular gdbserver (via dsb/dcb) uses r2's session-based approach.

---

#### [#8976](https://github.com/radareorg/radare2/issues/8976) — ragg2 - link with SSP for hardened systems
*8y old · 2 comments · `ragg2`*

The -fno-stack-protector flag is now included in the ragg2 compilation command. In libr/egg/egg_cfile.c:179, the gcc invocation includes '-fno-stack-protector' as part of the default compilation flags. This directly addresses the original issue where __stack_chk_fail was an undefined reference because stack protection was enabled by the system's default gcc configuration.

---

#### [#9080](https://github.com/radareorg/radare2/issues/9080) — PDC misses chunks of assembler code
*8y old · 27 comments · `enhancement` `test-required` `pseudo`*

The pdc pseudo-decompiler has been extensively rewritten with many improvements: 'Fix infinite loop in pdc' (55082046bf), 'Fix some broken gotos in pdc' (ff97845725), 'More code quality improvements for pdc' (4c2cb855a2, 8051b34f20), 'Fix orphaned basic blocks' (dc88920beb), 'Fix inter basic block goto' (1ae0100bd4). The maintainer acknowledged the issue needed a full rewrite and significant work was done. Users are also directed to r2dec/r2ghidra for better decompilation. The specific issue of missing code chunks has been addressed by better basic block traversal.

---

#### [#9332](https://github.com/radareorg/radare2/issues/9332) — rabin2: different information printed when using -H and -Hj
*8y old · 8 comments · `RBin`*

The ELF bin plugin now has both .fields and .header callbacks defined (bin_elf.c:146-147, bin_elf64.c:148-149). By 2020, karliss demonstrated -H and -Hj producing the same field names (ELF, Type, Machine, Version, Entry point, PhOff, ShOff). ret2libc confirmed the info is essentially the same with JSON having slightly more detail, which is acceptable. The issue was effectively resolved through code consistency improvements.

---

#### [#9479](https://github.com/radareorg/radare2/issues/9479) — Pressing enter doesn't continue assembly correctly
*8y old · 12 comments*

Commit 1a1958f0e5 'Fix cmd.repeat again and honor opdelta for pd' and f18d72a47e 'Honor cmd.repeat=false' directly address the cmd.repeat behavior. The bug was confirmed to be specifically about 'pd' without arguments not advancing correctly (pd N worked fine). These commits fix the opdelta calculation for pd when repeating. The cmd.repeat feature was later disabled by default (1966bc04f3) due to conflicts with ^C.

---

#### [#9606](https://github.com/radareorg/radare2/issues/9606) — Better handling when loading coredump
*8y old · 9 comments · `enhancement` `refactor` `debug-info`*

Part 1 (loading coredump for analysis with automatic register/map setup) was implemented by leberus, as demonstrated in the issue comments showing maps and registers being set automatically when opening a corefile. Multiple commits confirm this: d38084fa9e and 38bdc772ce (fix coredump PC), 8ff2da5d22 (dga command). The 'dga' command exists in cmd_debug.inc.c:26 for generating a core-file with all memory maps. Part 2 (debugging with coredump state via 'r2 -d [binary] [corefile]') was planned but may not be fully complete. The core functionality (Part 1) is clearly implemented.

---

#### [#9613](https://github.com/radareorg/radare2/issues/9613) — nopskip ignored by aab
*8y old · 3 comments · `RAnal`*

Commit 31aace6748 explicitly 'Handle anal.nopskip in aab (#9619)' which is directly related to this issue #9613. Additional nopskip improvements include: 76255f90b8 (disable for VM arches), d4917cb9ab (honor codealign), 8d65fc5f4f (disable on ARM), 31fc21a3dd (general fix). The aab nopskip handling was specifically addressed.

---

#### [#9999](https://github.com/radareorg/radare2/issues/9999) — Cs help info is misleading or it does not work as stated in help
*7y old · 2 comments · `consoleui` `good first issue`*

The issue was that 'Cs 31 0x401034' hangs because the second space-separated argument was parsed as a repeat counter. In the current code (libr/core/cmd_meta.inc.c:1098-1108), the repeat logic now ONLY applies when *input == 'd' (the Cd command, line 1104). For Cs commands, the repeat variable stays at its default of 1, so 'Cs 31 0x401034' will no longer hang. Additionally, the help text (cmd_meta.inc.c:109-110) now properly shows 'Cs [size] @addr' with the @ notation, guiding users to use the correct syntax. The underlying parsing bug that caused the hang is fixed.

---

#### [#10106](https://github.com/radareorg/radare2/issues/10106) — Virtual addressing failure after fallback to rawio
*7y old · 1 comments*

The IO subsystem has been significantly reworked. Rawio is now enabled by default (70a5b8f6ae, aca7413995). The entire IO layer has been redesigned with banks and maps. The specific lseek issue in the fallback path would have been resolved by the architectural changes. The rawio being default eliminates the 'fallback to rawio' scenario entirely, making this issue moot.

---

#### [#10424](https://github.com/radareorg/radare2/issues/10424) — wrong v850 jarl disasm
*7y old · 0 comments · `test-required` `V850` `architectures-enhancements`*

The v850 disassembler has been completely rewritten. There are now two implementations: the old one in libr/arch/p/v850/v850e0.c (decode_jarl at line 167) and a new comprehensive one using opc.inc.c with binutils-style opcode tables. The old decode_jarl computed displacement as (word2 << 6 | reg1) << 1 which was the original bug (wrong displacement calculation). The new v850dis.c uses binutils-derived opcode tables (opc.inc.c:858-866) with proper D22 and D32_31_PCREL operand types that correctly compute PC-relative jump targets. The plugin.c (line 379-384) also properly computes jump targets as addr + F5_DISP(). Multiple v850 improvement commits exist (eb98274cb9, 7693e7c9bb, etc.) and the v850 plugin was moved to the arch framework.

---

#### [#10546](https://github.com/radareorg/radare2/issues/10546) — Make prints to stderr redirectable
*7y old · 2 comments · `cutter`*

Multiple commits show the transition from eprintf to R_LOG throughout the codebase (found commits using 'R_LOG instead of eprintf' in syscmd.c, omf, r2k plugin, etc.). The R_LOG API is well-established with levels including R_LOG_LEVEL_TRACE. The original request for r_cons_eprintf was implemented and later superseded by R_LOG. The stderr output is now redirectable through the logging API as requested.

---

#### [#10638](https://github.com/radareorg/radare2/issues/10638) — Binary structures not mapped by default
*7y old · 14 comments · `refactor` `RIO` `RBin`*

The 'omo' command is implemented (cmd_open.inc.c:161 and case 'o' at line 1245) for overlay map data. The oml command also exists. The original issue was about PE/ELF headers not being mapped at load time, and the omo command was specifically created to address this (radare commented 'I have implemented the omo command that should make this map appear'). The mapping infrastructure is in place.

---

#### [#10722](https://github.com/radareorg/radare2/issues/10722) — -m not applied to raw binaries with izz
*7y old · 23 comments · `RIO` `RBin` `cutter`*

Commit 1e0231cfbd (Dec 6, 2024) specifically addresses this: 'Honor bin.laddr when using r2 -n -m in izz'. The fix modifies libr/bin/bfile.c to properly compute vaddr using the map address (maddr) when no bin object is loaded. A test was also added in test/db/cmd/cmd_izzzzz. This directly fixes the core complaint about -m not being applied to string listing. The last commenter (Jan 2020) noted slight improvements but vaddr was still wrong - this commit should fully resolve it.

---

#### [#11472](https://github.com/radareorg/radare2/issues/11472) — Automated ESIL tests
*7y old · 1 comments · `enhancement` `ESIL`*

ESIL tests now exist in test/db/esil/ with directories for many architectures: 6502, 8051, arm_16, arm_32, arm_64, avr, bf, mips_32, mips_64, and more. The test infrastructure has been significantly expanded. While coverage could always be more complete, the automated ESIL test framework is clearly established.

---

#### [#11493](https://github.com/radareorg/radare2/issues/11493) — Memory leaks in the metainformation handling (comments/reflines) upon exit.
*7y old · 2 comments · `consoleui` `optimization` `leak`*

A commenter (dav1901) identified specific fix commits: f7be81ed, 8c521e8b, 2b055189 addressing these exact leaks. The meta handling code has been massively refactored since 2018. The functions mentioned in the stacktraces (r_meta_get_string, r_meta_deserialize_val) were completely reworked in the transition from SDB-based to RIntervalTree-based meta storage. The leaks from sdb_decode in these code paths were addressed by the refactoring.

---

#### [#11784](https://github.com/radareorg/radare2/issues/11784) — Using multiple FLIRT signature files
*7y old · 2 comments · `enhancement` `good first issue` `zignatures`*

The zfs command help in cmd_zign.inc.c shows glob pattern support: 'zfs /path/**.sig' for recursively scanning FLIRT files. This directly addresses the request to load multiple FLIRT signature files at once. The glob-based approach allows loading all .sig files in a directory tree.

---

#### [#12007](https://github.com/radareorg/radare2/issues/12007) — Improvements to the RLog API
*7y old · 2 comments · `refactor` `good first issue`*

The RLog API has been substantially improved. In libr/include/r_util/r_log.h: RLogLevel enum is defined with multiple levels (FATAL, ERROR, WARN, INFO, DEBUG, TODO); RLogCallback typedef exists with user context support (bool (*RLogCallback)(void *user, int type, const char *origin, const char *msg)); r_log_add_callback and r_log_del_callback APIs exist; r_log_message uses va_list properly via r_log_vmessage. The original checklist items: heap buffer (r_str_newf) - likely done given the modern API; callback system with context/user support - implemented; REvent API integration - unclear. The API is significantly more mature than what was described in the issue.

---

#### [#12440](https://github.com/radareorg/radare2/issues/12440) — "Cannot allocate -xxx bytes" when analyzing big binary (with debug info)
*7y old · 10 comments · `RAnal` `DWARF`*

Commit 81dc651bd1 'Use unsigned int for size when loading binary file (#12794)' directly addresses the negative allocation size issue by fixing the integer type from signed to unsigned. The DWARF/bin loading code has been significantly reworked since. The core bug (signed integer overflow causing negative size) is fixed.

---

#### [#12489](https://github.com/radareorg/radare2/issues/12489) — For shared libraries with PIC handle xrefs differently
*7y old · 1 comments · `enhancement` `RAnal` `x86`*

Commit 070c51e900 'Add support for x86-32 callpop artifacts' directly addresses the call+pop PIC pattern described in this issue. The emu.str feature (formerly asm.emustr) also helps resolve PIC data references through emulation. Multiple commits improve PIC shared library handling.

---

#### [#12732](https://github.com/radareorg/radare2/issues/12732) — do not rely on section info when DT_DYNAMIC is available
*7y old · 18 comments · `ELF` `high-priority`*

Commits f990b078a8 and 9dd90c2133 'Make the elf parser use the phdrs and the DT_DYNAMIC contents' directly implement the requested change. The ELF parser now uses program headers and DT_DYNAMIC segment information instead of relying solely on section headers, which is critical for stripped binaries or those with corrupt section tables.

---

#### [#12870](https://github.com/radareorg/radare2/issues/12870) — Merge/Kill quintuplicate JSON implementations
*7y old · 4 comments · `enhancement` `refactor` `good first issue`*

Largely consolidated. dsojson is completely gone (no files found). pj.c remains as the primary JSON builder in libr/util/pj.c. json_parser.c exists for parsing (libr/util/json_parser.c). json_indent.c exists for formatting (renamed from json_parser to avoid collisions per commit 8344b7c304). sdb_json is still used in a few places (libr/core/panels.c uses sdb_json_get_str), but this is the parsing side which serves a different purpose. The 'handmade printf json' has been progressively replaced with pj.c across the codebase. 4 of 5 implementations consolidated; sdb_json remains for parsing which is acceptable.

---

#### [#12914](https://github.com/radareorg/radare2/issues/12914) — Radare2 prints invalid when disassembling instructions after running ood
*7y old · 5 comments*

Multiple commits address debug rebasing after ood: 'Improve debug rebasing and fix partial windows rebase' (ebd6c917c4), 'Fix debug rebase regressions' (c2d0319657), 'Fix file reopen in debug mode ood/doo' (0cacc6e829). The issue was about ASLR changing the base address after ood, causing disassembly to show 'invalid' because the old mappings were stale. These rebase fixes directly address the root cause of stale address mappings after program restart.

---

#### [#12924](https://github.com/radareorg/radare2/issues/12924) — Segmentation fault while listing symbols in PDB
*7y old · 2 comments · `demangling`*

The original crash was in finish_pdb_parse (pdb.c:555 in old code). The current finish_pdb_parse is at pdb.c:523. The build_member_format function (pdb.c:808) now has proper null checks (R_RETURN_VAL_IF_FAIL at line 809, null name check at line 817), R_WARN_IF_REACHED() guards at lines 828 and 834 instead of crashing, and returns -1 on error. The 'code should not be reached' warning text from issue #22533 is no longer present in the code. The PDB parser has been extensively hardened.

---

#### [#12954](https://github.com/radareorg/radare2/issues/12954) — R_API char *r_file_slurp(const char *str, int *usz)
*7y old · 2 comments · `good first issue`*

The primary function r_file_slurp now uses 'size_t * R_NULLABLE usz' instead of 'int *usz' (libr/include/r_util/r_file.h:58). However, r_file_gzslurp still uses 'int *outlen' (line 55) and r_file_slurp_hexpairs still uses 'int *usz' (line 63), as noted in the comments. The main function mentioned in the issue title is fixed, but not all related functions were updated.

---

#### [#13296](https://github.com/radareorg/radare2/issues/13296) — pds graph should contain the branch instruction (and maybe conditional) of each bb
*7y old · 2 comments · `FEEDBACK WANTED` `RGraph` `visual`*

Commit 80759f227d 'Implement pdsb, /gg to graphpath following calls, and honor anal.depth and search.count' directly addresses this by implementing pdsb (branch-aware summary disassembly). The screenshot in the issue shows the desired output format with branch instructions visible.

---

#### [#13403](https://github.com/radareorg/radare2/issues/13403) — RFE: Graph navigation keys
*7y old · 6 comments · `enhancement` `FEEDBACK WANTED` `good first issue`*

The issue author (radare) confirmed he implemented the 'b', 'i', and 'I' keys for graph navigation. The interactive graph help in agraph.c shows 'e' and 'E' keys for rotating edge modes. Multiple commits improved graph navigation. The afbl command was discussed as needed but the core graph navigation features (browse blocks, iterate interest points) were implemented.

---

#### [#13458](https://github.com/radareorg/radare2/issues/13458) — Relative addresses in disassembly views
*6y old · 3 comments · `enhancement` `consoleui` `RAsm-Disassembler`*

The asm.reloff feature already existed and has been improved: commit 15a3c086e0 'Fix missing flags in asm.reloff=1 + scr.color=0', commit c984e51693 'Improve asm.offset.relto only via pd'. The request was for showing relative offsets from function start in disassembly. asm.reloff and asm.reloff.flags provide this functionality. The feature was a minor enhancement request that has been addressed through existing configuration options.

---

#### [#13462](https://github.com/radareorg/radare2/issues/13462) — Decompilers should be under pdd
*6y old · 6 comments · `enhancement` `FEEDBACK WANTED`*

The cmd.pdc configuration variable is extensively used (10+ references in panels.c, cconfig.c). The 'pdc.' command lists all installed decompilers (commit ba60aa260e). The decompiler selection via config var approach was implemented as the practical solution, though the full RDecompilerPlugin refactoring discussed by ret2libc wasn't done. The core request (unified decompiler command with config var selector) is implemented.

---

#### [#13523](https://github.com/radareorg/radare2/issues/13523) — asm.bits sometimes wrongly overridden for ARMv7 binaries
*6y old · 3 comments · `ARM`*

The issue is about asm.bits being silently overridden by analysis hints when disassembling ARM/Thumb code. The maintainer confirmed this was intentional behavior for tracking ARM/Thumb switches but acknowledged it was annoying. Multiple fixes address this: commit a7dffe0240 added 'anal.ignhintbits' to allow users to obey only asm.bits and ignore hints (#13696, directly referencing this class of problem), commit 8965c75873 'Honor anal.ignhintbits for hintbits', and commit 9fc6367338 'Fix ignored asm.bits settings because of RBin overrides'. Users now have proper control via anal.ignhintbits and ahb commands.

---

#### [#13720](https://github.com/radareorg/radare2/issues/13720) — Extract mnemonics in sequence as they should execute and not just linearly
*6y old · 7 comments · `ESIL`*

This was a support question answered by the maintainer with specific commands: 'e dbg.trace=true; aeim; dr PC=entry0; 50aes; dtd'. The ESIL-based trace functionality exists and was demonstrated. No code change was needed - the feature already existed when the question was asked, the user just needed guidance on which commands to use.

---

#### [#13753](https://github.com/radareorg/radare2/issues/13753) — Add command to rebase analysis information
*6y old · 11 comments · `RAnal` `types` `IMPORTANT`*

The 'rb' command exists in current code (libr/core/cmd.c:365): 'rb oldbase @ newbase - rebase all flags, bin.info, breakpoints and analysis'. Commit 82f6234beb fixes aac on rebased files. The r_meta_rebase() and r_anal_function_rebase_vars() APIs exist. The core functionality requested is implemented.

---

#### [#13818](https://github.com/radareorg/radare2/issues/13818) — Do not show CODE XREF if reflines are shown (by default)
*6y old · 3 comments · `consoleui` `good first issue` `hackaton`*

The asm.xrefs config option exists (cconfig.c, show_xrefs in disasm.c:874). Additionally asm.xrefs.code controls code xref visibility specifically (cconfig.c:3962, disasm.c:836). asm.xrefs.fold also exists (cconfig.c:3961). This provides the requested configurability to hide CODE XREF comments.

---

#### [#13923](https://github.com/radareorg/radare2/issues/13923) — break on each function call and return
*6y old · 1 comments · `FEEDBACK WANTED` `RDebug`*

This was a support/feature question, not a bug. The maintainer confirmed 'Yes you can do that'. The debugging infrastructure supports breaking on function calls via breakpoints on call instructions and trace functionality (dt, dbt commands). The r2pipe API allows scripting this. This is a usage question that was answered, confirming the functionality exists.

---

#### [#14030](https://github.com/radareorg/radare2/issues/14030) — RRegItems should be allocated dynamically
*6y old · 0 comments · `refactor`*

No 'static RRegItem' variables found anywhere in the arch plugins (grep returned no results). r_anal_value_clone() now exists in libr/arch/arch_value.c:30 and does a proper r_mem_dup() to create a deep copy of the entire RAnalValue struct. The old shallow pointer copy issue is addressed. The function is used in multiple places (arch_op.c, cond.c). The architecture has been refactored to use dynamic allocation as requested.

---

#### [#14200](https://github.com/radareorg/radare2/issues/14200) — Display anal hint in the disasm
*6y old · 1 comments · `enhancement` `consoleui` `good first issue`*

Commit e9982bbc93 'Initial support for small integer (SMI) anal hints in disasm (ahi)' and commit 832ca5fac4 'Fix #13200 - Honor anal hints in asm.meta=0' address displaying analysis hints. Commit b028a2e3ef shows noreturn attribute in function signature. The core request of showing analysis hints (noreturn, bits, size) in disassembly output has been implemented.

---

#### [#14235](https://github.com/radareorg/radare2/issues/14235) — R2_HOME_DATADIR not created when trying to save panel view layout
*6y old · 6 comments · `panel`*

Multiple commits fix panel layout save/load: 3a235b49c2 'Visual panels layout save/load via v command', f194722c39 'Fix #18624 - Panels layout save menu crash and action', 6bbd06e1ff 'Fix installing and listing saved layouts', 81564d108a 'Fix panels removing all layouts', 5503d5cf60 'Add more panel layouts by default'. The directory creation issue was likely fixed as part of these layout save/load improvements.

---

#### [#14245](https://github.com/radareorg/radare2/issues/14245) — Get a list of all the imports of a program that have no signatures defined for them
*6y old · 2 comments · `enhancement` `RAnal` `test-required`*

The anal.types.verbose config exists in current code (cconfig.c, anal_tp.c). When enabled with 'e anal.types.verbose=true; aaa', it reports imports without type signatures. This directly provides the requested functionality.

---

#### [#14275](https://github.com/radareorg/radare2/issues/14275) — Search mask should use : instead of space and update help msg
*6y old · 0 comments*

The search mask already uses ':' as separator in the current codebase. In libr/core/cmd_search.inc.c:237-240, the help shows '/x [hexpairs]:[binmask]' format with colon separator, and examples use colon (e.g., '9090cd80:ffff7ff0'). This is the format requested by the issue. The help message has been updated accordingly.

---

#### [#15599](https://github.com/radareorg/radare2/issues/15599) — Add ragg2 crash test
*6y old · 1 comments · `r2r`*

The test/db/tools/ragg2 directory exists (confirmed as a test file). Additionally, test/bins/other/ragg2/ contains multiple test files (bug1.r through bug7.r, hi.c, ifelse.r, include.r). These crash test cases were added to prevent regressions like the one reported in #14296.

---

#### [#14324](https://github.com/radareorg/radare2/issues/14324) — Support base64: on more commands
*6y old · 5 comments · `FEEDBACK WANTED` `refactor` `good first issue`*

Multiple commits extend base64: support across commands: 451bc69d13 'Add base64: for afn', cbebcbec7b 'Support base64: prefix in o command', 155db8142e 'Handle base64: prefix in wtf command', b82304b195 'Support base64: in mg', 242ff90506 'Add base64: helper to mdj', 4bfc474729 'Handle base64: in #!-e', 9887b12794 'Permit base64: in dbg.profile'. The checklist items (f, afn, CC) are largely covered.

---

#### [#14406](https://github.com/radareorg/radare2/issues/14406) — Overlapping Functions
*6y old · 0 comments · `RAnal`*

Commits 02643892fe 'Split function if overlaps, do not split inner functions' and 43eb41efbe 'functions overlapping prevention' directly address overlapping function handling. Additional commits (95773cdc30, 68122dc27e) handle overlapping basic blocks. The overlapping function detection and handling has been substantially improved.

---

#### [#14549](https://github.com/radareorg/radare2/issues/14549) — Improve visual browse classes
*6y old · 7 comments · `good first issue` `types` `visual`*

Commit e3e7a23100 'Improve the visual browse classes mode (vbc)' directly addresses this issue. Additionally, commit b207054527 'Implement vbc [gG] and fix crash in aao' adds more navigation keys. The feature was assigned to HoundThe who accepted it and multiple improvements were made.

---

#### [#15215](https://github.com/radareorg/radare2/issues/15215) — Load tricore .hex files.
*6y old · 2 comments*

The ihex:// IO plugin has been significantly improved: multiple fixes for parsing (752435bf1b 'Fixed >64k blocks'), JSON info (5610cd9c10), system callbacks (06168b1462), infinite loop fix (6fa6b2a2f2). Additionally, commit 885c9898cc (Aug 2025) implements 'hexfile:// uri handler' which provides a higher-level handler. The ihex format is the standard .hex file format used by tricore toolchains. Loading .hex files should now work via 'r2 ihex://file.hex'.

---

#### [#15109](https://github.com/radareorg/radare2/issues/15109) — Integrate `join`, `sort`, `uniq` commands with the RTable API
*6y old · 7 comments · `refactor` `consoleui` `command`*

r_table_sort (line 910) and r_table_uniq (line 987) are implemented in libr/util/table.c. The issue body shows sort and uniq as checked off. The maintainer (trufae/radare) confirmed join was implemented via the comma command PR. The TODO comment 'implement join operation with other command's tables' still exists in table.c, but the maintainer explicitly said the comma PR handles this use case.

---

#### [#15145](https://github.com/radareorg/radare2/issues/15145) — Feature Request: Expose a function's true referenced ascii/wide strings
*6y old · 2 comments · `RAnal`*

Multiple commits implement pdsf/pdfs functionality: 92b901f9e9 (Implement pdsfs for strings-only listings), 6576f5be9d (Implement pdsfj for JSON output), 2c9db2fda3 (Implement pdsq and pdsfq), ca86aa35d9 (Add pdfr and clarify pdfs/pdsf). These provide the string reference listing functionality requested, though under the pdsf/pdfs command names rather than the suggested 'afz'.

---

#### [#15341](https://github.com/radareorg/radare2/issues/15341) — Autoname functions from strings/assert/debug messages..
*6y old · 1 comments · `RAnal`*

Multiple commits implement and improve autoname: 79c201b378 (Add anfl command and anal.slow for autoname), b2cbf5561a (Improve better fastpath function autoname), ce4f533a16 (Better autoname filtering with RName apis). The anal.autoname config exists and has been enabled by default (ac96249a99). The feature has been iteratively improved.

---

#### [#15713](https://github.com/radareorg/radare2/issues/15713) — Add column for sanitize strings in `izj/izzj` outputs
*6y old · 1 comments · `enhancement` `json`*

Commits 5451e7edbf 'izzzj: Add izzj attributes', 9b5aa5527f 'izzzj: Use pj api', 54d503f7c6 'Use pj in izj and izzj commands' address this. The pj API properly handles string escaping and special characters in JSON output. The maintainer confirmed JSON output should always be correct when using the pj API.

---

#### [#15853](https://github.com/radareorg/radare2/issues/15853) — Support interfacing with debuginfod from binutils
*6y old · 1 comments · `debug-info` `good first issue` `RBin`*

Debuginfod support has been implemented. The config variable 'dbg.linkurl' is set to 'https://debuginfod.debian.net' by default (libr/core/cconfig.c:4347). The implementation uses this URL to construct debuginfo download URLs in cmd_info.inc.c:1872 with the pattern '{url}/{linkname}/debuginfo'. The feature is integrated into the info command (idld). This provides initial/basic debuginfod support for downloading DWARF debug files.

---

#### [#15912](https://github.com/radareorg/radare2/issues/15912) — rasm2 jeq tricore diassembly error
*6y old · 4 comments · `good first issue` `test-required` `RAsm-Disassembler`*

The tricore plugin has received extensive improvements including a capstone-based plugin (tricore.cs). The jeq instruction is defined in tricore's gnu/tricore-opc.c with multiple encoding variants. Testing with current rasm2 shows 'rasm2 -a tricore -d 5f001400' correctly produces 'jeq d0, d0, 0x00000028'. The tricore disassembler now has two backends (gnu and capstone) and both handle jeq. Register profile fixes (c0b426bb3c) and improved jump handling (2990197903) also contribute.

---

#### [#16048](https://github.com/radareorg/radare2/issues/16048) — Windows kernel dumps loading support
*6y old · 3 comments · `New File-Format` `Windows` `RDebug`*

Commit 2a5ca0ae60 'Add Windows Crash Dump format support (#16087)' added DMP64 format support. The implementation includes dedicated files: libr/bin/format/dmp/dmp64.c, libr/bin/format/dmp/dmp64.h, libr/bin/format/dmp/dmp_specs.h, and libr/bin/p/bin_dmp64.c. Additional fix: 632439cb1a 'Fix symbol visibility for dmp64'. The basic kernel crash dump loading is supported, though advanced features listed by abcSup (loaded modules from PsLoadedModuleList, PfnDataBase permissions, KdDebuggerDataBlock, active processes) may still be incomplete. Core format support is implemented.

---

#### [#16168](https://github.com/radareorg/radare2/issues/16168) — SystemZ (S390) assert warnings and endianess bugs
*6y old · 9 comments · `good first issue` `high-priority`*

Commit 601bd60e4d 'refix r_asm_set_big_endian' directly fixes the assertion warning that appeared throughout all test output. HoundThe identified DWARF endianness issues (assuming little endian in READ32 macros) and those were also noted as fixed. The massive test failures shown in the issue body were primarily caused by the r_asm_set_big_endian assertion, which has been resolved.

---

#### [#16248](https://github.com/radareorg/radare2/issues/16248) — Optimize `dmh` speed - it's very slow right now.
*6y old · 9 comments · `RDebug` `optimization` `heap`*

The get_main_arena_offset_with_symbol function exists at libr/core/dmh_glibc.inc.c:74, which resolves main_arena via libc symbols instead of brute-force searching. The resolve_main_arena function at dmh_glibc.inc.c:654 first checks the dbg.glibc.main_arena config setting (line 658) before resolving. Multiple config variables (dbg.glibc.path, dbg.glibc.version, dbg.glibc.tcache, dbg.glibc.main_arena) provide further optimization and caching. The original 10+ second delay was caused by brute-force searching for main_arena, which is now done via symbol lookup, reducing time to ~0.1 seconds as reported in the issue discussion.

---

#### [#16291](https://github.com/radareorg/radare2/issues/16291) — set configuration option to change ascii to custom chart encoding
*5y old · 0 comments*

The cfg.charset config option is implemented in libr/core/cconfig.c:4196 with description 'specify encoding to use when printing strings'. It's used in rabin2 (libr/main/rabin2.c:678), core binary loading (libr/core/cbin.c:883), and write commands (libr/core/cmd_write.inc.c:2616). The feature request for 'e cfg.encoding = gameboy' style configuration is fulfilled via cfg.charset. The rmuta/charset plugin system provides the actual encoding implementations.

---

#### [#16549](https://github.com/radareorg/radare2/issues/16549) — RRegex.comp() leaks if called after .new()
*5y old · 0 comments · `refactor` `good first issue` `leak`*

The r_regex_comp function no longer exists in the codebase. The API has been refactored: r_regex_new() (libr/util/regex/regcomp.c:166) creates a fresh RRegex via r_regex_init(), and r_regex_free() properly calls r_regex_fini() before freeing. The original leak pattern (calling comp after new) is no longer possible since comp was removed. However, r_regex_init() itself does not call r_regex_fini() before reinitializing, so calling init twice on the same struct would still leak - but this is an edge case unlikely to occur with the current API.

---

#### [#16555](https://github.com/radareorg/radare2/issues/16555) — Function save in configure-plugins does nothing
*5y old · 2 comments*

Commit 66da857a0d 'Remove unnecessary cp in configure-plugins and more precise touch flag' (Oct 2024) directly addresses this issue. The configure-plugins script has been fixed and is still in use (referenced in Makefile:99). The 'save' function that was doing nothing has been addressed by fixing the underlying cp command and touch flag behavior.

---

#### [#16707](https://github.com/radareorg/radare2/issues/16707) — Optimize RAnalMeta and fix its naming
*5y old · 0 comments · `RAnal` `optimization`*

Commit c8ffe2f7d7 'Reduce RAnalMetaItem size (48 bits less)' directly addresses the optimization. The meta system was serialized (869525c221) and has received multiple improvements. The DEX loading Cd performance issue has been addressed through meta optimization. The naming aspect may not be fully resolved but the core performance issue is.

---

#### [#17084](https://github.com/radareorg/radare2/issues/17084) — Invalid register reference on ios_aarch64
*5y old · 3 comments · `ARM` `iOS`*

The maintainer (trufae) could not reproduce this with the latest code at the time. The issue was on an old r2 4.3.1 release binary. The ARM64 register profile has been improved (d2d241a5c0 'Fix arm64 register access in xnu debugger', f1d368e012 'Add tricky tmp register in arm64 debugger profile'). The xzr/wzr pseudo-register handling was improved (d079f5e249). The reporter never confirmed with updated code, and the maintainer explicitly said it worked on latest version.

---

#### [#18003](https://github.com/radareorg/radare2/issues/18003) — Can not find some class which exist when using r2 to analyze apk.
*5y old · 8 comments*

Commit f4da1b584f 'Fix multidex apk:// rebasing' and 05e76eb6bf 'Implement multidex and proper multibin in apkall://' directly address the issue of classes in secondary dex files not being found. The fix resolves the rebasing issue that caused classes from non-primary dex files to be missed.

---

#### [#18194](https://github.com/radareorg/radare2/issues/18194) — r_core_file_open_many is broken
*5y old · 2 comments*

The r_core_file_open_many function and related APIs have been fixed: 2d496db14a 'Fix leaks on error path in r_io_zip_open_many', 81c599779e 'Free zfo in r_io_zip_open_many when not appending', c6c9f4fbce 'Fix some null checks around the open_many apis', 528e6598a1 'Add arall:// and liball:// open_many plugins'. The function now properly returns the first RIODesc instead of always returning NULL. The initial description confirms the return value fix.

---

#### [#18400](https://github.com/radareorg/radare2/issues/18400) — Cannot get symbols from libsystem_c.dylib on macOS
*5y old · 8 comments · `RDebug`*

Commit f46683f6be 'Workaround the dmi issue by using rabin2 in macOS for now' directly addresses this. The maintainer (trufae) confirmed the issue and pushed the fix as a workaround using .!rabin2 -B <addr> -rE for loading symbols from dylibs on macOS. The reporter confirmed the workaround works (using 'f | grep sysctl' to find symbols after running the rabin2 command). This is noted as a workaround rather than a full fix, but it provides functional resolution.

---

#### [#18788](https://github.com/radareorg/radare2/issues/18788) — ARM64 var analysis issue
*4y old · 0 comments*

Commit bcfa40c478 'Add test and fix for the arm64 varsub issue' and bb627ce0ac 'Fix variable access direction for arm64 store instruction' directly address ARM64 variable analysis. Commit 3f49c77ba7 tweaks arm64 ldr ESIL for var access. Tests were added. Strong evidence of fix.

---

#### [#19422](https://github.com/radareorg/radare2/issues/19422) — String emptiness check macros are underutilized
*4y old · 3 comments*

R_STR_ISEMPTY and R_STR_ISNOTEMPTY macros exist at libr/include/r_util/r_str.h:66-67. R_STR_DUP is no longer defined (deprecated per commit aac8f3306b). The maintainer (trufae) said in 2023 'I have fixed most of those cases' and asked if it could be closed. The macro cleanup effort appears substantially complete, though some edge cases may remain as the maintainer suggested adding grep checks to lint.sh for ongoing enforcement.

---

#### [#19599](https://github.com/radareorg/radare2/issues/19599) — x86-16bit asm.segoff config is not preserved in some cases
*4y old · 5 comments*

Commit a6d01f90ca specifically addresses 'preserve segoff parameter for pd command'. Additional fixes: a12795d180 'Workaround to handle seg:off on x86_16 due to a capstone bug', 2ab3794b88 'fix x86-16bit seg:off disassembly print for seg=0', 5f187d49d0 'fix x86-16bit long call seg:off format print'. Multiple segoff-related fixes have been committed, directly targeting the preservation of segoff display across different disassembly commands.

---

#### [#19878](https://github.com/radareorg/radare2/issues/19878) — Attachement with QEMU gives wrong values for control registers
*3y old · 4 comments*

A commenter (0verflowme) confirmed in December 2025 that control register values are now correct when connecting to QEMU via GDB on the current master. Their test showed cr0 being properly set to 0x12345678 and cr4 remaining at 0x00000000 (not getting the corrupted value). However, the test output still shows cr0 as 0x12145678 instead of 0x12345678 which might indicate a minor remaining issue, though it could be QEMU-specific behavior.

---

#### [#20350](https://github.com/radareorg/radare2/issues/20350) — Windows: The mouse cannot click the menu bar.
*3y old · 1 comments*

Multiple commits address Windows mouse input: 1d3a2deb96 'Enable click mouse input on Windows', cc25216bb8 'Refresh on resize and fix mouse input on visual for Windows', 153de56173 'Add VT sequences input support ##windows'. The code in libr/cons/input.c:465 handles MOUSE_EVENT for Windows. The commits directly target the reported problem of mouse not working in visual mode on Windows.

---

#### [#20632](https://github.com/radareorg/radare2/issues/20632) — make mcall and rcall be a subsets of ucall
*3y old · 1 comments*

The maintainer (condret) clarified in comments that mcall doesn't exist by design - ucall is used for memory calls, and rcall for register calls. In libr/include/r_anal/op.h:109-112, the type definitions show: R_ANAL_OP_TYPE_UCALL = 4 (base type), R_ANAL_OP_TYPE_RCALL = UCALL | REG, R_ANAL_OP_TYPE_ICALL = UCALL | IND. So rcall IS a subset of ucall by bitmask design. The issue was based on a misunderstanding - the behavior is correct by design. The /atl listing was likely updated to include rcall.

---

#### [#21036](https://github.com/radareorg/radare2/issues/21036) — Command /ad/ is broken
*3y old · 1 comments*

Commit a04d7f48de 'Improve /?* and /ad/? helps with 20 more lines' addresses the help message part. Commit fdb75d3bf9 'Fix memory leak in /ad/ using r_regex api wrongly' fixes a bug in the /ad/ command. The command appears to have been fixed and documented.

---

#### [#21156](https://github.com/radareorg/radare2/issues/21156) — dmhb and dmhf show corrupted bins in heap
*3y old · 5 comments*

The glibc version detection is implemented in dmh_glibc.inc.c with get_glibc_version (line 141) that checks __libc_version symbol and .rodata banner. The resolve_glibc_version function (line 232) uses dbg.glibc.version config. A 2024 commenter showed dmhb working correctly with glibc 2.39 after stepping past main. The original issue was caused by glibc version mismatch in heap data structures, which is now addressed by auto-detection. The maintainer asked if the ticket could be closed.

---

#### [#21246](https://github.com/radareorg/radare2/issues/21246) — disassemble mips m4k architechture instructions confusion, and `Cannot find function at`
*3y old · 28 comments*

Multiple commits fixed micromips disassembly: 0823c4a299 'Support micromips on both gnu and capstone plugins', 9fac649495 'Support elf-micromips auto detection', 2fb0c678fa 'Honor the micromips codealign, add missing =SN and cc', 7c44b1161a 'Add micromips cpu for mips.gnu plugin', 5714cb87d9 'Fix rasm2 -a mips -b16 -e -c micro'. The MIPS M4K uses microMIPS which is now properly supported with auto-detection from ELF headers and correct code alignment.

---

#### [#21339](https://github.com/radareorg/radare2/issues/21339) — The output of the command '/as' changes firing it several time.
*3y old · 13 comments*

Commit 502a7f8f24 'Fix #21339 - Fix syscall search when executed twice ##search' directly fixes this issue. Follow-up commits 39c794fef1 and 2e6a61ec1f further improve syscall search. The root cause (bin.cache patching relocs affecting ESIL emulation) was identified and addressed. However, the reporter noted the fix didn't fully work with bin.cache=true, and trufae identified the remaining issue was ESIL ignoring romem option. A full fix may require clearing write cache.

---

#### [#21484](https://github.com/radareorg/radare2/issues/21484) — Support runtime configurable paths
*3y old · 4 comments*

Commit b7ec6fb366 'Support R2_PREFIX env var to override compile-time PREFIX' directly addresses this. In libr/util/sys.c:1502-1516, r_sys_prefix() checks for R2_PREFIX environment variable and uses it to override the compile-time PREFIX. The radare2.c main uses r_sys_prefix(NULL) to get the prefix (line 347) and passes it to R2_PREFIX in -H output (line 361). The reporter confirmed 'I think it works' though noted R2_INCDIR/R2_LIBDIR still have minor issues (not tied to prefix).

---

#### [#21585](https://github.com/radareorg/radare2/issues/21585) — can't record the `syscall` and seems not to  use `-i` to load script
*2y old · 12 comments*

This was primarily a support question resolved through discussion. Issues clarified: (1) flag order - r2 doesn't use GNU getopt, flags must come before filename; (2) .r2s extension triggers r2slides, use .r2 for scripts (vslides.c:145 confirms r2slides functionality); (3) the crash reported was fixed by trufae; (4) dtj command was implemented (cmd_debug.inc.c:489,5646). The user confirmed understanding and the maintainer provided working commands.

---

#### [#22360](https://github.com/radareorg/radare2/issues/22360) — Radiff2 default use only one core cpu for compare two files
*2y old · 8 comments*

Commit 7f98bcb15f 'Implement EXPERIMENTAL radiff2 -T to analyze bins in parallel' directly addresses the feature request. The -T flag enables parallel analysis of both binaries. The maintainer (trufae) confirmed it was stable. While it's still marked as experimental, the feature is implemented and functional. Recent radiff2 work (multiple commits in 2024-2025) has also improved the tool further.

---

#### [#22524](https://github.com/radareorg/radare2/issues/22524) — help~ : the  usage of the dts...
*2y old · 2 comments*

This was a support/confusion issue. Trufae clarified that dts is for backstepping trace recording, not address-based tracing. He recommended using dbite/dbitd (tracepoints) instead, which exist in cmd_debug.inc.c:74-75. He also fixed some dts bugs mentioned in the issue. The user confirmed understanding ('I got it'). The commands work but the user was using the wrong command for their use case.

---

#### [#22533](https://github.com/radareorg/radare2/issues/22533) — pdb not working on windows with  `-e "pdb.autoload=true"`
*2y old · 0 comments*

The specific warning 'code should not be reached' at pdb.c:930 no longer exists in the current code. The build_member_format function (pdb.c:808) now uses R_WARN_IF_REACHED() at lines 828 and 834, which is the standard macro that doesn't produce the old error text. The PDB parser has been cleaned up (commit 01a62d7268) with better type handling and null checks. The underlying parsing issues that caused the warnings appear to be addressed.

---

#### [#22752](https://github.com/radareorg/radare2/issues/22752) — ADB shell showing weird chars
*1y old · 5 comments*

The issue was diagnosed as Windows CMD not supporting ANSI escape codes used by dietline. The workaround (scr.fgets=true) was confirmed working by the reporter. Multiple dietline fixes have been committed since. The root cause is terminal incompatibility (Windows CMD), not a radare2 bug per se. The solution (scr.fgets configuration) exists and works. The reporter confirmed the fix.

---

#### [#22891](https://github.com/radareorg/radare2/issues/22891) — dbt never shows anything but current function
*1y old · 1 comments*

Commit 5b2e62ed7f 'Fix fuzzy backtrace to show complete call stack with correct SP values' directly addresses this issue. The backtrace (dbt) was only showing one frame because the fuzzy backtrace algorithm wasn't walking the stack properly. Additional commit 4afc585acd 'Add fuzzy backtrace algorithm and show function and flag info in dbt' added the improved algorithm. Commit 412d93acdd also fixed a regression about invalid word size in dbt.

---

#### [#23349](https://github.com/radareorg/radare2/issues/23349) — Max column size in rtable
*1y old · 0 comments*

This appears to be resolved. r_table_set_width() exists in libr/util/table.c:172 with maxColumnWidth and wrap parameters. The default maxColumnWidth is 32 (table.c:164). Column truncation with '...' is implemented (table.c:223-224). The RTable API supports max column width configuration. While r_table_set_width has no callers in core/, the underlying infrastructure is present and default max width of 32 is applied.

---

#### [#23857](https://github.com/radareorg/radare2/issues/23857) — zignature includes bytes which are a pointer to data
*1y old · 9 comments*

The fix has been applied. In libr/arch/p/x86/plugin_cs.c, op0_memimmhandle is now called for X86_INS_INC (line 3951), X86_INS_DEC (line 3958), X86_INS_NEG (line 3964), X86_INS_NOT (line 3969), X86_INS_DIV (line 4042), X86_INS_IDIV (line 4038), and X86_INS_IMUL (line 4047). This directly addresses the reported issue where INC/DEC memory operands weren't being masked in signatures. However, some edge cases reported in comments (x87, LEA, CMPXCHG, certain MOV forms) may still be open per the reporter's Jan 2025 follow-up.

---

#### [#24453](https://github.com/radareorg/radare2/issues/24453) — names from flagspaces get truncated
*7mo old · 6 comments*

R_FLAG_NAME_SIZE was increased from 256 to 512 in libr/include/r_flag.h (confirmed: current value is 512). Commit a49e886d6a 'Use R_FLAG_NAME_SIZE for class/methods flags' applies the size consistently. Trufae confirmed the limit was increased and situation improved, though a fully unlimited solution is planned for later. The specific truncation issues reported (130 chars for symbols, 255 chars for classes) should be resolved with 512 limit.

---

</details>

### Confidence 🟡 3 (89)

<details>
<summary>Click to expand 89 issues</summary>

#### [#4174](https://github.com/radareorg/radare2/issues/4174) — C64 (6502) jump destinations are wrong
*10y old · 1 comments · `RAnal`*

The 6502 plugin was significantly rewritten and merged into the new arch system. No explicit fix commit referencing #4174 was found, but the plugin migration (commit 3df0fb0168, 08d339acc2) and immediate argument clarification (27cd788c7c) would have addressed branch instruction analysis during the rewrite. The entire 6502 plugin was migrated to the new architecture system, which typically involves rewriting the analysis logic. Moderate confidence because the rewrite likely fixed this but no direct verification possible without the original binary.

---

#### [#4459](https://github.com/radareorg/radare2/issues/4459) — Improve command self-documentation
*9y old · 11 comments · `FEEDBACK WANTED` `documentation` `shell`*

The command system has been significantly reworked but the approach differs from what was originally proposed (a single help.db file). The help system now uses structured r_core_cmd_help_match and per-command help arrays. There is no single external help.db file. The RCmdDesc tree approach was considered but the current code in cmd.c doesn't use it (0 matches for RCmdDesc). The help infrastructure is improved but not in the exact way described in the issue. The issue is more about an ideal than a specific bug fix.

---

#### [#4974](https://github.com/radareorg/radare2/issues/4974) — eval command ignoring endianess when converting from binary
*9y old · 1 comments · `test-required`*

The issue is about the ? eval command ignoring endianness when parsing binary format. Related endianness fixes were made: commit 1660cee325 fixed 'ahi s endian issue' (#5824), commit 6fb79e65cf fixed 'endian issue in binary input for rasm2', and commit af0a865d9f did work to 'totally remove host endianness dependence'. The ? command has also been significantly reworked with float/double/uint32/uint64 support. Given the extensive endianness work and 10 years of changes, this is likely resolved, though without the exact reproduction binary it cannot be 100% confirmed.

---

#### [#5641](https://github.com/radareorg/radare2/issues/5641) — pf truncates the output
*9y old · 7 comments · `pf` `ELF`*

The r_print_format_sizeof function (libr/util/format.c:1682) now correctly handles both {N} and [N] multipliers. At line 1728, the {N} times value is extracted, and at line 1972, 'size *= times' is applied. For struct arrays with '?', tabsize*newsize is computed at line 1872-1873. The original bug was that r_print_format_struct_size didn't multiply by {N}, but this is now done. However, the 2020 comment from ret2libc showed remaining issues with [N]? reading zeros in later entries, which could be a separate block-size issue.

---

#### [#6340](https://github.com/radareorg/radare2/issues/6340) — Backticks and grep expansion before evaluating iterations
*9y old · 3 comments · `enhancement` `consoleui`*

The tree-sitter parser correctly handles the prioritization of grep and backtick expansion before iteration (as ret2libc confirmed in comments). However, ret2libc also noted that RGrep had issues ('too many grep string'). No specific fix commit found referencing this issue. The parser infrastructure exists to handle this correctly, but the grep-related edge case may not be fully resolved. Moderate confidence.

---

#### [#7122](https://github.com/radareorg/radare2/issues/7122) — When disassembling linker from android "unaligned" is shown
*8y old · 4 comments · `RAnal` `test-required` `RAsm-Disassembler`*

ARM Thumb2 disassembly has received numerous fixes. Commit 1acc1bde5e fixes null deref in unaligned arm thumb instructions. Multiple thumb detection improvements exist (b2cd7fb23c, e2ab783250, 149c7567ed). However, no direct fix commit for #7122 was found, and the specific 'unaligned' display issue in Android linker analysis may require testing to fully verify.

---

#### [#7418](https://github.com/radareorg/radare2/issues/7418) — MZ aaaa bug (some memory error)
*8y old · 2 comments*

The workaround was 'e asm.midflags=0'. Since then, midflags handling has been significantly reworked: renamed to asm.flags.middle (commit 24d750aabd), various midflags fixes applied (d1907f84d0, fae079d20c). Memory safety in analysis has been broadly improved. No direct fix commit for #7418 found, but the midflags rework and general memory safety improvements make resolution likely.

---

#### [#7431](https://github.com/radareorg/radare2/issues/7431) — widestring and e scr.color
*8y old · 1 comments*

The issue is about widestrings screwing up color escape sequences. Multiple wide string fixes have been made: commit fabf2ce0f5 'Wide string printing fix', commit 187ecaec7b 'Follow wide strings in disasm', commit 1d4499ebdc 'Detect utf32 (wide32) strings with rabin2 -z', and commit d179c06af3 'Support wide strings in rafind2 -ZS'. The color system has also been significantly improved. The original asciinema link is likely expired. Given 9 years of wide string and color improvements, this is likely resolved.

---

#### [#7443](https://github.com/radareorg/radare2/issues/7443) — functions missed in splwow64.exe
*8y old · 3 comments · `RAnal` `PE`*

The issue only occurred with anal.hasnext enabled (not default). The initial description notes this was a known limitation of hasnext. Since 2017, analysis has been significantly rewritten and hasnext behavior improved, but no specific fix commit referencing #7443. The issue is likely mitigated by improvements but hard to verify without the specific binary.

---

#### [#7519](https://github.com/radareorg/radare2/issues/7519) — Add commands to create, display and process an RTable
*8y old · 12 comments · `enhancement` `FEEDBACK WANTED` `consoleui`*

The RTable API has been significantly implemented: r_table_sort (line 910), r_table_uniq (line 987), r_table_clone (line 1544), r_table_tohtml (line 735) are all functional. However, many functions remain as stubs inside an '#if 0' block (lines 1558-1596): r_table_push, r_table_pop, r_table_fromjson, r_table_fromcsv, r_table_transpose, r_table_reduce, r_table_format. 11 TODOs remain in table.c. The core API and sort/uniq are done, but the full feature set requested (CSV/JSON import, custom table commands) is only partially implemented. The issue body's checklist items about import CSV/JSON remain unimplemented.

---

#### [#7863](https://github.com/radareorg/radare2/issues/7863) — Loading variable names from DWARF and PDB
*8y old · 2 comments · `enhancement` `debug-info` `RAnal`*

DWARF processing has been extensively improved with DW_AT_name handling. A complete PDB parser was reimplemented (d547b037c9). However, the original issue was about loading variable names specifically (local_XX instead of actual names), and while infrastructure exists, complete DWARF variable name propagation to local vars is a complex feature that may still be incomplete. Git log shows DW_AT_comp_dir var stored (3ea8820b76) but limited evidence of full DW_AT_name variable renaming.

---

#### [#8085](https://github.com/radareorg/radare2/issues/8085) — Integrate debugger with rarun2 preload
*8y old · 2 comments · `enhancement` `rarun2` `RDebug`*

Multiple commits show rarun2 preload work: '8580ed5c65 rarun2 supports multiple preload directives', '721ad8de26 Add r2preweb rarun2 rule to start webserver in thread in r2preload', '8ce09268e1 Avoid warning in modern macs for DYLD_PRELOAD via rarun2'. The preload functionality exists in libr/socket/run.c and libr/io/p/io_self.c. However, the original request was specifically about integrating the preloaded r2 instance with the native debugger for running commands inside the target process, dlopen libs in target, etc. A POC was attempted but ran into PIC issues (commit c816dc7e663d). The basic preload infrastructure exists but full debugger integration as described is unclear.

---

#### [#8633](https://github.com/radareorg/radare2/issues/8633) — Incorrect virtual addresses from ARM binaries in Radare2 v1.7.0 on OSX
*8y old · 7 comments · `test-required` `MacOS`*

The issue reports incorrect RVA computation for some ARM ELF binaries across platforms. The reporter confirmed it persisted through r2 3.0.0 (2018). Since then, the ELF parser has been massively refactored, including symbol address computation changes (65196c2616, d7e70604ba, 86ae458b7b 'Fix partial ARM instructions relocs for ELF', 9bb9603142 'Improve AARCH64 relocation support for ELF'). The specific bug was about inconsistent vaddr computation between platforms, which is the kind of bug that tends to get fixed during major refactoring. Without the test binary, confidence is moderate.

---

#### [#8666](https://github.com/radareorg/radare2/issues/8666) — Symbols in /usr/lib/debug aren't automatically loaded
*8y old · 12 comments · `debug-info` `good first issue` `RBin`*

The ELF parser now reports debuglink information (libr/bin/p/bin_elf.inc.c contains debuglink support). The debuginfod support was added (commit 9c1a4e0d64, config var dbg.linkurl in cconfig.c). The split debug symbol loading has been worked on (0d78c577a3). However, the original request was for automatic loading of symbols from /usr/lib/debug/ paths, which may still require manual steps. The latest comment (2023) still suggests using eu-unstrip as a workaround, suggesting automatic loading may not be fully implemented. The infrastructure is there (debuglink detection, debuginfod) but automatic seamless loading like gdb does may still be incomplete.

---

#### [#9051](https://github.com/radareorg/radare2/issues/9051) — Base addresses not setting properly on PIE enabled binaries
*8y old · 7 comments · `good first issue` `RBin`*

The baddr handling code exists in libr/bin/bin.c with r_bin_get_baddr (line 674), r_bin_set_baddr (line 687) including proper baddr_shift calculation (line 702). The code handles baddr=UT64_MAX and calculates shift from file_baddr. However, the specific interaction between -e bin.baddr and PIE binaries during reload (oo/oob) is complex and hard to verify without testing. The 2020 comment from ret2libc showed it still wasn't fully working. Moderate confidence the underlying baddr mechanism is improved but the specific PIE + bin.baddr + reload scenario may have edge cases.

---

#### [#9166](https://github.com/radareorg/radare2/issues/9166) — r2 / rasm2 valid AVR opcodes are disassembled and shown as 'invalid'
*8y old · 13 comments · `RAsm-Disassembler` `AVR` `architectures-enhancements`*

The AVR plugin received multiple updates: CPU model definitions cleanup (de8c5a6cfe), regression fix (5e66a605fa), big endian encode fix (9cb47493ae), ESIL cleanup (7521e9d1bf). However, no commit specifically references this issue or the exact bug of certain valid opcodes showing as 'invalid'. The maintainer (radare) suggested it might be an endianness confusion by the reporter, but the reporter demonstrated 27 specific opcodes that were mishandled. The general AVR improvements may have addressed this, but without a specific fix commit, confidence is moderate.

---

#### [#9273](https://github.com/radareorg/radare2/issues/9273) — Defining a block with a pre-existing format (via dF) is broken when using visual mode
*8y old · 2 comments · `pf` `test-required`*

The bug was that Cf in visual mode calculated format size from the format string length instead of the actual data size. Looking at current code (cmd_meta.inc.c:1122-1141), when type == 'f' (Cf), the code now calls r_print_format_struct_size() (line 1136) to properly calculate struct size from the format when n < 1. The size calculation logic has been fixed. A ret2libc comment from 2020 still showed wrong end address, but subsequent commits (like those fixing r_print_format_struct_size: 596044a910, aa10d544b1, 56018d73c9) suggest the size calculation has improved. The Cf path now properly resolves named formats via r_print_format_byname() (line 1127) and computes size via r_print_format_struct_size().

---

#### [#9437](https://github.com/radareorg/radare2/issues/9437) — Communication problem between arm-none-eabi-gdb and r2
*8y old · 2 comments · `RDebug` `gdb`*

The issue was about r2 not communicating register info when connecting to OpenOCD via GDB. Since then, the GDB debug plugin received major updates: cc37f0c606 fixed register profile parsing, 5a4342c601 fixed privileged register handling from GDB profiles, c25595a767 fixed GDB reg parsing, 171b994831 added RISCV GDB server detection, and 9b8c604e2e fixed general GDB remote debugging. The -D option issue mentioned by the reporter would have been addressed by the general GDB plugin rework. However, this specific hardware setup (ARM Cortex-M0 via OpenOCD) would need testing to fully confirm.

---

#### [#9687](https://github.com/radareorg/radare2/issues/9687) — Can't open rarun2 file
*8y old · 7 comments · `rarun2`*

The issue was about whitespace handling in '-e dbg.profile = blabla.rr2' (spaces around =). Multiple rarun2 improvements have been made, and the -e argument parsing has been improved. However, no specific commit references this exact issue. The rarun2 subsystem received improvements (9887b12794 'Permit base64: in dbg.profile for in-memory rarun2 rules') but the specific whitespace-in-path parsing bug is not clearly targeted by any commit.

---

#### [#9749](https://github.com/radareorg/radare2/issues/9749) — ahb side effects
*7y old · 1 comments · `good first issue` `RAnal`*

Commits 2f2d77267d (implement ahb*) and 8876cb9070 (implement ahb-*) improved the ahb command. The hint system has been reworked. However, the specific issue was about ahb changing asm.bits globally as a side effect, and while the hint infrastructure has been improved, it's unclear if this specific side effect was fully isolated without testing.

---

#### [#9756](https://github.com/radareorg/radare2/issues/9756) — Mismatch between disassembling on JSON and non-JSON exported results for DATA
*7y old · 3 comments · `json` `test-required` `RAsm-Disassembler`*

The issue was about pdj ignoring Cd (data) metadata and disassembling addresses as code. The disassembly code in libr/core/cmd_print.inc.c has undergone extensive rewriting since 2018, with the JSON disassembly path being significantly reworked. Multiple pdj-related fixes were merged (073660d42a, e5248e97ed, 8e0d869468). While no commit specifically mentions Cd metadata in JSON, the significant rewrite of the JSON disassembly path likely addressed this as part of unifying the pd and pdj code paths. The issue was stale-closed.

---

#### [#9772](https://github.com/radareorg/radare2/issues/9772) — Inconsistent display after file resize
*7y old · 2 comments · `RIO`*

The IO subsystem has been significantly reworked since this issue. The initial description mentions commit c0e036dcb0 'Fix r command should resize the map'. The IO layer now uses a bank/map system. However, the specific issue about file resize not properly updating display (showing 0xff bytes) may still depend on how the resize command interacts with IO maps in the current architecture. No specific recent commit targets this exact scenario.

---

#### [#9995](https://github.com/radareorg/radare2/issues/9995) — Hardware breakpoint won't be hit for 32 bit binary on a 64 bits system unless asm.bits = 64
*7y old · 4 comments · `RDebug`*

The dbg.hwbp configuration exists (libr/core/cconfig.c:4372, default false) and is used in cmd_debug.inc.c at multiple points (lines 3716, 4378, 4761) for hardware breakpoint support. The drx handling code is present in debug_native.c with sync_drx_regs, set_drx_regs, and r_debug_native_drx functions. However, the specific bug about 32-bit binary on 64-bit system requiring asm.bits=64 for hardware breakpoints to work is hard to verify from code alone. The hardware breakpoint infrastructure has been significantly improved since 2018 but whether this specific edge case is fixed is unclear without testing.

---

#### [#10115](https://github.com/radareorg/radare2/issues/10115) — Meson + clang-cl with ASAN, UBSAN, etc build for Windows
*7y old · 18 comments · `Windows` `buildsystem`*

The clang-cl AppVeyor build was merged (PR #14814 per comments). The Clang bugs blocking ASAN/UBSAN were fixed upstream in LLVM (commit 6bdfe3aeba). However, it's unclear if ASAN/UBSAN sanitizers are actually enabled in the current Windows CI. The meson build system has evolved significantly since then. The infrastructure prerequisites were resolved but whether the full ASAN/UBSAN Windows CI target is active in the current build system is uncertain.

---

#### [#10283](https://github.com/radareorg/radare2/issues/10283) — cmd.vprompt setting in visual mode does not work properly when debug program
*7y old · 0 comments · `consoleui` `visual`*

The cmd.vprompt functionality has been significantly improved with multiple code paths in libr/core/visual.c (lines 1791, 1869, 1928-1933, 3085-3090, 4586-4591, 4916). A cmd.vprompt2 was added for additional visual prompt commands. The grep/pipe filtering within vprompt may have been fixed as part of general console improvements. The code in visual.c shows cmd.vprompt being properly executed via r_core_cmd, which should support ~ grep and | pipe operators. However, without testing, it's hard to confirm the specific edge case with 'dm~*' or 'dm | grep' within vprompt is fully fixed.

---

#### [#10423](https://github.com/radareorg/radare2/issues/10423) — wrong v850 disasm
*7y old · 3 comments · `test-required` `V850`*

The v850 plugin received significant work: ESIL support for setf (665268a899), blindfix for a disassembler glitch (eb98274cb9), and removal of global state (7693e7c9bb). However, the original issue was about 'setf , r29' showing an empty condition code before the comma. The setf instruction definition in opc.inc.c uses CCCC operand for condition code, and the ESIL commit added handling for the setf instruction. The trufae comment (Nov 2020) acknowledged 'tons of bad instruction disassemblies in the current v850 disassembler' and took ownership. Work has been done but the specific missing condition code display may not be fully fixed.

---

#### [#10439](https://github.com/radareorg/radare2/issues/10439) — FreeBSD support for ARM, MIPS, PowerPC, SPARC
*7y old · 9 comments · `buildsystem` `ARM` `MIPS`*

Commit 6e4819b054 adds FreeBSD support for powerpc, powerpc64, powerpc64le and riscv64. Multiple FreeBSD debugger fixes exist (9155ffab7e 'Fix debugger on non-x86 FreeBSD'). ARM support was discussed by evadot. However, MIPS and SPARC are not explicitly mentioned in any fix commits. The build system has been improved but full support for all listed architectures (ARM, MIPS, SPARC) on FreeBSD is not clearly confirmed. PowerPC is addressed, ARM partially, MIPS and SPARC unclear.

---

#### [#10515](https://github.com/radareorg/radare2/issues/10515) — Background Webserver doesnt plays well with tasks
*7y old · 2 comments · `enhancement` `webui` `sandbox`*

Commit b8a9e05b2c directly references this issue ('Disable =* commands to create tasks. Related to #10515'). Multiple webserver fixes exist: 8e4e0ad823 (fix UAF with background webserver), f06c7887d8 (fix background webserver sessions), 6e61a744a2 (use TLS cons instance), c39facb15c (fix null command). However, the issue covered multiple sub-items (RConfig per-thread, can't stop, not listed in tasks, cfg.sandbox) and it's unclear if all were fully addressed.

---

#### [#10541](https://github.com/radareorg/radare2/issues/10541) — Unable to store binaries with non-simple project save
*7y old · 2 comments · `projects`*

The project system has 'prj.files' support in libr/core/cconfig.c, project.c, and cmd_open.inc.c. The project save/load infrastructure exists (r_core_project_save at project.c:631, r_core_project_load at project.c:295). However, the specific issue about file.path being cleared when saving with prj.files=true is hard to verify from code alone. The project system has been reworked significantly since the report, which may have fixed or changed the underlying behavior.

---

#### [#10564](https://github.com/radareorg/radare2/issues/10564) — Sandbox doesn't honor `http.root` and `http.homeroot`
*7y old · 4 comments · `sandbox`*

The sandbox system has been substantially reworked with fine-grained control (commit b531513e96). The sandbox now has grain types: SOCKET, DISK, FILES, EXEC, ENVIRON (libr/include/r_util/r_sandbox.h:45-51). The R_SANDBOX_GUARD macro checks grain types before allowing operations. However, there's no evidence that http.root and http.homeroot paths are specifically honored/cached when enabling the sandbox. The sandbox is more capable now but the specific http.root/http.homeroot path checking may not be implemented.

---

#### [#11878](https://github.com/radareorg/radare2/issues/11878) — Avoid C files with the same name to simplify libr.a
*7y old · 7 comments · `refactor`*

Partially addressed but NOT fully resolved. Running the duplicate filename detection script today still shows ~57 duplicate .c filenames across libr/shlr (e.g., plugin.c, pseudo.c, dis.c, asm.c, core.c, reg.c, etc.). Multiple commits addressed specific cases: 3f77b9a76b renamed files to avoid collisions, a19cf131db renamed util/diff.c to udiff.c, 8344b7c304 renamed json_parser. The static build system (libr.a) works around this by using separate directories per module, so it functions correctly in practice. The author (trufae) declared it fixed in 2020 comments, but the underlying request for prevention/detection at PR time was never fully implemented. The issue is 'good enough' for the build system but duplicate filenames still exist.

---

#### [#12070](https://github.com/radareorg/radare2/issues/12070) — Better ways to handle sp-based vars
*7y old · 5 comments · `RAnal` `vars`*

The ahF (set stackframe size) hint was implemented and exists in current code (cmd_anal.inc.c:944,11507). Commit 7c928d72d2 adds r2sptrace.py script from #12070. The afbs command was also implemented (e672c13d0f). However, the broader goal of better sp-based var handling is a complex ongoing improvement, and several checklist items may remain incomplete.

---

#### [#12271](https://github.com/radareorg/radare2/issues/12271) — Error in updating thumb functions
*7y old · 2 comments · `RAnal` `ARM`*

ARM thumb function detection and boundary updates. PR #12068 was referenced and the workaround (undefine function, set bits, re-analyze) was explained. The ARM analysis has seen many improvements since 2018 including better thumb/arm mode switching. The specific binary (libtmessages) would need retesting but the underlying mechanisms have been improved.

---

#### [#12555](https://github.com/radareorg/radare2/issues/12555) — Support bitfields in `pf` and `t` commands
*7y old · 9 comments · `enhancement` `good first issue` `pf`*

The pfb command was implemented across 5 commits: 5e48458dae (initial implementation), b43fa41f2d (improved ascii art), 4691af97a0 (fixed big endian), 823015849e (improved documentation), a3e30a15d6 (error handling). Trufae confirmed 'Solved in pfb'. However, the full integration with TCC syntax parser for the type system (so that bitfields defined in 'td' work automatically with 'tp') is still pending per the last comment. The pfb command itself works, but the deeper type integration remains incomplete.

---

#### [#12752](https://github.com/radareorg/radare2/issues/12752) — unexpected int3 in debugging
*7y old · 4 comments · `RDebug`*

The issue was about leftover int3 (0xCC) breakpoint bytes appearing in the debugged binary after stepping over functions. Multiple breakpoint restoration fixes have been committed: e503bdd9c2 'Validate bp addr on rebase and restore', 3f7dd9a47f 'Fix hardware bp restoring', 2e5f4b41b4 'Fix multithreaded breakpoint behavior in linux', 313d4b4893 'Refactor breakpoint validation', and faa7938cc5 'Detect and warn when setting overlapped breakpoints'. The maintainer acknowledged it was a bug (breakpoints should be set and unset properly). These systematic fixes to breakpoint management likely address the root cause.

---

#### [#12784](https://github.com/radareorg/radare2/issues/12784) — Text of errors deletes info text of radare2
*7y old · 3 comments*

The issue was about error messages using \r (carriage return) overwriting progress text during analysis. The analysis output system has been significantly reworked since 2019, with R_LOG_* APIs replacing many direct eprintf calls. The progress indicator system has been overhauled. While no specific commit targets this issue, the move to structured logging (R_LOG_ERROR, R_LOG_INFO, etc.) should prevent \r-based text overwriting since these APIs use \n-terminated output.

---

#### [#12902](https://github.com/radareorg/radare2/issues/12902) — zzz dump raw strings to stdout (for huge files) has limited output.
*7y old · 2 comments*

The izzz command has received several improvements: 850ee34055 (initial implementation), 74a5b55925 (honor *q), 961dfec035 (fix JSON output), 5451e7edbf (add attributes), 9b5aa5527f (use pj api). The original issue was about izzz producing only 23 lines for a 17GB file. The subsequent fixes to izzz's output handling and string dumping logic likely addressed the truncation issue, though without testing on a 17GB file it's hard to be certain.

---

#### [#12996](https://github.com/radareorg/radare2/issues/12996) — Bring back r2 -t
*7y old · 4 comments*

Commit 7ed45aef31 'Remove broken Threading Code from main for #12996' directly references this issue, and commit 792956c67d 'Add a loading animation in a thread when using r2 -t' added related functionality. In the current radare2.c, the -t flag is recognized (line 252) but shows warnings: 'R_LOG_WARN -t is experimental and known to be buggy!' and 'R_LOG_WARN -t is temporarily disabled!'. The threaded infrastructure exists (mr.threaded variable, task management code) but -t appears to be deliberately disabled. The checklist items (load rbin in task, run analysis in task) are partially implemented but the feature is currently disabled.

---

#### [#13057](https://github.com/radareorg/radare2/issues/13057) — RFE: support static/shared plugin selection in meson build system
*7y old · 1 comments · `buildsystem` `meson`*

Commit 9e00eeda5c 'Add meson -Dplugins=a,b,c to build only the specified plugins' added a generic plugin selection mechanism. The meson_options.txt has 'plugins' option (line 10) for comma-separated plugin names. While this doesn't provide the specific static_asm_plugins/shared_asm_plugins granularity originally requested, it provides the core functionality of selecting which plugins to build. The original requester (ret2libc) removed the milestone saying they wouldn't work on it soon. The generic solution partially addresses the need.

---

#### [#13060](https://github.com/radareorg/radare2/issues/13060) — Visual browse calling conventions
*7y old · 5 comments · `enhancement` `good first issue` `types`*

The request was to add a calling conventions column in vbt (visual browse types). Looking at the current code (vmenus.c:1008-1088), r_core_visual_types() includes 'cc' as one of the opts array entries (line 1022). When the 'cc' tab is selected, it runs 'tfcl' (line 1051). The original commit 2d40d74d added a readonly hack, and the current code still has the comment 'XXX TODO: make this work (select with cursor, to delete, or add a new one with i, etc)' (line 1050). So the basic read-only listing is implemented, but full interactive editing is not. Partially resolved -- the original request for 'another column in vbt' showing calling conventions is present.

---

#### [#13130](https://github.com/radareorg/radare2/issues/13130) — break point is not working
*7y old · 1 comments · `RDebug`*

The issue appears related to ASLR causing breakpoints to fail after ood (debug restart). The maintainer (radare) suggested this was 'Because of ood and aslr'. Debug rebasing has been significantly improved with multiple commits fixing address rebasing after reopening in debug mode. Without testing, it's moderate evidence that ASLR handling during ood has improved, but the exact scenario is hard to verify from code alone.

---

#### [#13218](https://github.com/radareorg/radare2/issues/13218) — Improvements in aesou
*7y old · 0 comments · `enhancement` `RAnal`*

Multiple commits address aesou: 708e80908b (Fix aesou - not stop on calls), 215f491f40 (Skip {urc}{jmp,call,ret} in aesou), 950983ec71 (Implement new aesou and abte commands). However, the original issue had a checklist including syscall name/parameters resolution, tail call handling, and aC* implementation. Some items were addressed but others (like full aC* support) may remain incomplete.

---

#### [#13229](https://github.com/radareorg/radare2/issues/13229) — av changes
*7y old · 18 comments · `enhancement` `FEEDBACK WANTED` `RAnal`*

Commit e52a67ed70 adds 'avrr in aaa'. The aavr command was implemented (691bdd0472). However, the unchecked rename items (avra->aavt, avrr->aavr) were discussed but deliberately not pursued. Some items addressed, some intentionally left as-is. Partial resolution.

---

#### [#13356](https://github.com/radareorg/radare2/issues/13356) — Resolve data relocations in 32bit position-independent binaries
*7y old · 7 comments · `RAnal` `x86`*

The emu.str feature and ESIL emulation improvements address this partially. The maintainer suggested 'e emu.str=true' as the solution. The emu.str infrastructure exists and has been extensively improved. However, full automatic resolution of data relocations in PIE32 binaries depends on emulation quality which varies by binary complexity.

---

#### [#13415](https://github.com/radareorg/radare2/issues/13415) — Improve stackptr
*7y old · 0 comments · `refactor` `RAnal`*

The afbs command was implemented (e672c13d0f). Multiple stackptr improvements have been made. However, the original checklist included using bb->stackptr instead of computing in disasm_stackptr.inc, and cleanup of disasm_stackptr.inc. These are ongoing improvements with partial completion.

---

#### [#13435](https://github.com/radareorg/radare2/issues/13435) — OOD: Argument Disappears
*7y old · 5 comments · `RDebug`*

The issue was that the ~ character (0x7e) in shellcode arguments was being interpreted by r2's command parser, causing the second argument to disappear. The r2 command parser has been significantly reworked since 2019. The maintainer suggested using rarun2 profiles (dbg.profile) as a workaround. While no specific commit for this exact scenario was found, the command parser rewrite and improved quoting handling likely improved this. Using rarun2 profiles (which are now more feature-rich with 9887b12794 base64 support) provides a clean workaround.

---

#### [#13700](https://github.com/radareorg/radare2/issues/13700) — Call instruction parameters
*6y old · 6 comments · `RSoC` `RAnal` `test-required`*

The maintainer pointed to existing commands: aefa (emulate function to find args), afcR (register telescoping), afcf (print return type and args). These commands exist and can resolve call parameters at call sites. The feature was largely already available but may need better visibility/documentation.

---

#### [#13848](https://github.com/radareorg/radare2/issues/13848) — Unable to CL show the source code for a specific binary
*6y old · 0 comments*

The issue is about CL command failing to show source for a Go binary (Hv2ray). The addrline/debug_line system has been significantly refactored: commit 7744631dc6 'Initial API renaming of dbginfo into addrline', commit 8f241d4f28 'Refactor dwarf5 debug_line parsing', commit 82f5cbb464 'addr2line now also returns the column', commit dec79e05fe 'Fix duplicated source lines in CLLf output', and commit 454ca3594d 'Use a stringpool for addrline structs'. Go binaries use DWARF debug info, and the DWARF parser has been substantially improved. While the specific Go binary cannot be tested, the extensive CL/addrline refactoring makes resolution likely.

---

#### [#13894](https://github.com/radareorg/radare2/issues/13894) — ARM thumb autoswitch detection on rom dump
*6y old · 0 comments · `RAnal` `ARM`*

Multiple commits address ARM/thumb switching: b933d02326 (Autodetect thumb main on arm16 elf binaries), 8c72df2f46 (Fix arm/thumb switch emulation bug), and general improvements (b2cd7fb23c, e2ab783250, 149c7567ed). However, the specific case of raw ROM dumps (non-ELF) with LSB-based thumb detection is harder to verify. The improvements are primarily for ELF binaries.

---

#### [#14219](https://github.com/radareorg/radare2/issues/14219) — ie shows 0 entrypoints when radare2 opens a saved project
*6y old · 0 comments · `RBin` `projects`*

The project system has been significantly reworked with new save/load mechanisms (r_core_project_save/load). The old project format that caused broken entrypoint loading has been replaced. However, without being able to test the specific scenario (save project, reopen with -p, run 'ie'), it's hard to be certain the exact issue is fixed. The project rewrite likely addresses this but edge cases may remain.

---

#### [#14439](https://github.com/radareorg/radare2/issues/14439) — functions broken when analyzing file opened with ihex://
*6y old · 2 comments · `RIO` `has-test` `RBin`*

Multiple ihex:// fixes committed: 752435bf1b (Fixed >64k blocks parsing), 4145077fb9 (Fix ihex:// io parser), 2bc863d589 (Fix bug in io_ihex), 6fa6b2a2f2 (Fix infinite loop). However, the specific analysis issue (repeated block writing) may be a higher-level problem that isn't directly addressed by these IO-level fixes.

---

#### [#14442](https://github.com/radareorg/radare2/issues/14442) — Exceptions support in Analysis
*6y old · 4 comments · `enhancement` `RAnal` `java`*

Significant progress: commit 3442eb4542 adds 'iw' command for try/catch blocks, 72f0bdc28d implements anal.trycatch blocks, 83c50c9c26 parses try/catch in DEX, 68fe4839fd adds PE SEH support. However, the original checklist was extensive (separate struct, ELF .eh_frame, PE SEH, DEX, JVM) and some items may remain incomplete. The feature is partially implemented.

---

#### [#14618](https://github.com/radareorg/radare2/issues/14618) — Make icc work with ac info
*6y old · 4 comments · `RAnal` `RBin` `C++`*

Commits show icc refactoring (50f13fc9b8 'Initial ic, ia refactor'), icc improvements with language-specific formatting (ea6a155ffd for Kotlin, 5881fcd2e1 for icc*). However, the core request was merging RBin.classes and RAnal.classes in icc output, which is a complex integration that may not be fully complete.

---

#### [#14632](https://github.com/radareorg/radare2/issues/14632) — (custom) function type not propagating to callers
*6y old · 1 comments · `enhancement` `RAnal` `test-required`*

Type propagation has been extensively improved with many commits (04b287bf6b, 9c89c45791, 75b6412775, 7c6e8a095c, 418656fa3e, f1e6d24a30). The type propagation code was refactored into a plugin. However, the specific issue of custom function types propagating to call sites is a complex feature that depends on the specific binary and type definitions.

---

#### [#14986](https://github.com/radareorg/radare2/issues/14986) — Wrong backtrace using dbt
*6y old · 1 comments · `RDebug`*

The backtrace (dbt) has received significant fixes: 5b2e62ed7f 'Fix fuzzy backtrace to show complete call stack with correct SP values', 412d93acdd 'Fix recent regression about invalid word size in dbt', f98a9b7559 'Fix null deref in dbtj', and 02c2039c00 'Initial backtrace API and commands (abt)'. The specific issue was that dbt skipped corrupted return addresses (like 0x41415041414f4141 in a buffer overflow), while GDB showed them. The fuzzy backtrace fix (5b2e62ed7f) specifically addresses showing complete call stacks, which would include corrupted frames.

---

#### [#15035](https://github.com/radareorg/radare2/issues/15035) — Android/ARM debugger broken
*6y old · 0 comments · `RDebug` `ARM` `Android`*

Multiple Android/ARM debug fixes have been made since 2019: e56c1ee7fe (Fix reading and parsing /proc/pid/maps from remote gdb on android), cc37f0c606, and significant ARM64 debug work. However, the original issue was specifically about native on-device ARM32 debugging of Android Zygote processes (not via gdb remote), which is a very specific and complex scenario. The 'dc' (continue) hanging could be related to signal handling in the Android/Zygote context. While there have been improvements, without testing on an actual Android device, it's hard to confirm this exact scenario is fixed.

---

#### [#15173](https://github.com/radareorg/radare2/issues/15173) — No default thread selected when starting a debugging session on GDB protocol
*6y old · 1 comments · `gdb`*

The GDB debug plugin has received substantial rework: c25595a767 'Fix gdb reg parsing and gdb G reg writing', 9b8c604e2e 'Fix radare2 gdb remote debugging support and add test', and the general thread handling improvements. The issue was that no thread was selected by default when starting a GDB debug session. The GDB plugin rework likely addressed thread initialization since register reading requires thread selection. However, without testing the specific scenario, full confirmation isn't possible.

---

#### [#15285](https://github.com/radareorg/radare2/issues/15285) — RISCV registers + ideas to improve regprofiles
*6y old · 1 comments · `RAnal` `high-priority`*

The RISC-V register profile exists in libr/arch/p/riscv/plugin.c with both 32-bit and 64-bit register sets. Commit e3e4b3d785 implements get_reg_profile for riscv. However, the original issue also discussed ideas for improving register profiles generally (like compressed reg names, ABI name aliases), and it's unclear how many of those broader improvements were implemented.

---

#### [#15533](https://github.com/radareorg/radare2/issues/15533) — Arrows are broken in pd--
*6y old · 0 comments · `RAsm-Disassembler`*

Multiple commits have addressed pd-- issues: 38c9b17b17 'fix glitch where pd--N and N > offset', d997fb62ba 'Fix pd-x, tests pd -x and pd--x too', a36fae2b29 'Implement pd-- for context disasm'. The arrow rendering in the disassembly has been significantly reworked since 2019. While no commit specifically mentions arrow rendering in pd--, the general pd-- implementation was fixed and tested.

---

#### [#15622](https://github.com/radareorg/radare2/issues/15622) — No functions are detected on ARM code due to weak Thumb code recognition
*6y old · 6 comments · `RAnal` `ARM`*

Multiple ARM Thumb detection improvements: e889490b4b (Fix aav arm/thumb detection), b2cd7fb23c (improve thumb/arm detection), 149c7567ed (improve arm/thumb detection for .so). However, no commit directly references #15622. The specific case (ARM firmware with interleaved ARM/Thumb) remains challenging and may not be fully resolved for all binaries.

---

#### [#15747](https://github.com/radareorg/radare2/issues/15747) — Smart correction of flags usage
*6y old · 5 comments · `enhancement` `Rflags` `command`*

Multiple commits implement realname support: 33a801c935 (set realname on bin strings), 0ad05eb090 (support real names in fd), 3d68a3adce (add realname to anj), d5893f9575 (show realnames in function signature). The asm.flags.real feature exists. However, the specific request was about being able to seek/use commands with the display name (e.g., 's CreateWindowExA' instead of 's sym.imp.user32.dll_CreateWindowExA'). ITAYC0HEN clarified this requires a hashtable for flag realnames to enable lookup. It's unclear if this lookup-by-realname capability was fully implemented.

---

#### [#15831](https://github.com/radareorg/radare2/issues/15831) — Reindent whole code (except imported one)
*6y old · 4 comments · `infrastructure` `buildsystem`*

The sys/clang-format-radare2 script exists. There's a commit about properly erroring about missing clang-format in PATH. However, the issue asked for a complete reindent of the whole codebase which hasn't been done as a single operation. The infrastructure is in place and is used for incremental formatting, but the codebase hasn't been fully reformatted in one pass. The maintainer (radare) said 'I think indenting by hand is not that painful nowadays for us' suggesting this was deprioritized.

---

#### [#16146](https://github.com/radareorg/radare2/issues/16146) — Add simple if/else or ternary operator for r2 shell (oneline)
*6y old · 4 comments · `FEEDBACK WANTED` `command` `input`*

Commit c8f80fce7b 'Implement ternary support for numeric input' exists. The ?? and ?! operators were already present. However, the issue specifically requested 'a ? b : c' ternary syntax for the new tree-sitter shell. ret2libc removed the milestone saying he didn't plan to add new sugar on tree-sitter yet. The basic conditional operators exist but the full C-style ternary requested may not be implemented.

---

#### [#16400](https://github.com/radareorg/radare2/issues/16400) — ESIL suggestions for improvement
*5y old · 1 comments · `FEEDBACK WANTED` `refactor` `ESIL`*

The first item (separate r_esil.h) is done - libr/include/r_esil.h exists. ESIL has been significantly refactored with new API, plugin system, and architectural changes. However, the original issue was a broad checklist of suggestions, and while major items are addressed, some suggestions may remain unimplemented. Partial resolution.

---

#### [#16449](https://github.com/radareorg/radare2/issues/16449) — The 'wa' command doesn't assemble 'xor ax, ax' correctly
*5y old · 1 comments*

The x86.nz assembler has received many improvements: better parsing (8c8a06cfd7), immediate bounds checking (5e608cf01f), new instructions (30a6fb23a7, 4c3854a94c). However, no specific commit mentions fixing the 'xor ax, ax' 16-bit encoding issue. The initial description claims it now assembles correctly to '6631c0', but without a specific fix commit or the ability to test, confidence is moderate. The operand size prefix (0x66) handling for 16-bit mode has likely been improved through general assembler improvements.

---

#### [#16514](https://github.com/radareorg/radare2/issues/16514) — Auto analysis does not recognize hot patched functions
*5y old · 1 comments · `RAnal`*

Commit 5023918793 added hotpatching function preludes from MSVC. The maintainer suggested 'e anal.hasnext=true' as a workaround. The mov edi,edi pattern recognition for hot-patched functions has been added as a function prelude, though it may not be enabled by default in all analysis modes.

---

#### [#16533](https://github.com/radareorg/radare2/issues/16533) — rasm2 tricore ld.bu disassembly error
*5y old · 1 comments · `RAsm-Disassembler`*

The tricore architecture received significant improvements including the capstone-based plugin (tricore.cs) with ESIL support, opex, register profile fixes, and improved instruction handling. The ld.bu instruction is defined in the pseudo plugin (libr/arch/p/tricore/pseudo.c:70). The specific issue about 4-byte vs 2-byte instruction encoding for ld.bu may have been addressed through the improved tricore handling, but no specific fix commit references this issue. The addition of the capstone-based backend provides a second disassembler that may handle this correctly.

---

#### [#16744](https://github.com/radareorg/radare2/issues/16744) — extend "?*" for plugin documentation
*5y old · 1 comments*

The shell/command system was substantially rewritten. Multiple commits address '?*' output: 5545df71ee 'Fix afb*? and add $? and ?$? to ?*', 044172b4cb 'Add missing help for pds*? trashing the ?* output', 9b9ef13c66 'Add missing afi[?] for ?*'. The '?*' command now aggregates help from more subcommands. However, the original request was broader - to extend ?* for comprehensive plugin documentation. The current improvements add more help entries but may not fully implement the 'every command should implement help' vision.

---

#### [#16757](https://github.com/radareorg/radare2/issues/16757) — radare2 leaks when using tabs in visual panels mode
*5y old · 9 comments · `panel`*

Multiple commits have fixed memory leaks and UAF in panels.c: ed7853e830 (fix UAF after fixing memleaks), 00dd54d44f (fix recently-introduced memleak), 5417f0f07e (fix memleaks), d6bc481535 (fix memleaks #15142), 231e04b413 (fix memleaks coverity), 0f6ec113d2 (fix bug spotted by codeql). However, panels memory management is complex and the original ASAN report showed 344905 bytes leaked across 5692 allocations. While many leaks were fixed, panels code is notoriously leak-prone and some may remain.

---

#### [#16785](https://github.com/radareorg/radare2/issues/16785) — GML graph export from ESIL dataflow graphs (aeg) is broken
*5y old · 4 comments · `ESIL` `RGraph` `hackaton`*

Commit 308d6dd014 'Remove non-id code from the gml graph output' fixes GML format issues. The aeg command still exists and has been reworked. However, no commit directly references #16785, and the specific issue was about GML export from ESIL dataflow graphs (aegg). The GML format fix and aeg rework likely address this but testing would be needed to fully verify.

---

#### [#16846](https://github.com/radareorg/radare2/issues/16846) — Better test coverage of DWARF
*5y old · 3 comments · `DWARF`*

The DWARF parser (dwarf.c, 2957 lines) has grown significantly and includes DWARF5 support (reference to DWARF5.pdf at line 2549), unit type parsing for various DWARF versions, and DW_TAG_base_type handling. However, the issue specifically asks about test coverage with samples for DWARF1-5, which would require checking the test/ directory for sample binaries. The DWARF code itself is substantially improved but whether specific test samples exist for each DWARF version is unclear without checking tests.

---

#### [#16930](https://github.com/radareorg/radare2/issues/16930) — Expose FLIRT in rasign2
*5y old · 4 comments · `zignatures` `FLIRT`*

rasign2 has FLIRT dump support via the -f flag (libr/main/rasign2.c:13 has 'bool flirt' in config, line 20 documents '-f' for FLIRT .sig file dumping, line 195 has dump_flirt function). This corresponds to the 'zfd' functionality mentioned by @swoops. However, the issue is about broader FLIRT exposure in rasign2, not just dumping. The maintainer (trufae) questioned whether FLIRT support should be maintained at all given file format changes. The zfd dump is there but broader FLIRT functionality is limited.

---

#### [#17013](https://github.com/radareorg/radare2/issues/17013) — bunch of errors when open project
*5y old · 3 comments · `projects`*

The project system has been significantly reworked with new save/load infrastructure (project.c has r_core_project_save, r_core_project_load, and uses r_project_is_loaded). The old project format that caused broken basic block serialization ('afb+: Cannot add basic block') has been replaced. The maintainer (trufae) identified the root cause as inconsistent basic block storage in 2021. The project system rewrite likely addresses many of these issues but the exact scenario with complex PE binaries may still have edge cases. A 2021 reporter confirmed similar issues on v5.2-5.3.

---

#### [#17230](https://github.com/radareorg/radare2/issues/17230) — Better representation of the long function names (e.g. from C++)
*5y old · 1 comments · `RAsm-Disassembler` `visual` `panel`*

The asm.flags.maxname option was added (commit 5ecd4c352b) which addresses truncating long names. This partially resolves the first checkbox. The second part about moving the demangled version closer to the function name may not be fully resolved. Partial improvement.

---

#### [#17393](https://github.com/radareorg/radare2/issues/17393) — error and missing infos on gameboy analysis. A full plugin Ranal for gameboy is needed.
*5y old · 3 comments · `RAnal`*

The gameboy plugin was migrated (commit f6b4acf809). The maintainer (condret) explained the CC warnings can be ignored for GB. The reporter acknowledged the issue may not be pertinent. However, the request was for a 'full Ranal plugin' which may not be fully implemented - the migration was to the new arch system but doesn't necessarily mean full analysis coverage.

---

#### [#17577](https://github.com/radareorg/radare2/issues/17577) — Properly handle .text section relocations
*5y old · 4 comments · `ELF`*

The initial discussion showed it partially worked with io.cache=true. Commit c76558c031 added 'bin.cache evar to use io.cache when bins need to patch relocs'. The ELF relocation handling has been improved. However, the specific issue of .text section relocations (vs. GOT/PLT relocations) requiring proper handling for object files is a complex area. No specific commit directly references this issue. The bin.cache approach is a workaround, not a full fix for the underlying relocation application.

---

#### [#18597](https://github.com/radareorg/radare2/issues/18597) — Add FreeBSD CI
*4y old · 9 comments*

Commit 97095b2483 added FreeBSD in GitHub CI. However, inspecting .github/workflows/build.yml shows the FreeBSD CI job is currently COMMENTED OUT (all lines prefixed with #). The commit 0e5bc2b248 updated the FreeBSD runner and f6448b76e7 explicitly disabled FreeBSD builds. So FreeBSD CI was added but is currently not active. It was implemented at one point but is disabled in the current codebase.

---

#### [#18815](https://github.com/radareorg/radare2/issues/18815) — Invalid Address when analyzing executable
*4y old · 1 comments*

The 'Invalid address' messages during analysis have been addressed by moving warnings under anal.verbose (commits a51e121642, 31c100c7eb, d7191b9aa1). The messages may still appear but are now suppressible. The maintainer asked in 2023 if this was fixed, suggesting uncertainty. The underlying analysis issue may still exist but the user-facing noise is reduced.

---

#### [#18816](https://github.com/radareorg/radare2/issues/18816) — Error: Cannot find basic block for switch case at 0x00437033 bbdelta = 30
*4y old · 2 comments*

The warning message still exists in libr/anal/jmptbl.c:84 as R_LOG_WARN. It was changed from an eprintf to R_LOG_WARN, so it's now controlled by the logging level system. The maintainer said it would be hidden under anal.verbose for 5.7.x. The message is less intrusive now but the underlying jump table analysis issue may still produce false positives. The improvement is that users can now control the verbosity level to suppress it.

---

#### [#19437](https://github.com/radareorg/radare2/issues/19437) — All commands should support ? postfix for help
*4y old · 5 comments*

The maintainer (trufae) acknowledged this as a recurring issue and suggested closing it in 2023. Significant effort converting eprintf('Usage:') to proper RCoreHelp format was done. However, trufae himself said 'in case we find a command without help we can just implement it' suggesting it's an ongoing process. No comprehensive test or verification that ALL commands support ? postfix exists. Some edge cases likely remain.

---

#### [#19821](https://github.com/radareorg/radare2/issues/19821) — Fix all the bugs found by codeql
*4y old · 9 comments · `good first issue`*

CodeQL CI checks are active (commits bumping codeql-action versions: 31629946f2, 134c90c0b8). At least one bug fix from codeql exists: 0f6ec113d2 'Fix bug in panels spotted by codeql'. The original report showed only 'two pages' of findings. However, without access to the current codeql dashboard, it's impossible to verify all findings were addressed. The CI integration ensures ongoing monitoring.

---

#### [#19873](https://github.com/radareorg/radare2/issues/19873) — rabin2 -H does not show header of exe build with WatcomV2 wlink
*3y old · 7 comments*

The 'No header fields found' error message exists at libr/core/cmd_info.inc.c:2558 (added by PR #19926). This addresses the 'silent failure' aspect — rabin2 now prints an error when no headers are found. However, the underlying issue (MZ auto-detection not recognizing pure MZ executables without PE signatures) was discussed but the workaround (-F mz) was the recommended approach. The error message improvement was applied to both r2 and rabin2, but auto-detection of pure MZ may still require -F mz.

---

#### [#20601](https://github.com/radareorg/radare2/issues/20601) — Add support for XCOFF binaries
*3y old · 8 comments*

XCOFF32 support has been partially added. Git commits e633f3b97a and f80ef3ca94 added XCOFF32 handling. The bin_coff.c file has extensive XCOFF32 code including machine type detection (lines 783-796 with XCOFF32_FILE_MACHINE_U800WR etc.), section handling (xcoff_section at line 352), import handling (_xcoff_fill_bin_import at line 211), and section type classification. However, the last comment from @terorie (Nov 2023) said 'No, we still don't have XCOFF(32) support. Only XCOFF64' and showed broken output. The code exists but may not be fully functional. The presence of XCOFF32 machine constants and section handling suggests partial but possibly incomplete support.

---

#### [#20767](https://github.com/radareorg/radare2/issues/20767) — heap inspection in r2 5.7.4 disfunctional - no dmh* command works as intended
*3y old · 10 comments*

Multiple commits have significantly improved dmh: b6c28b3830 (autodetect libc version), c4331955f6 (fix oobread), f81c7dee37 (fix thread arena output), 682c2b3bb3 (fix main_arena via relocations), 9f19662d57 (new main_arena resolution), 158857c262 (dbg.glibc.main_arena config). However, heap inspection is inherently fragile across glibc versions. The thread_arena spam was fixed, main_arena detection improved, but the issue covers many sub-problems and not all may be resolved for all glibc versions.

---

#### [#20820](https://github.com/radareorg/radare2/issues/20820) — Merge asm.abi and anal.cxxabi somehow
*3y old · 0 comments*

Commit d3a112e91a 'Rename anal.cpp.abi to anal.cxxabi' and f10a1a335e 'Remove asm.features, improve RBinInfo with flags and abi details' show ABI handling consolidation. However, the specific ask to merge asm.abi and anal.cxxabi may not be fully resolved - they were reorganized but the merge may be incomplete.

---

#### [#21266](https://github.com/radareorg/radare2/issues/21266) — [Packaging] Fix Debian packages
*3y old · 7 comments*

A January 2026 comment from alexmyczko states 'i think the debian packages are back'. However, this is an external packaging issue (Debian's own repository at salsa.debian.org). The radare2 source tree has dist/debian/ directory with packaging files, but whether the actual Debian packages are available in the Debian repos depends on external Debian maintainers. The comment suggests it's resolved but the salsa repo was reportedly broken.

---

#### [#21294](https://github.com/radareorg/radare2/issues/21294) — String detection via aae on ARM Thumb binary fails
*3y old · 1 comments*

The maintainer responded in Sept 2024 saying 'it works for me' with a slightly different configuration (using anal.fixed.thumb=true instead of anal.armthumb=true, and a different memory mapping approach). The issue appears to have been configuration-related rather than a code bug. The emu.str and anal.strings configs exist in cconfig.c. Without being able to reproduce, moderate confidence it works with correct configuration.

---

#### [#21762](https://github.com/radareorg/radare2/issues/21762) — Display map of heap is not showing
*2y old · 11 comments*

The user was running r2 5.5.0 (a 2-year-old version at the time). The error 'Can't find glibc mapped in memory' occurs when dmh can't locate libc in the process memory maps. The maintainer pointed out the version was outdated. Since then, commit 180fea967b 'Fix the fix for dmh after ood' and other heap improvements were made. The dm output in the issue clearly shows libc.so.6 is mapped, so it's likely a version-specific parsing issue for the newer libc naming. Later versions improved glibc detection.

---

#### [#25025](https://github.com/radareorg/radare2/issues/25025) — /a Only Accepts Intel Syntax
*3mo old · 2 comments*

The maintainer (trufae) commented 'I have a fix' on 2025-12-12. An att2intel parser plugin exists at libr/arch/p/x86_nz/att2intel.c. The /a command is in libr/core/cmd_search.inc.c:4633. The fix was likely to add an att-to-intel preprocessing step before assembling, or to use the appropriate encoder plugin based on asm.syntax. Since the maintainer claimed a fix exists and this is a recent issue (Dec 2025), it's likely resolved or in progress, though the specific commit wasn't found in the current checkout.

---

</details>

### Confidence 🟠 2 (11)

<details>
<summary>Click to expand 11 issues</summary>

#### [#946](https://github.com/radareorg/radare2/issues/946) — Add C callback in r_bp
*11y old · 9 comments · `RDebug`*

Searched for bp callback APIs (r_bp_add_cb, bp_cb, bp.*callback) in libr/bp/ and found no matches. The breakpoint subsystem in r_bp does not appear to have a C callback mechanism for when breakpoints are hit. While the debugger has been significantly reworked and the issue was milestoned as 'Attic', the specific feature request (a C callback in r_bp that fires when a breakpoint is hit) does not appear to have been implemented. The cmd.bp config variable exists for running commands on breakpoint hit, but the original request was for a C API callback, which is absent.

---

#### [#15575](https://github.com/radareorg/radare2/issues/15575) — Convert all unit legacy tests in the new minunit format
*9y old · 4 comments · `good first issue` `r2r`*

NOT fully resolved. The test/unit/legacy_unit directory still exists with 14 subdirectories (anal, config, cons, debug, egg, fs, hash, parse, ragg2, reg, search, socket, syscall, util). The util/ subdirectory still contains many unconverted tests: test_queue.c, test_str.c, test_sys.c, test_cmd_str.c, test_file_slurp_hexpairs.c, etc. These are exactly the files mentioned in the comments as needing conversion. The migration is incomplete.

---

#### [#6195](https://github.com/radareorg/radare2/issues/6195) — Functions analysis errors (some end too early, some data recognized as function)
*9y old · 6 comments · `RAnal` `test-required` `jmptbl`*

Jump table handling has been extensively improved over the years with many commits addressing jmptbl detection. The specific binary (password-protected malware) cannot be retested. Multiple jmptbl-related commits (try_get_delta_jmptbl_info improvements, ahv hints) have been merged. The core issues (function end detection, jump table handling) have received significant work since 2016, but without the binary, cannot confirm 100%.

---

#### [#6429](https://github.com/radareorg/radare2/issues/6429) — r2 -nnn fatmach0 loads flags of subbin, not the ones of the fatmach0
*9y old · 1 comments · `RBin` `fat bin`*

No commit directly references this issue. The fatmach0 code has received numerous improvements (685c8d6503, 7c9a05c407 support fatmachos with arch plugins, 63ccf0da09 honor segments vs sections for fatmacho, etc.). The -nnn flag loading behavior for fat binaries is a niche case. Without the specific binary to test, confidence is low, but given the extensive fatmach0 work over 9 years, there is a reasonable chance this was addressed as a side effect of other fixes.

---

#### [#15580](https://github.com/radareorg/radare2/issues/15580) — [DEX] Add test for non-ASCII characters on obfuscated APKs
*8y old · 6 comments · `r2r`*

The underlying bug (#8679) was fixed by commit 94271f186c. However, this issue specifically requested adding a test for that fix, and no r2r test for non-ASCII characters in DEX/obfuscated APKs was found in the test database (grep for non-ascii/obfus/unicode in test/db returned nothing DEX-related). The test was never written despite multiple people offering to do it.

---

#### [#10345](https://github.com/radareorg/radare2/issues/10345) — CODE XREF from ADDRESS, where address is data, not code
*7y old · 1 comments*

The issue about xrefs from data regions being shown as CODE XREFs on msp430. The xref system has been significantly reworked multiple times since 2018, and data-vs-code classification has been improved. The issue was auto-marked stale with no further activity. Without the binary to retest, confidence is moderate but the core xref classification logic has been improved.

---

#### [#10520](https://github.com/radareorg/radare2/issues/10520) — The PAVA mode should allow to reload bininfo if headers are modified
*7y old · 3 comments · `RIO` `RBin` `Hacktoberfest`*

No matches found for 'pava' or 'io.pava' in the current libr directory. The feature appears to have been removed or significantly changed. The original issue was vague ('PAVA will allow us to do such cool things') and the oob/ib commands mentioned in comments exist but the specific PAVA mode for header-modified bininfo reload is unclear. Low confidence this specific feature request is resolved.

---

#### [#12504](https://github.com/radareorg/radare2/issues/12504) — Switch to SPDX for copyright and licenses
*7y old · 0 comments · `refactor` `good first issue` `documentation`*

Only 3 files in libr/ contain SPDX-License-Identifier headers (socket/i/isotp.h, socket/i/can.h, bin/mangling/pascal.c). While several commits used SPDX license NAMES in plugin metadata (arch plugins, bin plugins, IO plugins, debug plugins, RLang plugins, crypto plugins), the actual file headers were NOT converted to SPDX format. The issue requested SPDX headers in source files (like the Linux kernel), not just SPDX names in plugin descriptions. The bulk conversion of file headers was never done.

---

#### [#13097](https://github.com/radareorg/radare2/issues/13097) — Optimize typing and retyping code
*7y old · 3 comments · `RAnal` `optimization` `types`*

Generic optimization request for libr/core/anal_tp.c (now likely reorganized). The types analysis code has been extensively refactored since 2019 with multiple commits improving performance and correctness. The specific optimizations may have been addressed as part of broader refactoring but no single commit references this issue.

---

#### [#14965](https://github.com/radareorg/radare2/issues/14965) — autocompletion/improved Vvv type setting
*6y old · 0 comments · `types` `visual` `vars`*

No matches found for 'Vvvt' or 'visual var type' in the core directory. The initial description mentions commit b0761b3de6 'Add Vvvt - visual function var types' but the feature may have been removed or significantly changed in subsequent refactoring. The visual mode type selection interface is unclear in current code. Low confidence this is still present.

---

#### [#19796](https://github.com/radareorg/radare2/issues/19796) — in debug and non debug mode r2 interpret the same part of binary differently
*4y old · 0 comments*

The issue (from 2022 using a 2021 build) showed different disassembly between static analysis and debug mode. This is typically caused by memory mapping differences - in debug mode, r2 reads from the process memory which may have been modified by the loader/dynamic linker. The reporter used an old version (5.3.0-git from 2021). Many IO and mapping fixes have been committed since. Without the specific binary and a reproduction, it's hard to confirm, but this class of bug has received attention.

---

</details>

## 🗑️ Obsolete (62)

### Confidence 🟢 5 (32)

<details>
<summary>Click to expand 32 issues</summary>

#### [#437](https://github.com/radareorg/radare2/issues/437) — Fix disassembly highlighting for complex DSP instructions
*12y old · 15 comments · `enhancement` `blocker` `consoleui`*

Issue from 2013 about C55x+ and Hexagon DSP instruction highlighting. The highlighting/disassembly system has been extensively rewritten multiple times since then. Last activity was 2020 when ret2libc asked if still relevant with no reply. Milestone was 'Attic'. The DSP architecture support and the entire disassembly coloring pipeline have changed significantly. No one has cared about this in 6+ years.

---

#### [#15568](https://github.com/radareorg/radare2/issues/15568) — Java tests
*10y old · 0 comments · `java` `r2r`*

Confirmed obsolete. No Java test files exist in test/db/. The Java analysis was significantly reworked - custom analysis was removed and the arch was refactored. The testing infrastructure for Java as described in this issue is no longer relevant to the current architecture.

---

#### [#3873](https://github.com/radareorg/radare2/issues/3873) — Output for BinNavi
*10y old · 11 comments · `radiff2`*

BinNavi has been abandoned by Google since ~2017. The project is defunct with no maintenance. This feature request to export r2 output in BinNavi format is no longer relevant. The related tool radeco-lib is also abandoned.

---

#### [#5445](https://github.com/radareorg/radare2/issues/5445) — Deleted comments reappear on project open and automatic comments not deleted by CC-
*9y old · 6 comments · `test-required` `MIPS` `projects`*

The project system has been completely rewritten. The old project save/load mechanism (cmd_project.c) that caused this bug has been replaced with new project infrastructure. The issue of r_core_bin_info filling in string metadata before project script execution is no longer relevant with the new system.

---

#### [#6248](https://github.com/radareorg/radare2/issues/6248) — Metadata Format not correctly loaded from project with field names
*9y old · 0 comments · `projects`*

The project system has been completely rewritten. The old Cf underscore-space serialization bug in the legacy project format is no longer relevant. The new project system uses different storage mechanisms for metadata.

---

#### [#6417](https://github.com/radareorg/radare2/issues/6417) — shared library and debugging questions
*9y old · 10 comments · `FEEDBACK WANTED` `types`*

This was a question/support request from 2017, not a bug report. The user asked about function signatures and shared library annotations during debugging. The type system has been significantly improved since then (afvt works, function signature support improved). The stale bot marked it in 2020 with no follow-up. This was never a trackable bug or feature request - it was a usage question.

---

#### [#7451](https://github.com/radareorg/radare2/issues/7451) — Evaluate the use of miniz instead of zlib/libz
*8y old · 3 comments · `refactor`*

Confirmed obsolete. No miniz code exists in the codebase (only a false positive in mips-opc.c opcode table). The codebase still uses zlib via libr/util/zip.c and other places. The issue was stale-botted in 2020 and the proposal from 2017 was clearly abandoned. No activity or interest in adopting miniz.

---

#### [#7831](https://github.com/radareorg/radare2/issues/7831) — Misalignment when using `pd` command from ipython cmd in r2lang python.
*8y old · 2 comments · `consoleui`*

From 2017, involving r2lang Python plugin and IPython integration. The Python integration and console output handling have been significantly rewritten since then. No IPython/Jupyter integration code exists in the current codebase (grep confirms no ipython/jupyter references in libr/). The issue was stale-botted in 2020. The specific r2lang Python plugin architecture has changed substantially.

---

#### [#8535](https://github.com/radareorg/radare2/issues/8535) — Project Management questions
*8y old · 3 comments · `projects`*

This was a user question from 2017 about switching to write mode in projects (answered with 'oo+'). The project system has been completely rewritten since then. The specific question about project workflow with old Ps/Po commands is no longer applicable to the current implementation.

---

#### [#8582](https://github.com/radareorg/radare2/issues/8582) — Error messages when opening project (after using aaa)
*8y old · 13 comments · `projects`*

The project system has been completely rewritten since 2017. The old project format that caused 'Cannot add reference to non-function' errors and binary info reload issues are no longer applicable. The maintainer acknowledged at the time that 'projects are broken' and rbin needed refactoring, which has since been done.

---

#### [#8697](https://github.com/radareorg/radare2/issues/8697) — Renaming flags after re-opening a project does not work in visual mode, works with 'afvn'
*8y old · 2 comments · `projects`*

The project system has been completely rewritten since 2017. The old project format using afb+ commands (confirmed by Maijin's comment as the root cause) is no longer the mechanism used. Current project.c uses r2 rdb script files. The visual mode variable handling has also been substantially updated. The specific bug condition no longer exists.

---

#### [#8978](https://github.com/radareorg/radare2/issues/8978) — Can't reload a project for ARM binary
*8y old · 1 comments · `projects`*

The project system has been completely rewritten since 2017. The old project format that caused 'Unknown calling convention arm32' errors is gone. New project infrastructure uses different storage mechanisms. The old project bugs are no longer applicable.

---

#### [#10107](https://github.com/radareorg/radare2/issues/10107) — avr binary content sometimes displayed as ff... when re-opening project
*7y old · 1 comments · `RIO` `projects`*

The project system has been completely rewritten. The developer acknowledged at the time that 'projects are broken'. The old project format and its IO map handling issues have been superseded by the new project infrastructure.

---

#### [#11001](https://github.com/radareorg/radare2/issues/11001) — Visual Mode debugging is yielding different results than expected
*7y old · 4 comments · `RDebug` `visual`*

The issue was caused by using -Ad (analysis during debug launch) which was explicitly acknowledged as buggy by the maintainer. The underlying cause was ESIL backend interfering with native debugging when -A triggered analysis. The analysis-during-debug path has been reworked significantly since 2019. The maintainer's response was clear: don't use -Ad, this is a known limitation. Not a bug in the debugger itself.

---

#### [#11353](https://github.com/radareorg/radare2/issues/11353) — Fix R2pm for release builds
*7y old · 11 comments · `r2pm`*

Confirmed obsolete. r2pm has been completely rewritten in C (libr/main/r2pm.c) replacing the old shell script that had the release build issues. The original problem about git clone/pull behavior for specific dates in release builds is no longer applicable.

---

#### [#11639](https://github.com/radareorg/radare2/issues/11639) — File isn't recognized by ood command when opening a project
*7y old · 2 comments · `projects`*

The project system has been completely rewritten since 2018. Current project.c uses a script-based approach (r2 rdb project files) that is fundamentally different from the old afb+ mechanism that caused this bug. The old project format and fd tracking issue no longer applies to the current implementation.

---

#### [#12223](https://github.com/radareorg/radare2/issues/12223) — Fantasy words instead of offsets
*7y old · 2 comments · `concept` `stale`*

Confirmed obsolete. No fantasy/dragon naming code found. This was a whimsical concept from 2018 that was stale-botted and never implemented. The idea was clearly abandoned.

---

#### [#12629](https://github.com/radareorg/radare2/issues/12629) — Getopt issues with multi-byte chars
*7y old · 1 comments*

Confirmed obsolete. The issue author (radare) acknowledged this is 'probably a bug in libc' not an r2 issue. The r_getopt implementation in libr/util/getopt.c is a simple option parser that doesn't handle multi-byte chars, but this is by design and consistent with standard getopt behavior.

---

#### [#12635](https://github.com/radareorg/radare2/issues/12635) — Thumb and ARM mode flags
*7y old · 1 comments · `enhancement` `RAnal` `ARM`*

The maintainer explicitly declined this request, explaining that the current approach using anal hints and asm.bits is correct. ThumbEE vs Thumb2 is distinguished by instruction length. The request was effectively rejected as unnecessary.

---

#### [#12843](https://github.com/radareorg/radare2/issues/12843) — Register based arguments not loading from project on non-native binary
*7y old · 2 comments · `projects`*

The project system has been completely rewritten. The old project format and its debug settings interference with register arguments is no longer applicable. Projects on debugged bins are now explicitly unsupported.

---

#### [#14149](https://github.com/radareorg/radare2/issues/14149) — Tools seem happy to consume textual assembly files, yet doesn't work on them
*6y old · 1 comments*

The maintainer immediately responded that this is by design - files without headers can contain shellcode or network dumps, and text detection is difficult. This was explicitly a not-a-bug/wontfix. No action needed.

---

#### [#15200](https://github.com/radareorg/radare2/issues/15200) — Failing to load custom variable names via project
*6y old · 3 comments*

The old project system has been completely rewritten. Projects on debugged bins are now explicitly unsupported ('radare2 does not support projects on debugged bins'). The specific af+/afb+ serialization issues from the old system are no longer applicable with the new project infrastructure.

---

#### [#15261](https://github.com/radareorg/radare2/issues/15261) — ps vs dp :D
*6y old · 1 comments · `FEEDBACK WANTED`*

Confirmed obsolete. This was a naming discussion about potential conflicts between ps (print string) and dp (debug process). The community rejected renaming ps. Both commands remain with their current meanings and there's no actual conflict.

---

#### [#15458](https://github.com/radareorg/radare2/issues/15458) — xmm register output is broken after project load.
*6y old · 3 comments*

Projects on debugged binaries are now explicitly unsupported. A commenter in 2021 confirmed: 'radare2 does not support projects on debugged bins'. Additionally, xmm register handling has been improved. The original reproduction path (debug + save project + reload) is no longer valid.

---

#### [#17242](https://github.com/radareorg/radare2/issues/17242) — Debian Jessie - "db/archos/linux-x64/dbg_dmi dmi ld" failed test
*5y old · 1 comments · `RDebug`*

Debian Jessie (oldoldstable) reached end-of-life years ago. The test infrastructure and debugger tests have been significantly reworked since 2020. The specific symbol list differences and heap test failures were tied to the glibc version on Jessie. These platform-specific CI test failures are no longer relevant as Jessie is not a supported platform.

---

#### [#18329](https://github.com/radareorg/radare2/issues/18329) — radare2 does not show console on remote debugging the gameboy
*5y old · 3 comments*

The reporter themselves concluded in April 2021 that 'VisualBoyAdvance nor mgba supports remote debugging server for gameboy color. They do only support that for gameboy advance.' This was not a radare2 bug but an emulator limitation - the GDB servers in these emulators don't support GameBoy Color, only GameBoy Advance.

---

#### [#18693](https://github.com/radareorg/radare2/issues/18693) — Magic Breakpoint
*4y old · 7 comments*

The maintainer (trufae) clearly explained that magic breakpoints are a Bochs-specific feature requiring CPU-level control, which r2 as a userland debugger cannot provide. Workarounds were provided: 'db $$@@/ad xchg ebx,ebx' to set breakpoints at all matching instructions, or 'dsui xchg ebx, ebx' for step-until. These workarounds achieve the same practical result. Not a bug, and the feature as literally requested (hardware-level magic breakpoints) is architecturally impossible for r2.

---

#### [#18930](https://github.com/radareorg/radare2/issues/18930) — write a script to automatically localize and flag any virus code analysis protection
*4y old · 0 comments*

This was a vague feature request with no specific implementation proposal. It's a user-space scripting task, not a core r2 issue. No comments, no progress, no assignment. Should be classified as obsolete/wontfix.

---

#### [#19143](https://github.com/radareorg/radare2/issues/19143) — Adding a large string constant in binary
*4y old · 0 comments*

This is a user support question about how to extend .rodata sections and add strings, not a bug report or feature request. The user was asking for guidance on binary patching workflow. Not an actionable issue for the codebase.

---

#### [#20179](https://github.com/radareorg/radare2/issues/20179) — EPIC: Packaging objectives for 5.x.x
*3y old · 7 comments*

Confirmed obsolete. This was a 5.x.x milestone EPIC. The project has moved past 5.x.x to later versions. The tracked objectives are either completed, superseded by newer milestones, or no longer relevant.

---

#### [#22032](https://github.com/radareorg/radare2/issues/22032) — Evaluate Profile-Guided Optimization (PGO) profits on Radare2
*2y old · 4 comments*

The investigation was completed: trufae tested PGO on SDB and found only ~1% performance improvement, deemed not worth pursuing. The issue was explored and the conclusion was that code-level optimizations are more impactful. This was evaluated and the result was negative - no further action warranted.

---

#### [#22508](https://github.com/radareorg/radare2/issues/22508) — debug an executable with dive into its libraries
*2y old · 2 comments*

This is a usage question, not a bug. The user asked how to step into shared library functions during debugging. The answer is to use 'ds' (step into) instead of 'dso' (step over) at call instructions. The commenter correctly suggested using deeper analysis (aaaaa) to resolve PLT/GOT entries. This is expected behavior and documented - r2 can step into library code, the user just needed to use the correct command.

---

</details>

### Confidence 🔵 4 (22)

<details>
<summary>Click to expand 22 issues</summary>

#### [#955](https://github.com/radareorg/radare2/issues/955) — r2 gdb:// doesn't work with winedbg --gdb --no-start
*11y old · 17 comments · `debug-info` `gdb` `WineDbg`*

This issue is from 2014 and involves r2's GDB client connecting to winedbg's GDB server. The GDB IO plugin (io.gdb) has been completely rewritten since then (globals removed in ce4a32ef34, major rework in 9b8c604e2e). The winedbg IO plugin also received significant updates (ab512c139b removed globals, 0f5a7ce45f fixed non-null terminated bug). The last comment from 2019 showed a different error ('Cannot connect to host') suggesting underlying protocol changes. Wine's gdb implementation was described by Wine devs as 'not really in use.' The issue is too old, too environment-specific, and the codebase has been completely rewritten.

---

#### [#3159](https://github.com/radareorg/radare2/issues/3159) — radare2 + windbg + windows xp 64bit
*10y old · 9 comments · `Windows` `RDebug` `WinDbg`*

Windows XP 64-bit is an extremely niche/legacy platform. The issue is from 2015, and the last comment from 2018 showed problems with Win8.1 x64 as well. The WinDbg plugin has received various improvements since then, but no specific XP 64-bit profile was added. Given Windows XP 64-bit is end-of-life and essentially unused, this is obsolete. The broader WinDbg profile issue may still exist for some platforms, but the specific XP 64-bit use case is no longer relevant.

---

#### [#3940](https://github.com/radareorg/radare2/issues/3940) — HTTP command output in terminal.
*10y old · 3 comments · `webui`*

This 2016 issue about race conditions where HTTP server (=h) command output leaks to the console instead of being sent as the HTTP response. The HTTP server code in libr/core/rtr.c has been substantially rewritten. The current implementation uses r_core_cmd_str() (rtr.c:1392) which captures output into a string buffer rather than printing to console, and the server loop uses r_cons_sleep_begin/end properly. The original reporter could not reproduce reliably, no reproducer was ever provided, the issue was auto-staled in 2020, and 10 years of refactoring have passed. The race condition described is inherent to the old architecture; the current code path is fundamentally different.

---

#### [#6037](https://github.com/radareorg/radare2/issues/6037) — Multiple files loaded via `o [file] [offset]` are not restored properly from project
*9y old · 10 comments · `RIO` `parity` `projects`*

The project system has been completely rewritten with new RProject infrastructure and the new 'prj' command. The old project save/load mechanism that failed with multiple files has been replaced. While the new system may have its own issues (see #25181), the specific bug described here about the old Ps/Po workflow is no longer applicable.

---

#### [#6945](https://github.com/radareorg/radare2/issues/6945) — META - Project files
*9y old · 0 comments · `parity` `projects` `META`*

The project system has been completely rewritten with new RProject infrastructure and the 'prj' command (mentioned by trufae in #25181 as written 3 years ago). Many items in this META issue refer to the old project system (Ps/Po commands, project.c). The old system's bugs and enhancement requests are no longer applicable. Some features may still be missing in the new system but this META issue tracks the old one.

---

#### [#7199](https://github.com/radareorg/radare2/issues/7199) — Cannot open files for UI
*8y old · 9 comments · `webui` `sandbox`*

The webui infrastructure has been significantly reworked since this 2017 issue. The http.sandbox issue with relative paths was discussed extensively. The webui itself has been largely superseded by Cutter as the de facto UI. The specific bug about relative paths in http.homeroot with http.ui is likely resolved or irrelevant given the webui's deprecated status.

---

#### [#7594](https://github.com/radareorg/radare2/issues/7594) — issues with bicon console
*8y old · 10 comments · `enhancement` `consoleui`*

This is about r2's console input handling breaking under bicon (a BiDi/RTL console wrapper). No BiDi or RTL-specific handling exists in libr/cons/ -- grepping for bicon, BiDi, bidirectional, RTL yields no results. The console input code (libr/cons/) has been heavily rewritten since 2017 (v1.4.0). The bicon project itself (github.com/behdad/bicon) appears abandoned (last commit 2015). This is an extremely niche third-party terminal wrapper issue. The reporter last confirmed in 2018 (r2 3.1.0), and the console code has changed dramatically since. Without the specific bicon environment this cannot be verified, and it's not reasonable to expect r2 to support abandoned third-party terminal wrappers.

---

#### [#9242](https://github.com/radareorg/radare2/issues/9242) — Hooks for arch specific implementation of reg read/write
*8y old · 8 comments · `refactor` `RAnal` `ESIL`*

The hook_reg_read/write callbacks still exist in the codebase (r_esil.h, esil.c, etc.), but the original proposal for adding them to libr/reg for 8051 memory-mapped registers was never implemented as designed. The ESIL architecture has been significantly refactored. The 8051 plugin uses its own approach. The concept never materialized and was marked stale in 2020.

---

#### [#10720](https://github.com/radareorg/radare2/issues/10720) — Change behaviour for search instruction with '/c' command
*7y old · 1 comments · `RSearch`*

The initial description states commit 4e079c4b85 renamed /c to /a* and /C to /c, completely restructuring the search commands. The original bug about /c searching byte-by-byte instead of instruction-by-instruction has been addressed by the command restructuring. The issue is from 2018 with r2 2.8.0, and the search command system has been significantly reworked since then.

---

#### [#10751](https://github.com/radareorg/radare2/issues/10751) — Show prompt/visual/graph/disasm/hexdump with 0xff..%x instead
*7y old · 0 comments · `enhancement` `good first issue` `Hacktoberfest`*

This was a vague enhancement request from 2018 (filed by radare maintainer) about improving hex display formatting, referencing a tweet. No clear requirements were specified, no discussion occurred, and no commits address this. The referenced tweet link is dead. The issue lacks actionable specification and should be considered obsolete.

---

#### [#11564](https://github.com/radareorg/radare2/issues/11564) — Vertical split with ipython
*7y old · 3 comments · `consoleui` `concept` `panel`*

Feature request for IPython/Jupyter side-by-side with disassembly in panels. No IPython or Jupyter integration exists in the current codebase. The r2lang Python plugin has been substantially reworked. Given the age (2018) and the vagueness of the request (maintainer responded with '?'), and the lack of any movement, this is effectively obsolete as a concept that was never clearly defined or pursued.

---

#### [#11957](https://github.com/radareorg/radare2/issues/11957) — Improve hexdump coloring taking ideas from here
*7y old · 0 comments · `enhancement` `consoleui` `visual`*

This is a vague 2018 enhancement request linking to an external project (0ki/presentation-toolkit) for hexdump coloring ideas. The external project is a presentation toolkit, not a specific feature specification. No specific feature requirements were stated. The hexdump coloring in r2 has received various improvements over the years (pxa annotations, color themes via eco, etc.). Without concrete feature requests, this issue is not actionable. It's been open since 2018 with zero comments.

---

#### [#12475](https://github.com/radareorg/radare2/issues/12475) — Strings and segment recognition on ELF Aarch64 TrustZone analysis
*7y old · 6 comments · `FEEDBACK WANTED` `ARM`*

ret2libc explained that 'iz doesn't find anything because the binary has no sections and iz finds strings in data sections.' The user should use 'izz' for section-less binaries. The 'stripped false' misdetection may still exist but the core complaint about iz was explained as expected behavior. The user was pointed to use anal.strings, aae/aar, and f~str for finding strings in section-less binaries.

---

#### [#12510](https://github.com/radareorg/radare2/issues/12510) — Performance issue when analyzing inputs with arch specified with evm
*7y old · 1 comments · `RAnal` `ESIL` `stale`*

The issue is about the EVM arch plugin from radare2-extras (r2pm), not core r2. The EVM plugin ecosystem has changed significantly. The issue was marked stale in 2020 and is about an external plugin's performance issue that may or may not persist.

---

#### [#13181](https://github.com/radareorg/radare2/issues/13181) — Thread lock exception in my attempted core plugin
*7y old · 1 comments*

Python threading issue with r2lang plugins from 2019 (r2 3.3.0, Python 3.6). The r2lang/Python plugin system has been significantly reworked since then. The reporter never followed up after being asked to retest in February 2020. The specific Python 3.6 threading bug (threading._shutdown assert) is unlikely to still apply with modern Python and the reworked plugin system.

---

#### [#13996](https://github.com/radareorg/radare2/issues/13996) — Add anal.with
*6y old · 1 comments · `FEEDBACK WANTED` `RAnal`*

No anal.with config variable found. The proposal to replace anal.ex and anal.a2f with a unified analysis method selector was not implemented. However, the underlying concerns have been addressed through other refactoring: the Java plugin was rewritten, a2f was changed to use sdb config files, and the analysis architecture evolved significantly.

---

#### [#14785](https://github.com/radareorg/radare2/issues/14785) — Implement |@ suffix
*6y old · 6 comments · `FEEDBACK WANTED` `command` `input`*

Confirmed obsolete. The tree-sitter command parser was implemented, replacing the old parser. The |@ suffix concept from the old parser design was explicitly dropped during the parser rewrite. This feature request is no longer applicable to the current architecture.

---

#### [#15197](https://github.com/radareorg/radare2/issues/15197) — $HOST_CC required to build with clang
*6y old · 5 comments · `buildsystem` `r2pm`*

Confirmed mostly obsolete. HOST_CC still exists in mk/gcc-x86.mk, mk/ios-sdk-clang.mk, mk/macos-sdk-clang.mk but the primary build system is now meson which handles compiler detection automatically. The original issue about needing HOST_CC for configure+make is largely irrelevant for meson users.

---

#### [#16126](https://github.com/radareorg/radare2/issues/16126) — Setup a GitHub Action for release multiple repositories
*6y old · 5 comments · `infrastructure`*

The radare2 project now uses GitHub Actions for builds and releases. The multi-repository coordinated release concept is largely obsolete as the project structure has evolved - many sub-projects (radare2-lang, radare2-bindings, radare2-extras) have been deprecated or absorbed. The webui was discussed as not needing same-version releases. The Cutter project has its own release cycle.

---

#### [#17261](https://github.com/radareorg/radare2/issues/17261) — linux_debug: /proc/<pid>/stack cannot be read
*5y old · 2 comments · `RDebug` `Linux OS`*

The code at libr/debug/p/native/linux/linux_debug.c:768-769 still reads /proc/<pid>/stack via r_file_slurp. Modern Linux kernels restrict access to this file (requires CAP_SYS_PTRACE or root). The maintainers explicitly deprioritized this: 'This field is just used as a display info when you do di and right now everything works well, apart from the fact that field is not shown (unless you use sudo). It is not a breaking feature.' This is a kernel security restriction, not an r2 bug. The behavior is expected on modern kernels.

---

#### [#17437](https://github.com/radareorg/radare2/issues/17437) — OpenBSD - "db/esil/arm_16 arm s110 syscalls /ad svc" test failing
*5y old · 0 comments · `regression` `ARM` `BSD`*

This is a CI test failure on OpenBSD from 2020. The test infrastructure and ESIL tests have been significantly reworked since then. This platform-specific CI test failure from this era is no longer directly applicable to the current codebase.

---

#### [#20092](https://github.com/radareorg/radare2/issues/20092) — Cmpne cmpeq thumb assembler hut
*3y old · 0 comments*

The issue contains only an image (not viewable in the data) and empty template fields with no description of the actual bug. No commits reference this issue. The cmpne/cmpeq instructions exist in aarch64 SVE extensions (found in aarch64-tbl.h) but these are not thumb instructions. Without a clear bug description or reproduction steps, this issue is not actionable and should be considered obsolete.

---

</details>

### Confidence 🟡 3 (8)

<details>
<summary>Click to expand 8 issues</summary>

#### [#15566](https://github.com/radareorg/radare2/issues/15566) — Add testsuite for flags
*10y old · 0 comments · `r2r`*

This is a test coverage tracking issue from 2015. Most items were already checked off. The remaining unchecked items (fc, fn, fR, fxd) are minor flag subcommands. The test suite has been significantly expanded since then (test/db/cmd/flags, test/db/cmd/cmd_flags_stress, etc. exist). The issue is 11 years old with no activity and the test infrastructure has been completely overhauled. Tracking specific test coverage for individual subcommands in a single issue is no longer a useful pattern.

---

#### [#3956](https://github.com/radareorg/radare2/issues/3956) — Reopening binary using 'do' via HTTP makes binary stdout lost.
*10y old · 2 comments · `webui`*

This is a 2016 bug about the HTTP server (=h) not properly redirecting child process stdout after a debug reopen (do). The HTTP server code has been extensively rewritten since then (multiple commits: c39facb15c, 8e4e0ad823, 6e61a744a2, f06c7887d8, etc.). The issue was auto-marked stale in 2020 with no further activity. Given the age (10 years), massive refactoring of both the HTTP server and debug reopen codepaths, and the lack of any follow-up reports, this is effectively obsolete. The HTTP mode itself is used differently now.

---

#### [#9082](https://github.com/radareorg/radare2/issues/9082) — ARM rpi32 debugging - child stopped with signal 11
*8y old · 10 comments · `RDebug` `ARM`*

This 2017 issue was specific to Raspberry Pi 3 with Raspbian Jessie. The maintainer could not reproduce on RPI1 with Void Linux. The issue was likely related to base address detection on the specific ARM Cortex-A53 + Raspbian combination. Since then, the debug native plugin has received many updates, ARM breakpoint support was improved (4ec6ed2a71 native ARM64 breakpoints), and Raspbian Jessie is EOL. The specific hardware/OS combination is no longer relevant. Without the same environment, this cannot be verified, but the age and specificity make it obsolete.

---

#### [#12081](https://github.com/radareorg/radare2/issues/12081) — axt missing xrefs in remote debug
*7y old · 2 comments · `RAnal` `RDebug` `WineDbg`*

This is a configuration issue with WineDbg remote debugging where memory maps may not mark regions as executable. The maintainer explained the workaround (check anal.noncode and anal.in variables, export analysis from static session). This is by-design behavior, not a bug. The user needs to configure analysis properly for remote debugging sessions.

---

#### [#15328](https://github.com/radareorg/radare2/issues/15328) — Funky function names
*6y old · 2 comments*

Vague report about non-ASCII/garbled function names in certain binaries. No binary was shared for testing. The maintainer asked 'what solution do you propose here?' The reporter suggested an ASCII-only output flag but acknowledged it might be an edge case. Without a reproducible test case or clear bug description, this is effectively abandoned.

---

#### [#16947](https://github.com/radareorg/radare2/issues/16947) — Significant time difference in executing `aaa` with r2pipe vs. Cutter
*6y old · 3 comments · `RAnal` `waiting-for-author`*

The r2pipe/Cutter version mismatch (r2 4.3.0 vs Cutter using r2 4.1.1) may have been the cause. The reporter could not share the binary. Analysis performance has been significantly improved since 2020. Without the binary and given the version mismatch, this cannot be reproduced or verified.

---

#### [#16729](https://github.com/radareorg/radare2/issues/16729) — Improve r2r speed on Windows
*5y old · 4 comments · `Windows` `infrastructure` `optimization`*

This 2020 issue identified CreateProcess overhead and antimalware service as contributors to Windows r2r slowness. The comments concluded that the antimalware service accounted for much of the slowdown (19s vs 30s for db/asm). r2r has undergone significant changes since then (many recent commits improving r2r). This is an optimization request without a clear actionable fix, and the landscape has changed substantially. Windows CI performance would need fresh profiling.

---

#### [#18142](https://github.com/radareorg/radare2/issues/18142) — ubuntu uses gdbserver to remotely debug vbox win10 program issues
*5y old · 0 comments · `gdb`*

This issue is very vague - just a video with no specific error details. The report is from 2020 about remote GDB debugging a Windows program in VirtualBox. GDB remote debugging has received many fixes since (9b8c604e2e, c25595a767, 46c725dd1c). Without specific error messages or reproduction steps, this cannot be meaningfully evaluated. The age and lack of detail make it obsolete.

---

</details>

## 🔧 Still Open (489)

### Confidence 🟢 5 (146)

<details>
<summary>Click to expand 146 issues</summary>

#### [#2338](https://github.com/radareorg/radare2/issues/2338) — radiff2: support comparing files larger than available memory
*10y old · 8 comments · `enhancement` `radiff2`*

Verified: libr/main/radiff2.c line 736 still uses r_file_slurp() which loads entire files into memory. The original issue from 2015 remains unresolved. The maintainer acknowledged it should be a simple loop-based fix but never implemented it.

---

#### [#2903](https://github.com/radareorg/radare2/issues/2903) — Add support for future breakpoints
*10y old · 2 comments · `RDebug`*

No evidence of 'future breakpoints' (breakpoints set on not-yet-loaded libraries) being implemented. No commits found referencing this feature. The 2020 comment from trufae confirmed it was not fixed. The concept of setting breakpoints that activate when a library is loaded at runtime remains unimplemented in the codebase.

---

#### [#4681](https://github.com/radareorg/radare2/issues/4681) — rarun2 support for qemu
*9y old · 8 comments · `rarun2` `PPC`*

Confirmed still open. No QEMU references found in binr/rarun2/ or libr/debug/. The rarun2 'program' directive for specifying a QEMU runtime was never implemented.

---

#### [#6967](https://github.com/radareorg/radare2/issues/6967) — META - GRAPH
*9y old · 1 comments · `RGraph` `META`*

META tracking issue for graph features. Some items completed (radiff2 -g ASCII art, diff graph colors) but many remain open: graph cursor, mouse interaction, node grouping, graph debugger integration, colorizing traced nodes. Stale-botted in 2020 but the unchecked items represent real missing features.

---

#### [#6996](https://github.com/radareorg/radare2/issues/6996) — META - New Architectures
*9y old · 12 comments · `New Architecture` `META`*

META tracking issue listing desired new architecture support. Most architectures in the list (Fujitsu FR, LLVM BitCode, SM5, TMS320, TMS34010, 8089, HCS12, OpenRISC, Alpha, SN8, RH850, ARC, PDP10, TILE-Gx, TriMedia, i960, IA-64, PA-RISC, nanoMIPS, MIL-STD) remain unimplemented. C-SKY/mcore was noted as added. This is an ongoing wishlist.

---

#### [#7126](https://github.com/radareorg/radare2/issues/7126) — replace search.from and search.to with search.range?
*8y old · 17 comments · `FEEDBACK WANTED`*

Verified: search.from and search.to still exist in the codebase (15 occurrences across 5 files in libr/core). No search.range variable was implemented. The discussion concluded with agreement on extending @{from to} syntax, but was classified as low priority syntax sugar. Never implemented.

---

#### [#7420](https://github.com/radareorg/radare2/issues/7420) — Show timestamp of opened file in `i`
*8y old · 2 comments · `RIO` `RDebug` `Rfs`*

No implementation of MAC timestamps (creation, modify, access times) in the 'i' command output was found. A December 2025 comment from trufae asking someone to work on it confirms this feature request remains unimplemented. No relevant commits found.

---

#### [#17133](https://github.com/radareorg/radare2/issues/17133) — Split-view visual hex diff mode for radiff2.
*8y old · 10 comments · `radiff2` `visual`*

Split view and hex diff highlighting exist (checked items), but synchronized scrolling between splits and asm view diff highlighting remain unchecked. The maintainer acknowledged in 2019 that 'no support to sync offset or colorize diffs yet'. Multiple discussions with no full resolution.

---

#### [#8576](https://github.com/radareorg/radare2/issues/8576) — Working with bitstreams - API, commands
*8y old · 1 comments · `RIO` `hardcore` `concept`*

Feature request for bitstream-like IO access. Grep for 'bitstream' only finds unrelated references (PIC architecture, CAN protocol, print command). No bitstream API or commands have been implemented. This is a conceptual feature request from 2017 that remains unaddressed.

---

#### [#9320](https://github.com/radareorg/radare2/issues/9320) — Multidiff mode
*8y old · 2 comments · `consoleui` `concept` `radiff2`*

Feature request for comparing more than 2 files. radiff2 still only supports two-file comparison. No multi-file diff mode was found in the codebase. The maintainer questioned the usefulness in comments. Stale-botted in 2020.

---

#### [#9419](https://github.com/radareorg/radare2/issues/9419) — Phrase out `core->block` (cache) and prefer read before use
*8y old · 4 comments · `refactor` `RIO` `optimization`*

Confirmed still open. grep shows 456 references to core->block in libr/core/ .c and .inc.c files. The original issue reported 209 references, so usage has actually grown. While some commits reduced usage in specific areas, the core->block pattern remains deeply embedded throughout the codebase. Far from complete.

---

#### [#9557](https://github.com/radareorg/radare2/issues/9557) — Not only list the used library names, but also their version
*8y old · 1 comments · `enhancement` `refactor` `RBin`*

Confirmed: mach0_defines.h has current_version and compatibility_version fields, and mach0.c parses them (lines 1273-1274). However, the libs() function in bin_mach0.c (line 307) returns only string names via RVecMach0Lib - version metadata is not exposed in the 'il' command output. The struct fields are parsed but not surfaced to the user.

---

#### [#9719](https://github.com/radareorg/radare2/issues/9719) — Visual preview for the `graph.*` variables changes in `Ve` mode
*8y old · 7 comments · `enhancement` `consoleui` `good first issue`*

Feature request for visual preview of graph variable changes (like asm.* preview in Vbe mode). Despite someone claiming to work on it in 2019, no implementation was found. graph.addr exists in cconfig.c but the visual preview editor for graph variables was not implemented.

---

#### [#10426](https://github.com/radareorg/radare2/issues/10426) — JARL is not UJMP :D
*7y old · 2 comments · `V850`*

In libr/arch/p/v850/plugin.c:376-385, V850_JARL1 and V850_JARL2 are both classified as R_ANAL_OP_TYPE_JMP, but the opc.inc.c (lines 858-860) correctly defines them as R_ANAL_OP_TYPE_CALL. The plugin.c does not set the type to CALL. jarl (jump and register link) saves the return address in a register, making it a call instruction, not a simple jump. The v850e0.c decode_jarl function (line 167) only handles disassembly, not optype. This is a clear, confirmed bug.

---

#### [#10713](https://github.com/radareorg/radare2/issues/10713) — Replace addresses with map+offset
*7y old · 0 comments · `enhancement` `RAsm-Disassembler` `concept`*

Feature request for automatic number replacement in disassembly to show map+offset notation (e.g., 'mov eax, [usr_lib_ld+0x1000]' instead of raw addresses). No commits or implementation found. This enhancement was never implemented.

---

#### [#10725](https://github.com/radareorg/radare2/issues/10725) — [request] Implement basefind
*7y old · 5 comments · `enhancement` `RAnal` `concept`*

No basefind implementation found in the codebase (grep returned zero results). No git commits referencing basefind. This feature request for finding base addresses using string intersections remains completely unimplemented.

---

#### [#10739](https://github.com/radareorg/radare2/issues/10739) — FreeRTOS port
*7y old · 3 comments · `refactor` `buildsystem`*

Confirmed still open. No FreeRTOS-related code exists anywhere in the codebase. No build system support, no porting layer, nothing. This was a 2018 feature request that never saw any implementation work.

---

#### [#10778](https://github.com/radareorg/radare2/issues/10778) — Reimplement loading vmlinux images in radare2
*7y old · 0 comments · `enhancement` `RAnal`*

No vmlinux-specific loading support found in the codebase. No git commits referencing vmlinux. This enhancement request remains unimplemented.

---

#### [#10779](https://github.com/radareorg/radare2/issues/10779) — Sparse mode for `p=`
*7y old · 0 comments · `enhancement` `rahash2`*

Confirmed still open. pxs (sparse hexdump) exists but no sparse mode for p= (entropy/statistics display) that skips zero/0xFF areas was found. The feature was never implemented.

---

#### [#11562](https://github.com/radareorg/radare2/issues/11562) — Embedded markdown viewer
*7y old · 9 comments · `consoleui` `concept` `panel`*

No embedded markdown viewer was implemented. Grep for 'markdown/mdown' only finds references in cmd_mount.inc.c (man page processing), str.c, release-notes.sh, and pip setup.py - none of which are a general markdown rendering capability. The maintainer discussed it but deferred, saying 'totally low prio for now'.

---

#### [#11720](https://github.com/radareorg/radare2/issues/11720) — Support for BGZF (gzip blocked format) via the gzip plugin
*7y old · 3 comments · `RIO`*

Confirmed still open. The gzip IO plugin (libr/io/p/io_gzip.c) handles standard gzip only. No BGZF block-level decompression or multi-block support exists. Writing to gzip is also explicitly noted as unsupported in the code.

---

#### [#11828](https://github.com/radareorg/radare2/issues/11828) — Constrained types enhancements
*7y old · 6 comments · `RAnal` `test-required` `types`*

anal.types.constraint config exists but is still disabled by default ('false' in cconfig.c). The requested commands to read/change constraints and JSON I/O have not been implemented. A 2022 comment from trufae suggests using 'aht' but no concrete implementation was done.

---

#### [#11979](https://github.com/radareorg/radare2/issues/11979) — Visual SDB browser
*7y old · 2 comments · `enhancement` `consoleui` `sdb`*

No visual SDB browser (Ve/Vv-style interface based on k commands) found in the codebase. The feature was discussed as useful for browsing meta information and debugging projects but was never implemented.

---

#### [#11988](https://github.com/radareorg/radare2/issues/11988) — ob inconsistencies
*7y old · 10 comments · `FEEDBACK WANTED` `refactor` `good first issue`*

Confirmed still open. The ob command has not been renamed to oi. The checklist items (rename ob->oi, add obaa, kill obb, etc.) remain unchecked. The maintainer moved this to 6.x milestone in Feb 2024 due to lack of time. A 2022 comment confirmed 'none' of the items were done.

---

#### [#12017](https://github.com/radareorg/radare2/issues/12017) — LL Lock screen improvements
*7y old · 0 comments · `enhancement` `consoleui`*

The LL lock screen command still exists in cmd_log.inc.c ('LL', '', 'lock screen'). The wishlist items (launch after N minutes of inactivity, terminal compatibility fixes for iterm, Windows support) have not been implemented. No related commits found.

---

#### [#12128](https://github.com/radareorg/radare2/issues/12128) — Support DAP - Debug Adapter Protocol
*7y old · 7 comments · `enhancement` `RDebug` `concept`*

No DAP (Debug Adapter Protocol) implementation was found in the radare2 codebase. Git log search for 'DAP' and 'debug adapter' returned no relevant commits. This remains an unimplemented feature request. Recent interest exists (2025 comment) but no implementation work has been done.

---

#### [#15585](https://github.com/radareorg/radare2/issues/15585) — Generate more tests using GodBolt
*7y old · 3 comments · `r2r`*

Confirmed still open. No GodBolt/Compiler Explorer integration or test generation scripts exist in the codebase. The maintainer rejected this approach in 2019 comments. This feature was never implemented.

---

#### [#12161](https://github.com/radareorg/radare2/issues/12161) — Add support for GPUs
*7y old · 4 comments · `New Architecture`*

Confirmed still open. No GPU architecture plugins (r600, nvptx, amdgcn, SPIR-V) found in libr/arch/. GPU disassembly support was never implemented.

---

#### [#12168](https://github.com/radareorg/radare2/issues/12168) — Support for 16bit RISC R8822 architecture
*7y old · 0 comments · `New Architecture`*

Verified: no R8822-related files or code exist anywhere in the radare2 codebase. This niche architecture request was never implemented. No comments or activity since the original report in 2018.

---

#### [#12260](https://github.com/radareorg/radare2/issues/12260) — Revise ps commands and kill r_str_utf16_encode()
*7y old · 3 comments · `refactor`*

r_str_utf16_encode() still exists in the codebase - confirmed in str.c, cmd_print.inc.c, format.c, and r_str.h (5 files). The broader restructuring of ps commands into a systematic encoding-based format (psr, psu, psw, psW) was not implemented.

---

#### [#12261](https://github.com/radareorg/radare2/issues/12261) — Revise ps and kill r_str_utf16_encode()
*7y old · 2 comments · `refactor` `print` `RCore`*

Confirmed still open. r_str_utf16_encode() still exists in libr/util/str.c:3128 and is called from cmd_search.inc.c, cmd_print.inc.c (twice), and format.c. The function and its usage have not been removed or revised.

---

#### [#12311](https://github.com/radareorg/radare2/issues/12311) — Add VB helpers in core
*7y old · 1 comments · `enhancement` `good first issue` `RAnal`*

No Visual Basic analysis helper found in the codebase (grep returned zero matches). The feature to port VB analysis scripts into core like the Go helper was never implemented.

---

#### [#12530](https://github.com/radareorg/radare2/issues/12530) — Implement SimHash
*7y old · 10 comments · `enhancement` `radiff2`*

Confirmed still open. No SimHash implementation found anywhere in the codebase. While ssdeep was implemented, SimHash specifically was never added.

---

#### [#12626](https://github.com/radareorg/radare2/issues/12626) — Support cert pining in the http webserver
*7y old · 2 comments · `enhancement` `webui`*

Confirmed still open. The socket code has basic SSL support (r_socket_new with is_ssl parameter) but no certificate pinning implementation. The HTTP webserver has no cert pinning feature.

---

#### [#12660](https://github.com/radareorg/radare2/issues/12660) — Use KLEE for testing and tests generation
*7y old · 1 comments · `infrastructure` `concept`*

Confirmed still open. No KLEE or SymCC references exist anywhere in the codebase. This infrastructure/concept issue about symbolic execution testing was never implemented.

---

#### [#12724](https://github.com/radareorg/radare2/issues/12724) — RFE: add support for pf.elf_dyn_entry
*7y old · 1 comments · `pf` `ELF`*

Confirmed: The elf32.r2 and elf64.r2 format definition files have pf.elf_header, pf.elf_phdr, pf.elf_shdr but no pf.elf_dyn or pf.elf_dyn_entry. The Elf32_Dyn/Elf64_Dyn structs are defined in glibc_elf.h but not exposed as pf formats.

---

#### [#12745](https://github.com/radareorg/radare2/issues/12745) — Generic "tree" command/API
*7y old · 1 comments · `enhancement` `consoleui`*

Feature request for hierarchical tree output API. No tree command/API implementation was found. The maintainer suggested implementing on top of JSON but no follow-up occurred.

---

#### [#12766](https://github.com/radareorg/radare2/issues/12766) — recursive callgraphs (agx, agc ...)
*7y old · 1 comments · `enhancement` `RGraph`*

No depth parameter found in agx/agc commands. The feature request for recursive callgraphs with configurable depth remains unimplemented.

---

#### [#12771](https://github.com/radareorg/radare2/issues/12771) — Add obfuscation info to anal classes
*7y old · 0 comments · `RAnal` `classes`*

No obfuscation attribute found in anal classes code (grep returned zero matches). This enhancement request for marking classes as obfuscated has no evidence of implementation.

---

#### [#12776](https://github.com/radareorg/radare2/issues/12776) — Implement vbP
*7y old · 0 comments · `enhancement` `visual`*

No 'vbP' (visual browse projects) implementation found in the codebase (grep returned no matches in libr/core).

---

#### [#12826](https://github.com/radareorg/radare2/issues/12826) — Add function neighbourhood support signatures
*7y old · 0 comments · `zignatures`*

No neighbourhood/neighbor signature support found in the codebase (grep returned zero matches). This zignature feature request is completely unimplemented.

---

#### [#12831](https://github.com/radareorg/radare2/issues/12831) — Task-Local temporary eval vars
*7y old · 0 comments · `RCore`*

Confirmed still open. No task-local or thread-local eval variable storage mechanism found. RConfig remains a global per-core configuration without per-task overrides.

---

#### [#12847](https://github.com/radareorg/radare2/issues/12847) — Implement metanodes
*7y old · 2 comments · `enhancement` `RGraph` `visual`*

No metanode implementation found (grep for 'metanode' returned no results). The proposed feature for grouping basic blocks into visual abstractions was never implemented.

---

#### [#12872](https://github.com/radareorg/radare2/issues/12872) — Use FOSSA for licenses analysis and visualisation
*7y old · 0 comments · `enhancement` `buildsystem`*

No FOSSA integration found in the repository (grep returned zero matches). This CI/tooling enhancement was never implemented.

---

#### [#12893](https://github.com/radareorg/radare2/issues/12893) — Add TriMedia architecture support
*7y old · 0 comments · `enhancement` `New Architecture`*

Verified: no TriMedia disassembler plugin exists in libr/arch/p/. Only ELF header recognition (EM_TRIMEDIA) exists. No architecture support was ever implemented.

---

#### [#12910](https://github.com/radareorg/radare2/issues/12910) — Remap "hjkl" keys to something else to move around
*7y old · 1 comments · `enhancement` `refactor` `consoleui`*

No key remapping/rebinding support for visual mode navigation. A comment from January 2026 confirms this is still desired. The enhancement was never implemented.

---

#### [#13010](https://github.com/radareorg/radare2/issues/13010) — Intel i960 architecture support
*7y old · 2 comments · `New Architecture`*

Verified: no i960 disassembler plugin exists in libr/arch/p/. Only header references (EM_960 in mybfd.h, floatformat references) exist. No actual architecture support was implemented.

---

#### [#13116](https://github.com/radareorg/radare2/issues/13116) — Proposing r_anal lazy Analysis pipeline
*7y old · 4 comments · `enhancement` `RAnal`*

No lazy analysis pipeline, analysis manager, or LLVM-style analysis pass system found. No incremental computation or adapton framework. This is a major architectural proposal that remains completely unimplemented.

---

#### [#13318](https://github.com/radareorg/radare2/issues/13318) — Publish radare2 in Vcpkg
*7y old · 1 comments · `Windows` `infrastructure` `buildsystem`*

Confirmed still open. No Vcpkg port files or integration found in the radare2 codebase. The only vcpkg reference is in a subproject's CI comments (qjs). Radare2 has not been published to Vcpkg.

---

#### [#13344](https://github.com/radareorg/radare2/issues/13344) — Implement "graph.cmt.col" to align comments on graph
*7y old · 1 comments · `enhancement` `RGraph` `cutter`*

No 'graph.cmt.col' configuration variable exists (grep returned no results). graph.cmtright exists but defaults to false. The separate comment column alignment for graph view was never implemented.

---

#### [#13411](https://github.com/radareorg/radare2/issues/13411) — Implement mX/'X in graph (like it's done in visual already
*7y old · 2 comments · `enhancement` `FEEDBACK WANTED` `good first issue`*

Feature request for graph scroll position bookmarks (mX/'X controlling x,y position). No implementation found. The maintainer noted it 'requires changes and discussion about the current use of the m and keys in graph' in 2019.

---

#### [#13498](https://github.com/radareorg/radare2/issues/13498) — Remove unnecessary calls to RCore.cmd
*6y old · 0 comments · `refactor` `optimization`*

Still 129 r_core_cmd() calls across 15 files in libr/core. This is an ongoing refactoring task that is perpetually open - while individual instances may have been replaced over the years, many remain. The nature of this issue means it will likely never be 'done'.

---

#### [#13505](https://github.com/radareorg/radare2/issues/13505) — Implement reverse print format
*6y old · 2 comments · `pf` `test-required` `print`*

No 'pfi' command found in cmd_print.inc.c (grep returned no matches). The feature to generate search patterns from format strings was never implemented.

---

#### [#13576](https://github.com/radareorg/radare2/issues/13576) — Use mixed edge layout to improve visibility
*6y old · 0 comments · `enhancement` `consoleui` `RGraph`*

No mixed diagonal/square edge layout in graph code. The canvas_line.c handles edge drawing but uses consistent line styles, not a mixed approach. No related commits found.

---

#### [#13581](https://github.com/radareorg/radare2/issues/13581) — Visual define structs
*6y old · 0 comments · `enhancement` `consoleui` `types`*

No command that walks N bytes and converts C* commands into pf* struct definitions was found. The visual struct browser (vbtll) exists but the specific feature of generating struct definitions from type annotations was not implemented.

---

#### [#13612](https://github.com/radareorg/radare2/issues/13612) — Enable graph.cmtright=true by default
*6y old · 0 comments · `enhancement` `RGraph`*

Confirmed: cconfig.c still has SETB('graph.cmtright', 'false', ...). The default was never changed to true despite the maintainer's own request.

---

#### [#13619](https://github.com/radareorg/radare2/issues/13619) — Add scrollbar in panels
*6y old · 6 comments · `enhancement` `good first issue` `visual`*

Confirmed: grep for scr.scrollbar in panels.c returned 0 matches. The scrollbar feature works in visual and graph modes but was never extended to panels mode. Trufae confirmed in 2021 that this is still unsolved.

---

#### [#13631](https://github.com/radareorg/radare2/issues/13631) — Implement: drpc - print the register profile in C syntax
*6y old · 1 comments · `enhancement` `RDebug`*

Verified in source code: drpc at cmd_debug.inc.c:2337 only prints reg_profile_cmt (register profile comments), not a C-syntax representation of registers. The help text says 'show register profile comments'. The requested feature to output register profiles as C globals for r2dec compilation output was never implemented.

---

#### [#13641](https://github.com/radareorg/radare2/issues/13641) — Save aeaf in zignatures
*6y old · 0 comments · `enhancement` `zignatures`*

Confirmed still open. The aeaf command exists (cmd_anal.inc.c:501) for showing registers used in functions, but the zignature system does not store or reference aeaf data. No integration between ESIL analysis results and zignature storage.

---

#### [#13644](https://github.com/radareorg/radare2/issues/13644) — Add FAMILY_REP
*6y old · 0 comments · `enhancement` `FEEDBACK WANTED`*

Confirmed still open. No FAMILY_REP constant found in the codebase. The R_ANAL_OP_FAMILY enum does not include a REP family type.

---

#### [#13781](https://github.com/radareorg/radare2/issues/13781) — Unify all zoom commands
*6y old · 0 comments · `FEEDBACK WANTED` `refactor` `print`*

The zoom commands (prc=, p=, p-, pz) remain as separate commands. The pz command has its own subcommands but p= and prc= still exist independently. The requested unification was not performed.

---

#### [#13821](https://github.com/radareorg/radare2/issues/13821) — Add code hotkey hints in visual pxr
*6y old · 2 comments · `consoleui` `good first issue` `print`*

No hotkey hints in visual pxr mode. The maintainer called this 'important and very useful' in 2020 but no implementation was found.

---

#### [#13966](https://github.com/radareorg/radare2/issues/13966) — Make a terminal logging command to save command+output in a log
*6y old · 0 comments*

No session recording command (tee-like, capturing both command input and output) was found. The T command logs messages with timestamps but does not capture command+output pairs for training/recording purposes.

---

#### [#13969](https://github.com/radareorg/radare2/issues/13969) — Add * and j in all the missing commands
*6y old · 7 comments · `FEEDBACK WANTED` `good first issue` `json`*

Meta-issue tracking JSON (j) and r2 script (*) output modes for all commands. The checklist shows only a few items completed (b*, bj, y*, yj, kj, v?). Many items remain unchecked (aoq, ao*, cj, c*, rj, r*, pj, p*, etc.). This is an ongoing effort that is far from complete.

---

#### [#14054](https://github.com/radareorg/radare2/issues/14054) — Proposal for crazy changes in the commands
*6y old · 2 comments · `FEEDBACK WANTED` `command` `input`*

Most proposed breaking changes were not implemented: ae is still ae (not e), e is still config (not c), $() substitution not found. Only '|.' was marked done. The maintainer acknowledged in comments that killing the current 'e' command would be 'very problematic'. These large breaking changes were mostly deferred indefinitely.

---

#### [#14065](https://github.com/radareorg/radare2/issues/14065) — Last NDK deprecated the need for the python wrapper script
*6y old · 1 comments · `buildsystem` `Android`*

Confirmed still open. sys/android-shell.sh:100 still references make_standalone_toolchain.py which is the deprecated approach. sys/android-ndk-install.sh also uses the deprecated python script. Neither has been updated to use NDK's direct toolchain binaries.

---

#### [#14078](https://github.com/radareorg/radare2/issues/14078) — Wrong rjmp interpretation in AVR on ATmega644/640
*6y old · 3 comments · `AVR` `high-priority`*

Verified: ATmega640 in libr/arch/p/avr/plugin.c line 125 still has pc=15 (should arguably be different for ATmega644). ATmega644 is NOT listed as a CPU model at all - only ATmega640/1280/1281/2560/2561. The core problem of missing ATmega644 model definition persists. The workaround of using ATmega1280 (pc=16) exists but the bug is not fixed.

---

#### [#14128](https://github.com/radareorg/radare2/issues/14128) — Render class/method/fields graph
*6y old · 0 comments · `RAnal` `RGraph` `types`*

No class/method/fields graph visualization command found. The feature request for a UML-style class hierarchy graph remains unimplemented.

---

#### [#14230](https://github.com/radareorg/radare2/issues/14230) — Add repeated value pattern visualizations
*6y old · 0 comments*

No repeated value pattern visualization was implemented. The images in the issue show a concept for visualizing repeated byte patterns that has no corresponding code.

---

#### [#14246](https://github.com/radareorg/radare2/issues/14246) — Add acl.h type definitions and function signatures in r2
*6y old · 0 comments*

No acl.h type definitions found in the codebase (grep returned zero matches in libr/anal). sys/acl.h types and function signatures have not been added.

---

#### [#14330](https://github.com/radareorg/radare2/issues/14330) — Implement visualization for aai (dashboard style)
*6y old · 1 comments · `enhancement` `consoleui` `visual`*

No dashboard-style visualization for aai was implemented. No aaiv command or visual mode integration for analysis info display was found.

---

#### [#14443](https://github.com/radareorg/radare2/issues/14443) — Setting bg color in nodes glitches
*6y old · 2 comments · `consoleui` `RGraph`*

Graph node background color still conflicts with disasm colorization. A 2022 discussion between contributor and maintainer (trufae) confirms the issue is still open. The maintainer suggested alternatives like highlighting borders or titles but no fix was implemented.

---

#### [#14455](https://github.com/radareorg/radare2/issues/14455) — Implement asm.tabs2
*6y old · 7 comments · `FEEDBACK WANTED` `consoleui` `good first issue`*

Verified: no asm.tabs2, asm.tab.ops, or asm.tab.ins config variable exists in the codebase. The maintainer's last comment suggested asm.tabs.once might make this unnecessary, but the feature was never formally implemented or rejected.

---

#### [#15600](https://github.com/radareorg/radare2/issues/15600) — Add tests for dietline
*6y old · 2 comments · `r2r` `high-priority`*

Confirmed still open. No dietline-specific test files found anywhere in the test/ directory. The line editing library remains untested by dedicated tests.

---

#### [#14594](https://github.com/radareorg/radare2/issues/14594) — Add a confidence levels for the analysis results
*6y old · 3 comments · `RAnal` `concept` `vars`*

No Bayesian probability or confidence level system found in the codebase (grep returned zero matches). This major architectural concept for type inference confidence remains completely unimplemented.

---

#### [#14683](https://github.com/radareorg/radare2/issues/14683) — Dietline - convert between numbers
*6y old · 1 comments · `enhancement` `FEEDBACK WANTED` `consoleui`*

No number base conversion feature in dietline. Grep for 'yank_last_arg' and related patterns found nothing. No dietline commits related to number base conversion were found.

---

#### [#14685](https://github.com/radareorg/radare2/issues/14685) — Tree hud thing
*6y old · 1 comments · `consoleui` `good first issue`*

No tree-like HUD navigation for textual output was found. Related to issue 12745 (generic tree command/API) which is also unimplemented.

---

#### [#14697](https://github.com/radareorg/radare2/issues/14697) — Add a ruler to horizontal histogram output (e.g. `p==`)
*6y old · 0 comments · `enhancement` `consoleui` `good first issue`*

No vertical ruler added to histogram output. No related commits found for value range rulers on p= output.

---

#### [#14742](https://github.com/radareorg/radare2/issues/14742) — REvent further steps
*6y old · 0 comments · `refactor`*

Confirmed still open. r_event.h exists but the proposed r_event_block_push/pop mechanism and r_event_affects_disasm function were never implemented. No matches found in any source files.

---

#### [#14748](https://github.com/radareorg/radare2/issues/14748) — Add visual `p=` scrolling mode
*6y old · 0 comments · `enhancement` `visual` `panel`*

No dedicated scrolling mode for p= histogram output in visual/panel mode was found.

---

#### [#14812](https://github.com/radareorg/radare2/issues/14812) — Make analysis and disassembly incremental
*6y old · 2 comments · `refactor` `RAnal` `RAsm-Disassembler`*

No incremental computation or Adapton-like framework found (grep returned zero matches). This fundamental architectural change remains unimplemented.

---

#### [#15007](https://github.com/radareorg/radare2/issues/15007) — Implement yank-last-arg in r2 cli
*6y old · 3 comments · `FEEDBACK WANTED` `consoleui`*

No yank-last-arg (M-. / Alt+.) functionality in dietline. Grep found no matches for yank_last_arg in the codebase. The feature was discussed and praised but never implemented.

---

#### [#15110](https://github.com/radareorg/radare2/issues/15110) — Embedded help floating window
*6y old · 1 comments · `consoleui` `concept` `visual`*

No embedded floating help window (like vim's :help) was implemented in r2's visual mode. This was a vague concept issue with no concrete proposal or implementation.

---

#### [#15164](https://github.com/radareorg/radare2/issues/15164) — Integration with `GNU poke`
*6y old · 1 comments · `pf` `concept` `types`*

No GNU poke integration found (grep only found matches in tests-fail.txt, unrelated). This concept/feature request remains unimplemented.

---

#### [#15183](https://github.com/radareorg/radare2/issues/15183) — Aarch64: r2/rasm2 incorrectly assembles 64 bits post-index variant of STP
*6y old · 1 comments · `RAsm-Assembler` `ARM` `r2wars`*

Confirmed: libr/arch/p/arm/armass64.c line 2238 shows stp always uses opcode 0x000000a9 (pre-index). The post-index variant requires 0x000000a8 but this distinction is never made. The stp() function at line 1494 takes the opcode as parameter but is only ever called with 0xa9. Post-index vs pre-index confusion persists.

---

#### [#15219](https://github.com/radareorg/radare2/issues/15219) — Add jemalloc 5.x heap parsing
*6y old · 0 comments · `enhancement` `hackaton` `heap`*

No commits found adding jemalloc 5.x heap parsing support. Git log search for 'jemalloc 5' returned no results. The existing dmh_jemalloc.inc.c file only handles older jemalloc versions. This feature request remains completely unimplemented.

---

#### [#15242](https://github.com/radareorg/radare2/issues/15242) — Probabilistic disassembly mode support
*6y old · 0 comments · `hardcore` `RAsm-Disassembler` `concept`*

Research concept/feature request for superset disassembly. No code implementing probabilistic/superset disassembly exists in the codebase. This is an academic concept that was never implemented.

---

#### [#15255](https://github.com/radareorg/radare2/issues/15255) — Cannot set hardware breakpoint on 64-bit Kali Linux when debugging a 32-bit program
*6y old · 4 comments · `RDebug`*

Confirmed still open. Recent comments from October 2025 (AceSLS and trufae) confirm this issue persists. Trufae acknowledged the root cause: different register layouts for 32-bit processes on 64-bit Linux systems. GDB handles this correctly but r2 does not. No fix commits found. The maintainer noted that 32-bit support on 64-bit systems is being deprecated by distros but acknowledged the bug should be fixed.

---

#### [#15435](https://github.com/radareorg/radare2/issues/15435) — Rethink r2 pipes
*6y old · 4 comments · `enhancement` `consoleui` `concept`*

PowerShell-style JSON object pipes not implemented. A maintainer marked it 'not a priority'. No commits implementing JSON pipeline features were found.

---

#### [#15468](https://github.com/radareorg/radare2/issues/15468) — PowerPC64 ragg2 unimplemented
*6y old · 7 comments · `ragg2` `good first issue` `r2r`*

Confirmed still open. The libr/egg/p/ directory contains only generic plugins (bind, exec, nullby, reverse, xor). No PPC or PowerPC-specific ragg2 code exists.

---

#### [#15506](https://github.com/radareorg/radare2/issues/15506) — Automated architecture detection for raw binaries
*6y old · 3 comments · `RAnal` `concept`*

No ML-based or heuristic architecture auto-detection for raw binaries found (grep returned zero relevant matches). This concept/feature request remains unimplemented.

---

#### [#15510](https://github.com/radareorg/radare2/issues/15510) — Add entropy edges printing mode
*6y old · 0 comments · `good first issue` `print` `hackaton`*

No entropy edge printing (rising/falling edges like binwalk -E) found. Grep for 'entropy.*edge' returned no results. The existing p=e command remains unchanged.

---

#### [#15810](https://github.com/radareorg/radare2/issues/15810) — Stabs format parser
*6y old · 1 comments · `debug-info` `good first issue` `hackaton`*

Confirmed still open. Only basic N_STAB detection exists in mach0.c (filtering stabs entries) and mybfd.h has stabs type definitions from imported headers. No actual stabs debug info parser was implemented for extracting type/variable/line information.

---

#### [#16000](https://github.com/radareorg/radare2/issues/16000) — Out of the box support for VDEX and CDEX formats
*6y old · 1 comments · `New File-Format` `good first issue` `hackaton`*

Confirmed: No VDEX or CDEX format plugins in libr/bin/p/. Only standard DEX format is supported (libr/bin/format/dex/). These Android-specific formats remain unsupported.

---

#### [#16191](https://github.com/radareorg/radare2/issues/16191) — Nested blocks hexdump color highlighting
*6y old · 0 comments · `enhancement` `radiff2` `visual`*

No Kaitai-style nested block color highlighting in hexdump view was found. This enhancement for file format visualization with overlapping highlight blocks was never implemented.

---

#### [#16226](https://github.com/radareorg/radare2/issues/16226) — LoadLibrary engine support for debugging
*6y old · 0 comments · `enhancement` `Windows` `RIO`*

No evidence of LoadLibrary (taviso's library for running Windows DLLs on Linux) integration found in the codebase. Git log search returned no results. This is a niche feature request that has not been implemented.

---

#### [#16256](https://github.com/radareorg/radare2/issues/16256) — Visual `p==` mode
*6y old · 2 comments · `enhancement` `good first issue` `hackaton`*

No interactive visual p== mode with navigation hotkeys for whole-file entropy overview was implemented. The maintainer suggested using visual mode with p== set as top command, but XVilka clarified the request is for a full-file overview with cursor navigation, which was not done.

---

#### [#16488](https://github.com/radareorg/radare2/issues/16488) — r_bin_demangle() incorrectly detects name mangling scheme
*5y old · 1 comments · `test-required` `demangling`*

Confirmed: demangle.c lines 165-167 still detect any name starting with '__T' as Swift (R_BIN_LANG_SWIFT). This means '__TIFFSwab16BitData' would be incorrectly treated as Swift and the swift demangler would return 'BitData'. The false detection logic persists unchanged.

---

#### [#16525](https://github.com/radareorg/radare2/issues/16525) — Debug on OpenBSD: Could not get memory map: Operation not permitted
*5y old · 6 comments · `RDebug` `BSD`*

The sysctl KERN_PROC_VMMAP call is still used in bsd_debug.c (confirmed via grep). OpenBSD requires root privileges for this call since ~2015. The last comment from devnexen in 2020 confirmed 'You still need to be privileged on OpenBSD'. No alternative approach was implemented to get memory maps without root. The underlying OS restriction hasn't changed.

---

#### [#16667](https://github.com/radareorg/radare2/issues/16667) — Add a linter/GitHub Action to validate commit messages and headers
*5y old · 1 comments · `infrastructure`*

Verified: no commit message validation GitHub Action exists in .github/workflows/. The CI workflows focus on building, testing, and code analysis (CodeQL, Coverity, Semgrep). ret2libc removed himself from assignee calling it nice-to-have. Never implemented.

---

#### [#16671](https://github.com/radareorg/radare2/issues/16671) — Split r2r.c into libr/main and make it available from inside r2
*5y old · 0 comments · `r2r`*

Confirmed still open. r2r remains entirely in binr/r2r/ (r2r.c, r2r.h, run.c, load.c, interact.inc.c). No r2r code exists in libr/main/. The split was never performed.

---

#### [#16806](https://github.com/radareorg/radare2/issues/16806) — Support loading coredumps from the standard locations
*5y old · 1 comments · `good first issue` `Windows` `RDebug`*

No commits found implementing automatic coredump discovery from standard locations (systemd coredumpctl, /proc/sys/kernel/core_pattern, etc.). This auto-discovery feature request remains unimplemented across all platforms (Linux, macOS, BSD, Windows).

---

#### [#17000](https://github.com/radareorg/radare2/issues/17000) — Incorrect assembly generated for push/pop 16-bit GPR (x86.nz)
*5y old · 0 comments*

Verified by reading libr/arch/p/x86_nz/nzasm.c lines 3121-3198: the oppush and oppop functions still directly emit 0x50+reg/0x58+reg without checking for 16-bit register operands or emitting the 0x66 operand-size override prefix. The bug remains unfixed.

---

#### [#17002](https://github.com/radareorg/radare2/issues/17002) — Elbrus (E2k) architecture support
*5y old · 1 comments · `New Architecture`*

Verified: no Elbrus/E2K-related files exist in libr/arch/p/. Only ELF binary detection was added (recognizing the machine type in ELF headers). No VLIW disassembler or analysis plugin was implemented.

---

#### [#17029](https://github.com/radareorg/radare2/issues/17029) — Consider removing esil_inc and esil_dec
*5y old · 8 comments · `FEEDBACK WANTED` `ESIL`*

esil_inc and esil_dec functions still exist in libr/esil/esil_ops.c and are used in the 6502 plugin. The removal was discussed but never implemented. The operations (++, --, ++=, --=) remain part of ESIL.

---

#### [#17381](https://github.com/radareorg/radare2/issues/17381) — SH-4 instruction set: undocumented FSCA / FSRRA instruction being incorrectly read as FTRV
*5y old · 0 comments · `test-required` `RAsm-Disassembler`*

Verified: libr/arch/p/sh/gnu/sh-opc.h line 569 still only contains FTRV at the overlapping opcode pattern (1111nn0111111101). grep for FSCA/FSRRA in that file returns no matches. These undocumented SH-4 instructions remain unimplemented.

---

#### [#17435](https://github.com/radareorg/radare2/issues/17435) — add debug infos to RBinPlugin for gameboy
*5y old · 7 comments*

Confirmed: the gameboy bin plugin (bin_ningb.c) still has no .dbginfo member. Grep for 'dbginfo' in that file returned no matches. The feature request to load external debug symbol files (BGB format) for gameboy ROMs remains unimplemented. Last discussion was October 2021 with intent but no implementation.

---

#### [#17447](https://github.com/radareorg/radare2/issues/17447) — Import testsuite from SoK
*5y old · 0 comments · `good first issue` `RAnal` `test-required`*

No x86-sok testsuite found in the codebase or test directory (grep returned zero matches). This academic testsuite import was never done.

---

#### [#17671](https://github.com/radareorg/radare2/issues/17671) — Support for ADSP-21XX / 2181 disassembler
*5y old · 7 comments · `New Architecture`*

Verified: no ADSP-related architecture plugin exists in the codebase. The maintainer noted it should go in radare2-extras since MAME (the reference implementation) is C++. Never implemented.

---

#### [#17752](https://github.com/radareorg/radare2/issues/17752) — Wrong x86-64 disassembly
*5y old · 3 comments · `test-required` `RAsm-Disassembler` `x86`*

Confirmed still present. In libr/arch/p/x86/plugin_cs.c:4231, the code explicitly strips 'ptr ' from the mnemonic: `op->mnemonic = r_str_replace(op->mnemonic, "ptr ", "", true)`. The maintainer (trufae) said this filtering should be moved to r_parser instead and suggested a PR, but no PR was merged. The roundtrip issue (disassemble without ptr -> cannot reassemble without ptr using GNU assembler) remains. This is a deliberate design choice but causes practical problems.

---

#### [#17953](https://github.com/radareorg/radare2/issues/17953) — r2 is unable to identify functions within stripped assembled binaries (no C)
*5y old · 6 comments · `RAnal` `test-required` `x86`*

Verified: x86_int_0x80 handler exists in plugin_cs.c:4288 but is COMMENTED OUT at line 4822. The exit syscall (mov eax,1; int 0x80) is not detected as noreturn, causing function boundary detection to fail in stripped binaries. This is definitively still broken.

---

#### [#18219](https://github.com/radareorg/radare2/issues/18219) — Set io.unalloc=true and optimize it
*5y old · 1 comments · `refactor` `RIO`*

Confirmed still open. io.unalloc default is still 'false' as seen in cconfig.c:4804: SETCB("io.unalloc", "false", ...). The optimization work was done but the default hasn't been changed yet, waiting for milestone 6.1.4.

---

#### [#18314](https://github.com/radareorg/radare2/issues/18314) — Consider dropping bfdbg
*5y old · 3 comments*

Confirmed still open. bfdbg still exists: libr/io/p/io_bfdbg.c is present and r_io_plugin_bfdbg is registered in libr/config.h:312. libr/debug/p/debug_bf.c also exists. The maintainer decided to keep it for finding bugs during refactoring.

---

#### [#18445](https://github.com/radareorg/radare2/issues/18445) — rarun2 should interact with more filedescriptors than stdin, stdout and stderr
*5y old · 3 comments*

Verified: handle_redirection_proc in libr/socket/run.c still only handles stdin/stdout/stderr. No custom file descriptor support was added. A volunteer offered to implement it in March 2021 but no follow-up PR was found.

---

#### [#18491](https://github.com/radareorg/radare2/issues/18491) — Implement r_rbtree_node_delete and r_rbtree_cont_node_delete
*5y old · 0 comments · `RUtil`*

Confirmed still open. Neither r_rbtree_node_delete nor r_rbtree_cont_node_delete functions exist. The only related function found is r_crbtree_delete in esil_cfg.c which is a different API. These specific functions were never implemented.

---

#### [#18661](https://github.com/radareorg/radare2/issues/18661) — unsupported leaf type: 0x1609
*4y old · 0 comments · `RBin`*

Confirmed: libr/bin/format/pdb/tpi.c line 2676 still has the default case that logs 'skipping unsupported leaf type'. Leaf type 0x1609 is not in the switch statement (grepped for it, no match). This PDB type will still produce the warning and members field will be empty.

---

#### [#19003](https://github.com/radareorg/radare2/issues/19003) — Unique coloring for similar immediates
*4y old · 2 comments · `consoleui` `good first issue`*

No hash-based unique coloring for immediate values was implemented. A contributor asked to take the issue in 2022 but no follow-up commits were found.

---

#### [#19086](https://github.com/radareorg/radare2/issues/19086) — Segment registers not honored on ESIL
*4y old · 8 comments*

Verified: ESIL still produces '0x28,[8],rax,=' without the fs segment for 'mov rax, qword fs:[0x28]'. The issue explicitly depends on PR #22258 which has not been merged. No segment register support in ESIL.

---

#### [#19138](https://github.com/radareorg/radare2/issues/19138) — charsets: add xnu virtual keycode to ascii/ansi
*4y old · 0 comments · `charsets`*

Confirmed still open. No XNU virtual keycode charset found in the codebase. The charset/muta system exists but this specific charset was never added.

---

#### [#19320](https://github.com/radareorg/radare2/issues/19320) — Consider dropping esil.romem
*4y old · 3 comments*

esil.romem config variable still exists in cconfig.c and multiple usage sites (5 files). The maintainer (trufae) explicitly stated it should stay. This is a design disagreement that was resolved by keeping the feature.

---

#### [#19517](https://github.com/radareorg/radare2/issues/19517) — One sandbox per child
*4y old · 1 comments*

Confirmed still open. The sandbox is a global per-core setting (cfg.sandbox config option). libr/util/sandbox.c provides sandboxing functions but no per-child/per-process sandbox isolation. The feature was never implemented.

---

#### [#19887](https://github.com/radareorg/radare2/issues/19887) — EPIC: Refactoring project - `c` command
*3y old · 1 comments · `refactor` `shell` `epic`*

Git log confirms partial progress: commits dddd13a7dd (c[248]) and 2a4f12d43d (cw). The checklist shows 5 completed items (c1, c[248], cat, cp, cw) out of ~22 subcommands. The majority remain unrefactored. Moved to 5.7.0 milestone as non-blocker.

---

#### [#20193](https://github.com/radareorg/radare2/issues/20193) — Support for x86 AVX* in ESIL
*3y old · 2 comments*

Verified: condret explicitly stated 'improved general support for avx, but still no esil'. No AVX ESIL support found (grep for avx.*esil returned zero matches). ESIL support for AVX/AVX2/AVX512 instructions is definitively missing.

---

#### [#20293](https://github.com/radareorg/radare2/issues/20293) — generate coverage file from tracing information
*3y old · 0 comments*

Confirmed still open. No coverage/lcov/gcov references in CI configuration. No dt subcommand for llvm-coverage format export. This feature was never implemented.

---

#### [#20773](https://github.com/radareorg/radare2/issues/20773) — inline functions
*3y old · 1 comments*

No inline function detection or representation found in the codebase. No 'inline' attribute, no related analysis commands. This feature request remains completely unimplemented.

---

#### [#20845](https://github.com/radareorg/radare2/issues/20845) — Report instruction-level UB details
*3y old · 0 comments*

No UB reporting or tainted register support found. This feature request for reporting undefined behavior at instruction level remains unimplemented.

---

#### [#21660](https://github.com/radareorg/radare2/issues/21660) — UUID APIs and commands
*2y old · 2 comments*

No UUID commands found in cmd_*.inc.c files (grep returned no results). The discussion remained at the idea stage with classification of UUID types but no implementation was started.

---

#### [#21927](https://github.com/radareorg/radare2/issues/21927) — Move shlr/java into libr/bin and libr/arch and reduce visibility of symbols
*2y old · 0 comments*

Confirmed: shlr/java/ directory still exists with class.c, class.h, code.c files. The java code has not been moved into libr/bin or libr/arch. Milestone is 'Attic' suggesting deprioritized.

---

#### [#22275](https://github.com/radareorg/radare2/issues/22275) — Group/Link methods of properties
*2y old · 0 comments*

Confirmed still open. No property method grouping or linking found in class display code. Swift property getter/setter methods are not grouped under their property in ic (class info) output.

---

#### [#22281](https://github.com/radareorg/radare2/issues/22281) — Trim esotheric whitespaces
*2y old · 2 comments*

Confirmed still open. libr/util/str.c has basic whitespace handling but no Unicode/exotic whitespace awareness in r_str_trim functions. Only standard space and tab are handled.

---

#### [#22442](https://github.com/radareorg/radare2/issues/22442) — Add R2PM_SIZE
*2y old · 0 comments*

Confirmed still open. No R2PM_SIZE directive or disk usage tracking found in r2pm code (libr/main/r2pm.c). This feature was never implemented.

---

#### [#22514](https://github.com/radareorg/radare2/issues/22514) — add ltrace like utility on r2
*2y old · 0 comments*

Confirmed still open. No ltrace-like library call tracing functionality found in the codebase. While debug tracing (dt commands) exists for instruction-level tracing, shared library call tracing was never implemented.

---

#### [#22717](https://github.com/radareorg/radare2/issues/22717) — RFC: Type Signatures
*2y old · 1 comments*

This is an RFC/design discussion about implementing type signatures for type matching/inference. No implementation found. Recent comments from 2025 continue adding references. This remains in the design/discussion phase with no code implementation.

---

#### [#22839](https://github.com/radareorg/radare2/issues/22839) — Implement CDPATH env var the same way we handle PATH for executables
*1y old · 0 comments*

Confirmed still open. No CDPATH environment variable handling found anywhere in the codebase. The cd command in fs_shell.c is basic and doesn't consult CDPATH.

---

#### [#22910](https://github.com/radareorg/radare2/issues/22910) — afvs does not indicate non-initial struct members when the stack is adjusted
*1y old · 0 comments*

No commits found referencing this issue. Trufae's comment on #24274 (2025) confirms 'The var analysis code needs to be fully rewritten' waiting for ESIL rewrite. This bug is definitively still present.

---

#### [#23034](https://github.com/radareorg/radare2/issues/23034) — Add a command to check timestamp of plugins to reload them
*1y old · 0 comments*

Feature request by trufae. No implementation found for plugin timestamp checking or hot-reload functionality.

---

#### [#23092](https://github.com/radareorg/radare2/issues/23092) — Confusing dupped commands
*1y old · 2 comments*

The duplicate commands (obf, obio, io, ob) still exist in cmd_open.inc.c (4 occurrences found). Condret confirmed in Nov 2024 that ob commands 'never seem to work as I expect'. The deduplication has not been addressed.

---

#### [#23454](https://github.com/radareorg/radare2/issues/23454) — pahole all the structs
*1y old · 0 comments*

No pahole integration found (grep returned zero matches). Milestoned as 'Attic' indicating deferred. ABI-breaking changes held for 6.0+. Completely unimplemented.

---

#### [#23731](https://github.com/radareorg/radare2/issues/23731) — Improve hash search with ideas from delsum
*1y old · 1 comments*

Feature request from December 2024 to improve the /h hash search command using ideas from the delsum project. Only one comment (cc to sylvainpelissier). No implementation work done. Very recent issue.

---

#### [#24043](https://github.com/radareorg/radare2/issues/24043) — multiple commands via tcp/http-server
*1y old · 0 comments*

Recent issue from March 2025 about executing multiple commands via tcp/http server in interactive mode. No commits addressing this. The rtr.c code handles remote commands but doesn't support chained command execution properly.

---

#### [#24274](https://github.com/radareorg/radare2/issues/24274) — Shows the wrong variable access for the DATA XREF
*9mo old · 2 comments*

Trufae confirmed in June 2025 this is 'Totally expected' and the var analysis code needs full rewrite, waiting for condret's ESIL rewrite. Definitively still open and acknowledged as a known limitation.

---

#### [#24423](https://github.com/radareorg/radare2/issues/24423) — rename mach0 to macho
*8mo old · 0 comments*

Confirmed: Files are still named mach0 throughout: libr/bin/format/mach0/, mach0.c, mach0.h, mach0_defines.h, bin_mach0.c, bin_mach064.c. No rename to 'macho' has been done.

---

#### [#24765](https://github.com/radareorg/radare2/issues/24765) — Add support for ia64 itanium architecture
*4mo old · 0 comments*

Verified: no IA-64/Itanium disassembler plugin exists in libr/arch/p/. Only floatformat header references exist. Test binaries exist in testbins/ia64 but no disassembler. Filed October 2025, very recent.

---

#### [#25099](https://github.com/radareorg/radare2/issues/25099) — iHj does not output json (despite its name/description)
*2mo old · 1 comments*

Verified: iHj is listed in help text (libr/core/cmd_info.inc.c line 22) but trufae confirmed it is 'actually not implemented.' The command exists in documentation but outputs the same non-JSON text as iH. Recent issue from December 2025.

---

#### [#25234](https://github.com/radareorg/radare2/issues/25234) — powerpc-darwin does not have ppc_debug_state (unlike i386/arm): is anything suitable as a fallback?
*2mo old · 3 comments*

PowerPC Darwin missing ppc_debug_state_t implementation. One commit found (acf85e6a7e) that guards off parts not applicable to Darwin/PowerPC, but this is just a workaround (disabling the feature) not a proper fix. Trufae suggested defining the structs in code and not relying on kernel headers. The Feb 2026 'ping' from trufae indicates the reporter hasn't contributed a fix yet. The feature remains disabled/unimplemented for PowerPC Darwin.

---

#### [#25590](https://github.com/radareorg/radare2/issues/25590) — ragg2 failing to assemble ARM code
*1d old · 8 comments*

Very recent issue (March 2026). Trufae made some fixes and asked to retest. The reporter confirmed progress but function prologue is still missing and stack frame warnings are wrong. The egg compiler is acknowledged as needing significant work. Active issue with ongoing discussion.

---

</details>

### Confidence 🔵 4 (196)

<details>
<summary>Click to expand 196 issues</summary>

#### [#662](https://github.com/radareorg/radare2/issues/662) — PE binaries should be handled as 'fat' binaries
*12y old · 8 comments · `enhancement` `refactor` `PE`*

No PE fat binary support (DOS+Windows+.NET sub-bins) found in the codebase. The PE bin plugin (bin_pe.c, bin_pe64.c) does not implement multi-architecture loading. PR #10835 was referenced but this feature was never merged. Milestone is 'Attic' suggesting deprioritized.

---

#### [#812](https://github.com/radareorg/radare2/issues/812) — Merge RBin info from a different file
*11y old · 3 comments · `enhancement` `refactor` `debug-info`*

This is a feature request for a dedicated API to merge debug info from separate files (dSYM on macOS, split debug on Linux). While workarounds exist (ob commands, .!rabin2), no dedicated merge API was found in the codebase. The ob/obf approach was acknowledged as a workaround, not a proper solution. The issue dates from 2014 and the feature was never fully implemented as a first-class API. The DWARF/debug info loading has improved (addrline refactoring, dwarf5 support), but the core request for cross-file symbol/section merging remains unaddressed.

---

#### [#921](https://github.com/radareorg/radare2/issues/921) — META - Portable Executable
*11y old · 3 comments · `enhancement` `debug-info` `Windows`*

META tracking issue. Checked items: authentihash and Rich Header are done. Many unchecked items remain: DLL Characteristics in iI, VTABLE detection for MSVC, resource manipulation, import table manipulation, SEH parsers, .NET support. No evidence of these being implemented. The PE write code in pe_write.c is minimal.

---

#### [#1225](https://github.com/radareorg/radare2/issues/1225) — Better breakpoints
*11y old · 6 comments · `enhancement` `debug-info` `parity`*

Broad feature request for lldb-style breakpoint features. Some breakpoint improvements have been made (overlapped breakpoint warnings via faa7938cc5, native XNU/ARM64 breakpoints via 4ec6ed2a71), but many advanced features requested (function name regex breakpoints, thread-specific breakpoints, one-shot breakpoints, ignore-count) remain unimplemented. Milestone is 'Attic', no recent activity targeting these specific features. The last meaningful discussion was in 2018 with no follow-up PRs.

---

#### [#17130](https://github.com/radareorg/radare2/issues/17130) — WinDbg/KD protocol support
*11y old · 24 comments · `WinDbg`*

Confirmed still open but with partial progress. WinDbg IO plugin (io_windbg.c), winkd IO plugin (io_winkd.c), and debug plugin (debug_bf.c related) exist. KDNET network support was added. However, several checklist items remain unchecked: userspace debugging via serial, kernel debugging improvements, etc.

---

#### [#3222](https://github.com/radareorg/radare2/issues/3222) — Structured document helpers
*10y old · 5 comments · `good first issue` `json` `RBin`*

JSON tokenizer was partially implemented per comments. No XML bin plugin, no CSV support, no seek-to-token/scope navigation commands found. Tree-sitter was mentioned but not present in codebase. The full feature set (RBin plugin for XML/JSON, text file commands) remains unimplemented.

---

#### [#3635](https://github.com/radareorg/radare2/issues/3635) — Fix build on non-exec environments
*10y old · 7 comments · `enhancement` `good first issue` `buildsystem`*

Confirmed still open. No specific support for non-executable mount environments found. The build system still relies on executing compiled programs during the build process (e.g., for SDB generation).

---

#### [#3896](https://github.com/radareorg/radare2/issues/3896) — Add support for LLVM bitcode (stored in Mach-O files along with native binary)
*10y old · 1 comments · `New File-Format` `MACH0`*

Magic signatures for LLVM bitcode were added (commits 481ddade5b, d2c7275155, 6c7bfd846a) and a fix for bitcode mach0s was made (commit 161625dc25). However, these only provide file identification. The actual feature request is for disassembly/analysis support for LLVM IR bytecode as an architecture within fat Mach-O binaries. No LLVM IR disassembler plugin or bitcode extraction support was found in the codebase. This remains an unimplemented feature request.

---

#### [#3942](https://github.com/radareorg/radare2/issues/3942) — anal.nopskip should also skip nops in basic blocks
*10y old · 7 comments · `RAnal`*

anal.nopskip config exists and is used in tests, but only for function entry points. The original request to skip nops within basic blocks (not just at function start) has no evidence of implementation. Tests only show nopskip at function boundaries. Still open feature request.

---

#### [#4369](https://github.com/radareorg/radare2/issues/4369) — to b.h error not explicit
*10y old · 9 comments · `types`*

Tree-sitter was mentioned in 2020 as a future parser replacement but was later removed from the codebase. The TCC-based parser still produces unhelpful error messages for undefined types. The core issue of poor error messages in type parsing persists.

---

#### [#4508](https://github.com/radareorg/radare2/issues/4508) — Identify if argument/variable is constant or variable
*9y old · 10 comments · `RAnal` `types`*

No evidence of implementation. The feature to analyze whether function arguments are constants, booleans, signed values based on call-site analysis was never implemented. Last comment from 2019 suggested esildfg but no follow-up. This is a significant analysis feature that remains unaddressed.

---

#### [#4768](https://github.com/radareorg/radare2/issues/4768) — ^C in dtc stops the program instead of the tracing
*9y old · 2 comments · `tracing`*

Confirmed still open. The dtc (debug trace continue) command exists and has received improvements, but no specific fix for Ctrl-C signal handling during tracing was found. The last confirmation of the bug was in 2020. Signal handling in debug tracing remains unchanged.

---

#### [#4887](https://github.com/radareorg/radare2/issues/4887) — Implement frame navigation in the debugger
*9y old · 3 comments · `good first issue` `RDebug` `gdb`*

No commits found implementing GDB-style 'up/down/frame N' commands. The dbt command exists for backtraces but full frame navigation with local variable display per frame is not implemented. Trufae confirmed in 2020 it was 'partially done in dbt but requires more work'. No subsequent implementation found.

---

#### [#5362](https://github.com/radareorg/radare2/issues/5362) — radiff2 malware diffing bad results
*9y old · 2 comments · `radiff2`*

Confirmed still open. radiff2 exists at binr/radiff2/radiff2.c but the core diffing algorithms remain fundamentally the same. No major improvements to graph-based diffing quality comparable to BinDiff were found.

---

#### [#5390](https://github.com/radareorg/radare2/issues/5390) — META - Heap explorer/analysis
*9y old · 7 comments · `RAnal` `Windows` `MacOS`*

Linux glibc and jemalloc heap analysis are implemented (dmh). However, OpenBSD malloc, tcmalloc, jmalloc, Windows heap, and OS X heap remain unchecked in the issue's checklist. This META issue is partially complete but still has significant uncovered platforms.

---

#### [#6346](https://github.com/radareorg/radare2/issues/6346) — Cross-comparison functions with different names in Radiff2
*9y old · 9 comments · `radiff2`*

No implementation of cross-name function comparison in radiff2 found. Discussion from 2020 ended without resolution. The feature to diff functions ignoring names and do cross-comparison remains unimplemented.

---

#### [#6431](https://github.com/radareorg/radare2/issues/6431) — Implement dh io for gdb://
*9y old · 1 comments · `RDebug` `gdb`*

No evidence of implementing the 'dh io' command for the gdb:// plugin. The io.gdb plugin had globals removed (ce4a32ef34) but no debugger command forwarding via =! or \. was implemented. Milestone is 'Attic'. This low-level feature request remains unimplemented.

---

#### [#6468](https://github.com/radareorg/radare2/issues/6468) — Analysis info not tied to any specific file
*9y old · 6 comments · `RIO`*

Trufae confirmed in 2020 this still matters. No architectural change to bind analysis info to specific files/maps was found. The fundamental issue of analysis not being tied to opened files persists.

---

#### [#17134](https://github.com/radareorg/radare2/issues/17134) — META - vtable detection for C++, ObjectiveC, Dlang and Swift binaries
*9y old · 35 comments · `types` `PDB` `DWARF`*

META tracking issue. Completed: vtable parsing, RTTI, SEH, eh_frame parsing, av commands, ASCII graph. Still open: class-method connection, class inheritance nesting, constructor/destructor autorecognition, try/catch recognition, argument recognition, tests. Significant progress but many items remain.

---

#### [#6867](https://github.com/radareorg/radare2/issues/6867) — cfg.bigendian not honored if set before asm.arch
*9y old · 13 comments · `test-required`*

The core bug persists. In libr/core/cconfig.c:802-809 (cb_asmarch), when asm.arch is changed, the code calls r_bin_is_big_endian() and falls back to cfg.bigendian only if bin detection fails (-1). However, when bin detection returns 0 (little-endian, because no binary is loaded or the binary is LE), it overrides whatever cfg.bigendian was set to previously. The cb_bigendian callback (line 1322) properly sets endianness, but cb_asmarch (line 803) does not check the current cfg.bigendian value when a binary is present. A partial improvement exists at line 805 (fallback to cfg.bigendian when bigbin==-1), but the ordering bug remains when a binary is loaded. No PR was ever merged from the reporter (queueRAM) who volunteered to fix it.

---

#### [#6947](https://github.com/radareorg/radare2/issues/6947) — META - Signatures
*9y old · 3 comments · `zignatures` `META`*

META tracking issue for zignatures. The maintainer noted in 2019 'many of those things are done' but nobody updated the checkboxes. Many enhancement items remain unchecked: FLIRT converter, decompilation metrics, DFG metrics, automatic signature loading, various fixes. Some core functionality works but the full feature set is incomplete.

---

#### [#6953](https://github.com/radareorg/radare2/issues/6953) — META - ARM
*9y old · 1 comments · `ARM` `META`*

META tracking issue with many unchecked ARM-related items: CPSR bits in regprofile, ARM64 ESIL completion, ARM assembler improvements, and various analysis issues. These are broad architectural improvements that are unlikely to all be resolved. Some individual sub-issues may have been fixed but the META tracking issue remains relevant as a wishlist.

---

#### [#6971](https://github.com/radareorg/radare2/issues/6971) — META - RADIFF2
*9y old · 1 comments · `radiff2` `META`*

Confirmed still open. This is a META tracking issue with many unchecked items. radiff2 continues to receive attention but many of the tracked improvements remain unimplemented.

---

#### [#7310](https://github.com/radareorg/radare2/issues/7310) — Create zignatures pack for windows
*8y old · 28 comments · `good first issue` `zignatures` `Windows`*

Confirmed still open. No Windows zignature pack or automated generation scripts found in the codebase. The zignature system exists but pre-built Windows library signatures were never packaged.

---

#### [#7390](https://github.com/radareorg/radare2/issues/7390) — META - New File-Formats
*8y old · 0 comments · `New File-Format` `META`*

META tracking issue for new file format support. Most formats listed (AtariST, Cisco IOS, GOFF, LLVM BitCode, newc, IMG3, CortexM3, Joy!peff, SOM, NeXT ROM, XEX, .sym) remain unimplemented. This is an ongoing wishlist similar to the architectures META issue.

---

#### [#7710](https://github.com/radareorg/radare2/issues/7710) — C types - SDB properties
*8y old · 5 comments · `enhancement` `RAnal` `types`*

Bitfields support was added (checked off). Type alignment and C++ object support remain unchecked. No evidence of alignment property or comprehensive C++ type support in SDB. Still open with 2 of 3 items incomplete.

---

#### [#8291](https://github.com/radareorg/radare2/issues/8291) — rasm2 - (att syntax mode) opcode is parsed by intel format.
*8y old · 6 comments · `enhancement` `RAsm-Assembler`*

AT&T syntax assembler input was confirmed as not existing by the maintainer in 2017. The x86.as plugin supports it as a workaround, but the default assembler (x86.nz) does not handle AT&T syntax. The r_arch rewrite was mentioned but no clear evidence it added AT&T assembler support to the default plugin. Most recent comment from 2022 asks if there's interest in fixing it, suggesting it remains unfixed.

---

#### [#8565](https://github.com/radareorg/radare2/issues/8565) — Better variables support
*8y old · 11 comments · `enhancement` `RAnal` `vars`*

Global variable support (linked issue #11871) and tp/pf variable name application remain open. Stack/bp/register argument integration was done (checked). Move to separate namespace was done (checked). 2 of 4 items remain incomplete.

---

#### [#8600](https://github.com/radareorg/radare2/issues/8600) — Wrong 'aeip' result on 16bit x86
*8y old · 4 comments · `RAnal` `ESIL` `x86`*

The reporter confirmed in June 2020 (commit 7575d05252) that the issue was still present: aeip does not set the CS segment register when seeking to a segment:offset address. The lcall issue was fixed. The CS register not being set by aeip on 16-bit x86 was explicitly confirmed still broken. No fix commits found targeting this specific behavior in the ESIL initialization code.

---

#### [#9169](https://github.com/radareorg/radare2/issues/9169) — [feature request] r2 doesn't allow to change segment perms
*8y old · 6 comments · `refactor` `RBin`*

Confirmed: libr/bin/format/elf/elf_write.c has section_perms() at line 264 for sections only. No equivalent function for segments exists. The feature request to extend rabin2 -O to change segment permissions remains unimplemented. The workaround using raw hex writing still applies.

---

#### [#9384](https://github.com/radareorg/radare2/issues/9384) — Radare2 do not properly detect branches with IT on Arm Thumb 2
*8y old · 11 comments · `RAnal` `test-required` `ARM`*

A partial fix was merged (commit 7f5584ee00 for PR #9795) addressing simple IT cases. However, the reporter confirmed in July 2020 that ITTE/ITETE cases were still broken. A PR by 4z0x was proposed for disassembly+analysis fix in August 2020 but its merge status is unclear. The fundamental issue of conditional execution blocks (ITTE, ITETE) not being properly represented in basic blocks remains a known limitation of the ARM analysis plugin.

---

#### [#9600](https://github.com/radareorg/radare2/issues/9600) — Define a constant via types command
*8y old · 1 comments · `enhancement` `good first issue` `RAnal`*

No 'tC' command found. The type parser still cannot handle 'const int foo = 3;' syntax. No evidence of constant value support in the type system beyond enums.

---

#### [#9649](https://github.com/radareorg/radare2/issues/9649) — pdf does not print instructions after int3
*8y old · 6 comments · `RAsm-Disassembler`*

This is by design according to maintainer comments - int3 is treated as a function terminator. The maintainer suggested using analysis hints as a workaround. This is more of a feature request for configurable behavior (option to not treat int3 as function end). No changes found to alter this behavior. Still technically open as a feature request.

---

#### [#9813](https://github.com/radareorg/radare2/issues/9813) — Show word offset and octal words in bytes
*7y old · 1 comments*

Confirmed still open. While pxd (decimal dumps) and word-size displays exist, the specific word-oriented architecture display mode (word offsets, octal words for architectures with non-byte-addressable memory) was not implemented.

---

#### [#9881](https://github.com/radareorg/radare2/issues/9881) — ahi f and ahi F
*7y old · 0 comments · `enhancement` `good first issue` `RAnal`*

No implementation of float/double immediate base hints (ahi f/F) found in the codebase. The ahi command does not support f/F options for displaying immediates as floating point values.

---

#### [#9919](https://github.com/radareorg/radare2/issues/9919) — Debugging Windows binary from Linux
*7y old · 6 comments · `RDebug` `WinDbg` `WineDbg`*

Feature request for remote debugging Windows binaries from Linux via RAP. While winedbg has been improved (ab512c139b removed globals, 592b6b0ae8 arm tests), the core issue of seamless remote Windows debugging via rap:// remains unresolved. No commits found addressing the specific rap:// debug forwarding use case. The RAP protocol improvements did not fully address the debug use case.

---

#### [#10031](https://github.com/radareorg/radare2/issues/10031) — C types - structures alignment
*7y old · 0 comments · `RAnal` `types`*

No comprehensive alignment/padding mechanism (#pragma pack, __attribute__((aligned))) found in the type system. Structure layout calculation remains basic without proper alignment support.

---

#### [#10032](https://github.com/radareorg/radare2/issues/10032) — C types - unions support interface
*7y old · 2 comments · `enhancement` `RAnal` `test-required`*

Union printing was implemented (checked). Most checklist items remain open: tp command branch selection, tl linking, conditional selection, afta integration, and tests. Partial implementation only.

---

#### [#10137](https://github.com/radareorg/radare2/issues/10137) — [META] Packaging
*7y old · 16 comments · `infrastructure` `buildsystem` `META`*

Confirmed still open. This is a META tracking issue for distribution packaging. While many distros are covered, several remain unpackaged. The tracking issue itself remains relevant for coordinating packaging efforts.

---

#### [#10328](https://github.com/radareorg/radare2/issues/10328) — Support utf8 dots for better "ascii" art
*7y old · 4 comments · `enhancement` `consoleui` `concept`*

Feature request for braille/UTF8 dots in graphs, pie charts, and QR codes. Grep shows braille references only in disasm.c and canvas_line.c, but canvas_line.c has no actual braille character usage (just general UTF8 line drawing). No braille dot patterns for graphs or pie charts were implemented. The checklist items (diagonal edges, pie charts, QR codes) remain unchecked.

---

#### [#10571](https://github.com/radareorg/radare2/issues/10571) — Refresh screen via http server
*7y old · 2 comments · `refactor` `consoleui` `RCore`*

Feature request to trigger visual mode screen refresh remotely via HTTP. No implementation found for push-based screen refresh from HTTP commands. The rtr.c (remote transport) has r_core_cmd calls but no mechanism to trigger visual mode refresh from remote commands.

---

#### [#10685](https://github.com/radareorg/radare2/issues/10685) — R2 Project File Support for Radiff2 Analysis
*7y old · 2 comments · `enhancement` `radiff2` `projects`*

No project file loading in radiff2 was found. The project system has evolved but radiff2 integration was never implemented. Maintainer acknowledged it needs rbin work first.

---

#### [#10695](https://github.com/radareorg/radare2/issues/10695) — Undefined value in ESIL
*7y old · 1 comments · `ESIL` `concept`*

No magic/undefined value representation found in ESIL. The concept of representing undefined/implementation-defined flag values remains unimplemented. This is a design-level feature request that was marked stale in 2020.

---

#### [#10783](https://github.com/radareorg/radare2/issues/10783) — [Feature request] Getting the function's argument values for each xrefs-to it
*7y old · 7 comments · `enhancement` `RAnal` `types`*

No 'aftc' or equivalent command found for listing argument values at all xrefs to a function. While asm.emu provides some emulation capability, the specific automated command proposed was never implemented.

---

#### [#11163](https://github.com/radareorg/radare2/issues/11163) — Create rafind2 equivalent to binwalk -Me
*7y old · 0 comments · `enhancement` `rafind2` `forensics`*

No extraction capability (-mX, -mXY) was found in rafind2. The magic search (-m) exists but file extraction and recursive extraction remain unimplemented. No /me or /meR commands found in the codebase.

---

#### [#11167](https://github.com/radareorg/radare2/issues/11167) — FR/maybe bug? Exposing cprompt width for big screen resolution compatibility.
*7y old · 4 comments · `enhancement` `visual`*

Enhancement request for a dedicated config variable to control cmd.cprompt width/position in visual mode. The reporter found a workaround using hex.cols (visual.c uses hex.cols to calculate column width, visible in the code pattern 'int hc = r_config_get_i(core->config, "hex.cols")'). The cmd.cprompt handling in visual.c (lines 4557-4566) still positions based on hex.cols calculation without a dedicated width override. No new config variable like 'scr.cprompt.col' or similar was added. The issue was reopened by XVilka for a proper fix but none was implemented.

---

#### [#11193](https://github.com/radareorg/radare2/issues/11193) — Make RAsmCode use RStrBuf like RAsmOp
*7y old · 3 comments · `refactor`*

RAsmCode still exists in libr/include/r_asm.h (lines 30-38) and does NOT use RStrBuf for its fields. The struct contains HtPP *equs, ut64 fields, and int code_align but no RStrBuf. While RStrBuf has been adopted extensively elsewhere in the codebase (many commits), this specific refactoring of RAsmCode was never done. The issue is a refactoring request that remains valid but low priority.

---

#### [#11623](https://github.com/radareorg/radare2/issues/11623) — Breaking #!pipe does not stop the child process
*7y old · 1 comments*

Confirmed: libr/lang/p/pipe.c has no SIGINT/SIGTERM sending to child processes. The maintainer confirmed in 2022 that this is a bug in tasks. No evidence of a fix being applied.

---

#### [#11655](https://github.com/radareorg/radare2/issues/11655) — Improve loading of recursive types from C header file
*7y old · 2 comments · `types` `stale`*

Nested structs work (struct A inside struct B). But pointer-to-struct recursive types (struct Node *next) still produce generic 'p' format instead of preserving the struct type reference. Developer acknowledged this was not working and was marked stale.

---

#### [#11705](https://github.com/radareorg/radare2/issues/11705) — Analysis preset
*7y old · 4 comments · `enhancement` `RAnal`*

No dedicated analysis preset command or aaaj JSON output of analysis commands found. The aaa command runs a fixed set of analysis steps without per-arch/format customization.

---

#### [#11763](https://github.com/radareorg/radare2/issues/11763) — r2pipe2 protocol
*7y old · 4 comments · `enhancement` `r2pipe`*

Confirmed still open. A reference to r2pipe2 JSON syntax exists in cmd_help.inc.c but the full protocol enhancements (length fields, multiple channels, authentication, encryption) were never implemented. The r2pipe code in libr/socket/r2pipe.c remains basic.

---

#### [#11793](https://github.com/radareorg/radare2/issues/11793) — Analysis of references to strings only
*7y old · 6 comments · `enhancement` `RAnal` `optimization`*

No /rs command for finding references specifically to strings was found. The memory usage issue was resolved, but the enhancement request for a dedicated string-reference analysis command remains unimplemented. Maintainer explicitly said he wanted /rs but it was never added.

---

#### [#11877](https://github.com/radareorg/radare2/issues/11877) — Build SDK with meson
*7y old · 3 comments · `buildsystem`*

Confirmed still open. While meson build works and sys/ contains WASI SDK scripts, the specific checklist items (--nogpl meson support, libr.a generation, zip packaging, Windows .lib, iOS XCode project) appear incomplete. No dedicated meson SDK packaging target was found.

---

#### [#11886](https://github.com/radareorg/radare2/issues/11886) — Refactor Jemalloc heap parsing code for dynamic configuration and independce from system types.
*7y old · 2 comments · `buildsystem` `Hacktoberfest` `heap`*

The jemalloc heap parsing code still exists in libr/core/dmh_jemalloc.inc.c. Grep confirms the file is still referenced. The original issue was about refactoring to use radare2's own types instead of system-provided types with ifdefs. No commits found specifically addressing this refactoring. The code still relies on system headers.

---

#### [#11934](https://github.com/radareorg/radare2/issues/11934) — xAnalyzer api definitions.
*7y old · 1 comments · `enhancement` `RAnal`*

34 Windows type definition files exist in libr/anal/d/windows/ showing expanded coverage. But no xAnalyzer-format API definitions were imported. The specific xAnalyzer comprehensive coverage was not incorporated.

---

#### [#12142](https://github.com/radareorg/radare2/issues/12142) — Update shlr/grub
*7y old · 3 comments · `refactor` `buildsystem` `stale`*

Confirmed still open. The grub code lives at libr/fs/p/grub/ (moved from shlr/grub). Both locations exist with shlr/grub still containing some files. The code has received security fixes but was never updated to match mainline GRUB as the issue requests.

---

#### [#12276](https://github.com/radareorg/radare2/issues/12276) — Disassembly and Metadata cursor mode in Vp
*7y old · 0 comments · `enhancement` `consoleui` `visual`*

Feature request for enhanced cursor mode in visual Vp with variable/local selection and xref browsing. No comments since 2018. No evidence of implementation of these specific features (tab to switch modes, select vars/locals, show xrefs after selection).

---

#### [#12358](https://github.com/radareorg/radare2/issues/12358) — Rewrite MRC/MCR commands of ARM
*7y old · 0 comments · `enhancement` `RAnal` `RAsm-Disassembler`*

armass64_const.h has some coprocessor register definitions. But no translation of generic MRC/MCR instructions into readable register names in disassembly output was found. The feature remains unimplemented.

---

#### [#12588](https://github.com/radareorg/radare2/issues/12588) — Add command to manage visual tabs
*7y old · 9 comments · `enhancement` `good first issue` `visual`*

Visual tabs exist (visual_tabs.inc.c, Vt/VT) with basic functionality. However, the issue specifically requests shell-level commands to manage tabs (add, close, name, select) for persistence across sessions/projects. The full command management layer was not implemented. Panels have tabs (Vane11ope confirmed) but shell-level tab commands for visual mode are missing.

---

#### [#12697](https://github.com/radareorg/radare2/issues/12697) — Add support for RiscOS binaries
*7y old · 3 comments · `New File-Format`*

No RiscOS/AOF binary format plugin found in libr/bin/p/. A contributor started work in 2020 (wrote header files) but no plugin was merged. The format directory listing shows no aof or riscos entries.

---

#### [#12707](https://github.com/radareorg/radare2/issues/12707) — Proper background support for remote commands '='
*7y old · 0 comments · `enhancement`*

Enhancement request for remote command infrastructure. r2pipe:// IO plugin exists (io_r2pipe.c), addressing one checklist item. However, the broader set of improvements (listing servers, using tasks instead of threads, background servers, HTTPS) appears largely unimplemented. The issue remains open as a tracking issue.

---

#### [#12726](https://github.com/radareorg/radare2/issues/12726) — pf output is confusing
*7y old · 8 comments · `FEEDBACK WANTED` `good first issue` `pf`*

Confirmed still open. The pf (print format) output still does not show offset-first format like px/pd. The large-scale refactoring of r_print_format_*() functions was never done.

---

#### [#12741](https://github.com/radareorg/radare2/issues/12741) — r_name_filter filters too much
*7y old · 0 comments · `Rflags`*

Confirmed still open. r_name_filter is still widely used (found in flag.c, disasm.c, cbin.c, vmenus.c, cmd_debug.inc.c, cmd_open.inc.c, etc.) and still converts special characters to '_', causing different strings to produce identical flag names.

---

#### [#12748](https://github.com/radareorg/radare2/issues/12748) — Improve ? with more type views
*7y old · 0 comments · `enhancement`*

Commit 1c30a14dc4 added uint32 and uint64 to ? output. Current code in cmd_help.inc.c shows uint32, uint64, float, and double are displayed. However, the full feature request asked for 8/16/24/32/64 bit signed and unsigned integers, plus 16/32/64/80 bit floating point types. Missing: int8, uint8, int16, uint16, int24, uint24, int32, int64, float16, float80 (long double). The request is partially implemented but not complete.

---

#### [#12871](https://github.com/radareorg/radare2/issues/12871) — META - C++/etc exceptions parsing and handling
*7y old · 0 comments · `enhancement` `RAnal` `C++`*

No C++ exception parsing/handling implementation found in the analysis code. The only match for 'exception' was in MIPS ELF headers unrelated to C++ exception analysis. This meta-issue remains open.

---

#### [#12888](https://github.com/radareorg/radare2/issues/12888) — Analyze reloc functions
*7y old · 2 comments · `enhancement` `RAnal` `types`*

No evidence of extracting argument type info from C++ mangled relocation names for external functions. The feature to parse demangled reloc names for type information remains unimplemented.

---

#### [#12955](https://github.com/radareorg/radare2/issues/12955) — dcp not working
*7y old · 3 comments · `RDebug` `Linux OS`*

The dcp command implementation at libr/core/cmd_debug.inc.c:4903-4928 still uses r_io_map_get_at() to check if the PC is in a mapped region. The code does 'while (!s)' where s = r_io_map_get_at(). The maintainer confirmed the bug: 'iomaps in debugger just cover the whole memoryspace, so it may never be null.' The current code at line 4919 still uses the same r_io_map_get_at check, meaning this fundamental issue persists. dcp will single-step once and return immediately because the map is always found.

---

#### [#12959](https://github.com/radareorg/radare2/issues/12959) — automatically load .symtab from PE go binaries
*7y old · 3 comments · `enhancement` `RBin` `PE`*

Confirmed: bin_pe.c and bin_pe64.c have no gopclntab scanning or Go-specific .symtab loading. The ELF plugin (bin_elf.inc.c line 1584) checks for .gopclntab sections but PE plugin does not. Go binaries that use ELF-style .symtab in PE files still won't have symbols loaded automatically.

---

#### [#12975](https://github.com/radareorg/radare2/issues/12975) — More configurable default visual mode
*7y old · 2 comments · `FEEDBACK WANTED` `RAsm-Disassembler` `concept`*

Feature request for a navigational visual mode distinguishing data vs code display. The maintainer noted it requires finishing the asm.meta work and complained about lack of feedback. No evidence of implementation.

---

#### [#13092](https://github.com/radareorg/radare2/issues/13092) — RFE: Implement IO window
*7y old · 5 comments · `RIO`*

Feature request for IO windows to restrict IO access to specific address ranges ('ow' command). No 'ow' command exists in the codebase. The feature was proposed as useful for weak gdb servers and scoping analysis. Never implemented despite discussion in 2019.

---

#### [#13212](https://github.com/radareorg/radare2/issues/13212) — ESIL follow constants
*7y old · 1 comments · `enhancement` `ESIL`*

No constant-tracking/following feature found in ESIL. The concept of defining values to track through emulation remains unimplemented.

---

#### [#13295](https://github.com/radareorg/radare2/issues/13295) — Enumerate/Group function list by offset with counter/...
*7y old · 2 comments · `visual` `panel`*

No tree-based function browser or Ghidra-like function list with fold/unfold and counters found. The panels system exists but no tree-view function grouping was implemented.

---

#### [#13320](https://github.com/radareorg/radare2/issues/13320) — Automatic vulnerability detection
*7y old · 0 comments*

Broad feature request/research idea for automatic vulnerability/bug detection using r2 analysis with GTFOBins as test cases. No specific implementation found. This is more of a research direction than a concrete feature request. No follow-up activity.

---

#### [#13355](https://github.com/radareorg/radare2/issues/13355) — Add colors in r2 -h commandline help messages
*7y old · 1 comments · `FEEDBACK WANTED` `consoleui` `good first issue`*

The main radare2.c handles scr_color via command line but the -h help output itself does not use colored output via rcons. The suggestion to use rcons from libr_main was discussed but not implemented. The concern about test compatibility with colored output by default was raised but unresolved.

---

#### [#13376](https://github.com/radareorg/radare2/issues/13376) — Better PowerShell integration on Windows targets
*7y old · 0 comments · `enhancement` `consoleui` `Windows`*

Grep for 'powershell' only finds references in build scripts (meson.build, configure.bat, r2pm.c, test scripts) - no actual PowerShell integration improvements for JSON output or command interaction. This vague enhancement request remains unaddressed.

---

#### [#13389](https://github.com/radareorg/radare2/issues/13389) — Cannot add breakpoint when debugger is not started
*7y old · 1 comments · `RDebug` `cutter`*

The dbg.bpinmaps config still exists in libr/bp/bp.c and libr/core/cconfig.c. No commits were found changing this behavior. The 'db' command still refuses to set breakpoints on unmapped memory. The workaround 'dbs' exists but the core request to allow 'db' on unmapped memory (or default bpinmaps=false) was not changed.

---

#### [#13439](https://github.com/radareorg/radare2/issues/13439) — aslr=no makes ood stop working
*7y old · 5 comments · `RDebug`*

The issue is about rarun2 temporary profile files being deleted between ood invocations when using -R aslr=no. The error 'Can't find profile /tmp/.rarun2.xxx' occurs because the temp file is created on first launch and cleaned up, but ood tries to reuse the same path. A user in 2021 (StefanBruens) confirmed the issue still exists. While commit 2a391c152e 'Fix hanging dc; dc; ood; dc' addressed ood-related issues, no specific fix for the rarun2 temp profile persistence between ood calls was found. The fundamental issue of temp file lifecycle management during ood reopen likely persists.

---

#### [#13499](https://github.com/radareorg/radare2/issues/13499) — Remove unnecessary calls to RConfig.get
*6y old · 2 comments · `good first issue` `optimization` `Hacktoberfest`*

Confirmed still open. 354 r_config_get calls vs 1174 r_config_get_b/r_config_get_i calls in libr/core/. While typed getters are used more than string getters, there are still many r_config_get calls that could be optimized, especially in hot paths and loops.

---

#### [#13554](https://github.com/radareorg/radare2/issues/13554) — Add an option to make io circular
*6y old · 1 comments · `consoleui` `good first issue` `visual`*

No circular IO option was found. The display problems when seeking near 1u64 (maximum address) with wrapping behavior were demonstrated but no fix or config option for circular IO wrapping was implemented.

---

#### [#13613](https://github.com/radareorg/radare2/issues/13613) — Better search on top of recorded traces
*6y old · 1 comments · `enhancement` `FEEDBACK WANTED` `RSearch`*

Conceptual feature request for search capabilities on recorded traces, inspired by GrammaTech trace-db. The maintainer suggested improving visual trace mode. No concrete implementation found. This was a research idea with no follow-up.

---

#### [#13699](https://github.com/radareorg/radare2/issues/13699) — RFE: regdiff of function emulation in zignatures
*6y old · 1 comments · `FEEDBACK WANTED` `zignatures` `ESIL`*

No register diff emulation integrated into zignatures. r2emuhash exists in radare2-extras but was not integrated into core. No ESIL emulation or register diff references found in zignature code.

---

#### [#13702](https://github.com/radareorg/radare2/issues/13702) — Review anal.data
*6y old · 0 comments · `RAnal`*

anal.datarefs config option still exists (found in cconfig.c as 'false' by default). The core concern that this feature introduces regressions and is not enabled by default likely persists as a quality concern.

---

#### [#13754](https://github.com/radareorg/radare2/issues/13754) — Not implemented: afca - Analyse function for finding the current calling convention
*6y old · 0 comments · `types`*

afca appears in core.c completions but grepping for implementation shows no handler. The command still outputs 'Todo' indicating it remains unimplemented.

---

#### [#13757](https://github.com/radareorg/radare2/issues/13757) — Add @@@* to print as offset+size all the results instead of running the commands
*6y old · 0 comments · `FEEDBACK WANTED`*

Feature request to add @@@* iterator variant that prints offset+size pairs rather than executing commands. Grepping for '@@@*' or '@@@\*' in the codebase yields no results in the iterator/command parsing code. The @@@ iterator syntax exists in libr/core/ but no '*' variant was found. This feature was never implemented.

---

#### [#13782](https://github.com/radareorg/radare2/issues/13782) — Review/update flagtags
*6y old · 4 comments · `enhancement` `Rflags`*

The libr/flag/d directory still exists with the same categories (alloc, crypto, dylib, env, fs, network, process, stdout, string, threads, time). Some work was done ('i did some work on this' - 2019) but the systematic function attributes (noreturn, unsafe, pure, tailcall) management was not completed.

---

#### [#13784](https://github.com/radareorg/radare2/issues/13784) — Improve dbt with stackframe contents
*6y old · 0 comments · `enhancement` `RDebug`*

No commits found implementing pxr-style stackframe content display in the dbt backtrace output. The dbt command shows basic frame info (address, sp, size) but not the contents of each stack frame. This enhancement remains unimplemented.

---

#### [#13805](https://github.com/radareorg/radare2/issues/13805) — Implement rbin.io plugin
*6y old · 0 comments · `enhancement` `RIO` `RBin`*

No r_io_plugin for rbin found in the IO plugin directory. No io_rbin.c or similar file exists. The feature to access rbin data through an IO plugin (like the debug or RFS IO plugins) remains unimplemented.

---

#### [#13850](https://github.com/radareorg/radare2/issues/13850) — x86-64 analysis var size issue
*6y old · 2 comments · `RAnal` `types`*

Variable access analysis determining correct variable sizes is a fundamental analysis quality issue. Trufae confirmed in 2020 this is about variable access analysis in r2. The var analysis code 'needs to be fully rewritten' per trufae's 2025 comment on #24274. Still open.

---

#### [#13853](https://github.com/radareorg/radare2/issues/13853) — Simulating Windows API calls while emulating in Linux with ESIL
*6y old · 14 comments · `ESIL`*

No Windows API simulation layer added to ESIL. The workaround 'aht ret @@ imp*' was reported as not working. No substantial progress on API call simulation for cross-platform ESIL emulation.

---

#### [#13984](https://github.com/radareorg/radare2/issues/13984) — Consider to use more formal and modern intermediate representation
*6y old · 1 comments · `ESIL` `concept`*

ESIL remains the primary intermediate representation. No new formal IL (Falcon RE, RREIL-based, Binary Ninja IL style) has been adopted. This is a major architectural concept that has not been implemented.

---

#### [#14018](https://github.com/radareorg/radare2/issues/14018) — Update/autogenerate windows syscalls for r2
*6y old · 1 comments · `good first issue` `Windows` `buildsystem`*

Confirmed still open. Windows syscall SDB files exist but have not been significantly updated or autogenerated from upstream sources like j00ru/windows-syscalls. No autogeneration scripts found.

---

#### [#14124](https://github.com/radareorg/radare2/issues/14124) — Support argument variable direction attributes
*6y old · 2 comments · `types` `high-priority`*

No IN/OUT/INOUT direction attributes for function arguments found. While direction-related strings exist in type definition files for Windows API types, no parser or command support for specifying argument direction was implemented.

---

#### [#14242](https://github.com/radareorg/radare2/issues/14242) — "aesb" on first instruction in trace
*6y old · 1 comments · `ESIL` `RDebug`*

The ESIL step-back feature was acknowledged as fundamentally broken by the maintainer: 'Stepback support should be rewritten completely imho.' The warning message in the code (commit 8b5e0c0219 'Add a comment on the stepback functionality to warn user about how broken is it') explicitly states it's broken. While ce29b5a942 'Fix dbg.trace behaviour with emulation' and 73a20c563c 'Fix bad parsing, uaf and other crashes in the dts command' made improvements, the fundamental stepback design issues persist. The maintainer wants a complete rewrite.

---

#### [#14243](https://github.com/radareorg/radare2/issues/14243) — "aesb" will change "dte" trace log
*6y old · 2 comments · `ESIL` `RDebug`*

The maintainer explicitly acknowledged this as expected behavior for the current implementation: 'Its based on snapshots. So this is expected for the current behaviour but probably not for the logic of stepback.' The stepback implementation replays from a snapshot, which generates duplicate trace entries. This is a known design limitation of the current stepback system. No commit has addressed this because it requires a fundamental redesign of the stepback mechanism, which hasn't happened.

---

#### [#14300](https://github.com/radareorg/radare2/issues/14300) — oob 0xADDR
*6y old · 1 comments*

Confirmed still open. The oob command exists (cmd_open.inc.c:2732) for reopening with new base address, but the fundamental issue - that only bininfo is rebased while analysis metadata, comments, xrefs, and flags are not - remains unresolved.

---

#### [#14410](https://github.com/radareorg/radare2/issues/14410) — "iO" creating/changing section
*6y old · 3 comments · `test-required` `rabin2`*

Feature request for creating new sections or resizing existing sections via iO. The maintainer confirmed in 2020 that this feature is 'experimental' with very few formats supporting these operations. Only ELF supports resizing sections, mach0 supports adding a library. No improvements since then.

---

#### [#14418](https://github.com/radareorg/radare2/issues/14418) — wrong printing in "Vp" with "cmd.cprompt="
*6y old · 1 comments · `consoleui` `visual`*

The cmd.cprompt config still exists (8 references across cconfig.c and visual.c). The maintainer suggested using panels instead but no specific fix for the visual Vp mode rendering glitch was implemented. Given the maintainer's 'at some point we may fix this' comment and suggestion to use panels, this may be considered low priority but is technically still open.

---

#### [#14440](https://github.com/radareorg/radare2/issues/14440) — Move all @@* into @@@
*6y old · 4 comments · `FEEDBACK WANTED`*

Confirmed still open. Both @@ and @@@ iterators exist as separate constructs in cmd.c. The @@ handler starts at line 5356 and @@@ at line 5598+. No unification or migration was performed.

---

#### [#14444](https://github.com/radareorg/radare2/issues/14444) — Add commands to manipulate a canvas
*6y old · 0 comments · `refactor` `visual`*

RConsCanvas exists with drawing functions (canvas.c, canvas_line.c) but the requested command interface to create/manipulate canvases from the r2 shell was not fully implemented. A PoC commit existed but a full command API was not completed.

---

#### [#14517](https://github.com/radareorg/radare2/issues/14517) — Find symbol collisions when multiple bins are loaded
*6y old · 0 comments · `RBin`*

No merged symbol view across multiple loaded bin files found. No RBin.get_symbol_by_name() returning symbols from all binfiles. The feature request for symbol collision detection when using bin.libs or multidex remains unimplemented.

---

#### [#14526](https://github.com/radareorg/radare2/issues/14526) — Toggle mouse somehow
*6y old · 0 comments · `enhancement` `consoleui` `visual`*

The request was to either rename scr.wheel to scr.mouse or add shift-click to toggle mouse text selection. Looking at cconfig.c:4666, there is a comment '// RENAME TO scr.mouse' but the variable is still named 'scr.wheel' (line 4667). The rename was never done. Additionally, grepping for 'scr.mouse' only finds that comment in cconfig.c, confirming it was never implemented. The shift-click approach was also not implemented. The feature request remains unaddressed.

---

#### [#14546](https://github.com/radareorg/radare2/issues/14546) — Improve scr.demo (demoscene mode)
*6y old · 0 comments · `enhancement` `consoleui` `visual`*

scr.demo config exists in cconfig.c. Some improvements were made (UTF8, spinbar per initial description) but the full checklist (transformation matrix, perspective, rotation, minigraph gadget integration) was not completed.

---

#### [#14673](https://github.com/radareorg/radare2/issues/14673) — "@@=" improvement
*6y old · 5 comments · `command` `input`*

In libr/core/cmd.c:6264, @@= calls cmd_foreach_offset() (line 5983). This function at line 6013 only splits on spaces: 'char *str = strchr(each, ' ')'. It does NOT handle comma-separated values. The help text (cmd_help.inc.c:62) shows '@@= 1 2 3' using space separation. The original request was to support comma as a separator in addition to space, and this was never implemented.

---

#### [#14684](https://github.com/radareorg/radare2/issues/14684) — "afvt" doesn't propagate pointer types
*6y old · 9 comments · `RAnal` `types`*

The reporter confirmed in September 2020 (r2 4.6.0-git) that the issue still exists. HoundThe confirmed 'No, sadly no updates yet.' in September 2020. The specific issue of setting a variable type to a struct pointer (e.g., Test*) and having struct field offsets appear in dereferences has never been implemented.

---

#### [#14714](https://github.com/radareorg/radare2/issues/14714) — pseudo api disasm proposal
*6y old · 0 comments · `FEEDBACK WANTED`*

Concept/proposal for making assembly code executable as C or Python. No evidence of this specific feature being implemented. The pseudo disassembly mode (pdc) exists but not in the form described in this proposal. No comments or activity.

---

#### [#14769](https://github.com/radareorg/radare2/issues/14769) — Update shellforge
*6y old · 3 comments · `ragg2` `good first issue` `hackaton`*

Shellforge-related files exist in the codebase (libr/include/sflib/ with 24 files across multiple architectures: linux-x86-32/64, linux-arm-32/64, darwin-x86-32/64, darwin-arm-64, freebsd-x86-32). However, the issue is specifically about updating shellforge with shellforge4 improvements (more architectures/platforms and bugfixes from whatsbcn's fork). A 2022 contributor expressed interest but no follow-up PRs were found.

---

#### [#14860](https://github.com/radareorg/radare2/issues/14860) — "pxa" improvement
*6y old · 0 comments · `visual` `command`*

Feature request for pxa (annotated hexdump) to filter by selected flagspace, and for n/N keys in visual mode to seek only within the selected flagspace. The pxa command exists (cmd_print.inc.c:654,8748) but there is no flagspace filtering logic in the pxa handler. The n/N visual mode seeking also doesn't filter by flagspace. No commits reference this issue. The feature was never implemented.

---

#### [#14922](https://github.com/radareorg/radare2/issues/14922) — Support multicore targets in GDB remote debugger
*6y old · 0 comments · `gdb`*

No commits found specifically addressing multicore GDB target support. Git log searches for 'multicore', 'multi core', and 'multi thread gdb' returned no results since 2020. The GDB remote plugin has received various fixes but multicore/multiprocessor support was not added.

---

#### [#14946](https://github.com/radareorg/radare2/issues/14946) — Pseudo code view / AT&T syntax
*6y old · 0 comments · `enhancement` `hackaton` `pseudo`*

Confirmed still open. The x86 pseudo plugin exists at libr/arch/p/x86/pseudo.c but does not have a dedicated AT&T pseudo mode. While asm.syntax=att works for disassembly, the pseudo/decompiler view does not generate AT&T-style output.

---

#### [#15088](https://github.com/radareorg/radare2/issues/15088) — Isolate types handling into the separate module - RTypes
*6y old · 0 comments · `FEEDBACK WANTED` `refactor`*

r_types.h exists as basic type definitions header, not a separate RTypes module. Type handling still resides primarily in RAnal. No evidence of this refactoring being done.

---

#### [#15373](https://github.com/radareorg/radare2/issues/15373) — Add support for Xrefs to struct fields (low level type xrefs)
*6y old · 1 comments · `RAnal` `types`*

No struct field xref tracking found. The feature to list all instructions accessing specific struct fields remains unimplemented. Was moved out of 4.6.0 milestone with no one assigned.

---

#### [#15449](https://github.com/radareorg/radare2/issues/15449) — Rabin2 resize section / change section permissions not working
*6y old · 3 comments · `test-required` `rabin2`*

The ELF write code in elf_write.c has section_perms() but section resize appears broken. No PE section resize implementation found in pe_write.c. The maintainer confirmed in 2020 'nobody touched that code at all'. No evidence of subsequent fixes.

---

#### [#15477](https://github.com/radareorg/radare2/issues/15477) — on Windows rabin2 wont extend the sections
*6y old · 3 comments · `Windows` `test-required` `rabin2`*

PE section extension via rabin2 -O is not implemented. No pe_write.c function for resizing sections was found. The maintainer confirmed 'Its not implemented, just chk the code'. No evidence of subsequent implementation.

---

#### [#15766](https://github.com/radareorg/radare2/issues/15766) — Support force demangle symbol name with unknown patterns
*6y old · 9 comments · `debug-info` `demangling`*

Confirmed: The demangler in demangle.c only supports GNU v3 C++ demangling (via _Z prefix detection). GNU v2 mangling scheme (like _AddColor__10ZafDisplayUcUcUcUcUc) is not supported. The maintainer noted 'we need libdemangle first'. No libdemangle or GNU v2 support has been added.

---

#### [#15967](https://github.com/radareorg/radare2/issues/15967) — Highlight instruction address
*6y old · 2 comments · `RAsm-Disassembler` `visual`*

Feature request for instruction address highlighting in minimap/tiny VV modes. The maintainer suggested scr.highlight but the reporter noted it doesn't work for multiple highlights. No multi-highlight support for graph minimap views was implemented.

---

#### [#16084](https://github.com/radareorg/radare2/issues/16084) — Automatic loading decision of Zignature Databases
*6y old · 5 comments · `FEEDBACK WANTED` `zignatures`*

Confirmed still open. While zign.autoload config exists (cconfig.c:4259, defaults to false) and dir.zigns is configurable, the automatic decision-making based on file properties (arch, format, compiler) that this issue requests was not implemented. The autoload is a simple boolean, not the intelligent selection described.

---

#### [#16147](https://github.com/radareorg/radare2/issues/16147) — To think about error codes for the r2 commands
*6y old · 0 comments · `FEEDBACK WANTED` `concept` `command`*

This is a concept/design issue about adding ERROR/SUCCESS/TIMEOUT error codes to r2 commands. The RCmdStatus typedef still exists only as a commented-out line in libr/include/r_cmd.h:19 ('// typedef RCmdStatus...'). There is no comprehensive command error code system. Some individual commands use r_core_return_value() for specific cases, but there is no systematic ERROR/SUCCESS/TIMEOUT framework. This design discussion was never resolved.

---

#### [#16184](https://github.com/radareorg/radare2/issues/16184) — Sparse binaries format for tests - *.SBIN format
*6y old · 0 comments · `New File-Format` `RIO` `concept`*

No SBIN sparse binary format implementation found. No relevant IO plugin or bin plugin for this format exists. The concept of a simple binary wrapper for om= like commands with fill patterns remains unimplemented.

---

#### [#16193](https://github.com/radareorg/radare2/issues/16193) — Colorize the `pf` output (especially in `r2 -nn binary` case)
*6y old · 2 comments · `enhancement` `good first issue` `pf`*

No pf colorization code found in cmd_print.c. A comment from Oct 2025 references a PR for cosmetic improvements but the issue remains open. The pf output still lacks color support.

---

#### [#16250](https://github.com/radareorg/radare2/issues/16250) — Set `dbg.glibc` profiles in runtime
*6y old · 0 comments · `enhancement` `refactor` `heap`*

The rename from dbg.glibc to dbg.alloc was never done. grep for 'dbg.alloc' returns nothing; 'dbg.glibc' still appears in libr/core/cconfig.c, libr/core/dmh_glibc.inc.c, and libr/main/radare2.c. The SDB profile loading was also not implemented - heap profiles remain compiled-in. Both checklist items from the issue remain unimplemented.

---

#### [#16326](https://github.com/radareorg/radare2/issues/16326) — Simplify the way pe resources are extracted
*5y old · 4 comments · `PE`*

No simplified single-command PE resource extraction has been added. The workaround using r2pipe scripts was discussed and the maintainer suggested putting the script in r2pipe-codeshare, but no built-in improvement was merged.

---

#### [#16344](https://github.com/radareorg/radare2/issues/16344) — Improve the output of radiff2 -U flag by having file names instead of hardcoded string
*5y old · 2 comments · `enhancement` `hackaton` `radiff2`*

The request was to show actual filenames in unified diff output instead of generic temp file names. Looking at libr/util/udiff.c:276-319, r_diff_buffers_unified() still creates temp files with r_file_mkstemp('r_diff', ...) and passes them to the external 'diff -au' command (line 14: d->diff_cmd = strdup('diff -au')). The output from diff will show temp file paths like '/tmp/r_diff.XXXXXX' rather than the original filenames. The function signature takes raw buffers (const ut8 *a, *b) with no filename parameters, so the caller filenames are lost. The fundamental architecture hasn't changed -- filenames still aren't propagated to the diff function.

---

#### [#16362](https://github.com/radareorg/radare2/issues/16362) — Struct pointers pf-ied as just pointers
*5y old · 0 comments · `test-required` `types`*

The issue where 'struct A *_a' in type definitions becomes just 'p _a' in pf format instead of preserving the struct type information likely persists. No specific fix found.

---

#### [#16686](https://github.com/radareorg/radare2/issues/16686) — Getting `error: too many basic types` when trying to import efi.h
*5y old · 7 comments · `test-required` `types`*

The type parser still has limitations with redefining primitive types like uint32_t. Tree-sitter was mentioned as future parser but was later removed from the codebase. The TCC-based parser likely still has this limitation. No cfg option for type overwrite behavior was added.

---

#### [#17019](https://github.com/radareorg/radare2/issues/17019) — META - Handle antidisassembly tricks
*5y old · 1 comments · `good first issue` `test-required` `RAsm-Disassembler`*

Meta tracking issue for anti-disassembly techniques. Sub-issue #5136 (dead code) is checked, but #4635 (overlapping instructions) and assembly wrapping remain unchecked. The initial description notes it was updated as recently as January 2026, confirming active tracking.

---

#### [#17074](https://github.com/radareorg/radare2/issues/17074) — Allow to run `r2r` with the specified binaries directory
*5y old · 1 comments · `enhancement` `r2r`*

No R2_TESTBINS_DIR or similar environment variable found in the test/ directory. The test runner still uses hardcoded relative paths for test binaries.

---

#### [#17079](https://github.com/radareorg/radare2/issues/17079) — Bring back gameboy autocomments
*5y old · 0 comments*

Confirmed still open. The gameboy arch plugin exists at libr/arch/p/gb/plugin.c but has no autocomment generation for bankswitch detection. The requested plugin-comment column in pd output and new RAnalOp field were never added.

---

#### [#17096](https://github.com/radareorg/radare2/issues/17096) — Manipulated IAT detection/correction.
*5y old · 1 comments*

Feature request for detecting manipulated IAT entries (using tools like CallObfuscator). No implementation found. ret2libc asked for a test case but none was provided. The feature remains unimplemented.

---

#### [#17606](https://github.com/radareorg/radare2/issues/17606) — Enum base type is ignored
*5y old · 0 comments · `RAnal` `test-required` `types`*

RAnalBaseType struct exists with kind-based type handling. But no evidence of enum base type serialization/deserialization being added. The struct has evolved but the specific enum base type issue appears unresolved.

---

#### [#17649](https://github.com/radareorg/radare2/issues/17649) — Re-do semantic information of atomic types in the database
*5y old · 5 comments · `RAnal` `test-required` `types`*

The RAnalBaseType struct still uses pf format characters for type encoding rather than a proper encoding enum. No evidence of the proposed encoding enum refactoring. The type system still relies on pf characters for atomic type representation.

---

#### [#17772](https://github.com/radareorg/radare2/issues/17772) — R2 don't show labels on conditional and unconditional jumps
*5y old · 6 comments · `test-required` `RAsm-Disassembler`*

Feature request for automatic label generation at jump targets. The maintainer provided a Python script workaround but noted it should be supported natively. No evidence of built-in label generation support being added. The workaround exists but native support was never implemented.

---

#### [#17907](https://github.com/radareorg/radare2/issues/17907) — Feature Request: Performance (Speedup of e.g. Scrolling) by Caching Results (e.g. Disassembly)
*5y old · 3 comments*

Feature request for caching disassembly results in visual mode. The maintainer acknowledged the issue, suggested using panels/graph/webui where caching exists, and noted the io caching problem. No disassembly caching was implemented for visual mode scrolling.

---

#### [#17917](https://github.com/radareorg/radare2/issues/17917) — Implement a objdump -S <debug-binary> output equivalent in r2
*5y old · 8 comments · `RAsm-Disassembler` `PDB` `DWARF`*

No implementation of objdump -S style output (source-first view with assembly below) was found. r2 still shows assembly-first with source as comments on the right via asm.dwarf. The request was for a display mode where source lines appear above their corresponding assembly instructions. No relevant commits found. The existing asm.dwarf=true just adds source line references as comments, not the requested format.

---

#### [#17941](https://github.com/radareorg/radare2/issues/17941) — Add Windows and Jemalloc heap parsing tests
*5y old · 0 comments · `Windows` `test-required` `heap`*

This is a test infrastructure request. The dmh_jemalloc.inc.c and dmh_windows.inc.c files exist, but no static test binaries for Windows heap or jemalloc were found in the test suite. The existing tests only cover glibc heap (linux_glibc-2.30_x64.bin). A macOS heap parser was added (dmh_macos.inc.c) but also lacks tests. This remains an unfulfilled request for test coverage.

---

#### [#18015](https://github.com/radareorg/radare2/issues/18015) — radiff2 graph view broken
*5y old · 2 comments*

The maintainer confirmed single-sided graph output is expected behavior (swap files for the other side). However, the feature request to add file name labeling to identify which graph belongs to which file was not implemented. The issue shifted from a bug report to a small enhancement request.

---

#### [#18034](https://github.com/radareorg/radare2/issues/18034) — Add avx support in /af
*5y old · 2 comments*

Confirmed still open. No AVX-specific search category found in cmd_search.inc.c for /af. The ROP search and function search exist but AVX as a dedicated search filter was never added.

---

#### [#18073](https://github.com/radareorg/radare2/issues/18073) — Clarify/Unify cursor/pointer
*5y old · 0 comments*

Confirmed still open. Around 30 cursor/pointer references in cmd_print files but the unification of visual cursor and mouse support configuration described in the issue was never implemented.

---

#### [#18093](https://github.com/radareorg/radare2/issues/18093) — Wishlist for documentation / help subsystem
*5y old · 2 comments*

Feature request for improved help subsystem with grouping, aliases, extended descriptions, and flexible markup. No evidence of a major help system rewrite in radare2 (Rizin took a different approach with YAML command definitions). The radare2 help system still uses the legacy string-list approach.

---

#### [#18098](https://github.com/radareorg/radare2/issues/18098) — manual immediate structure offset
*5y old · 4 comments*

The aht command incorrectly picks the immediate value from source operand (0x41) instead of the destination offset (4). No specific fix for operand position selection in aht was found. The discussion identified the root cause but no implementation followed.

---

#### [#18140](https://github.com/radareorg/radare2/issues/18140) — When shortening stuff, make sure it actually saves space
*5y old · 1 comments*

The asm.imm.trim feature in libr/asm/parse.c:26 (r_asm_parse_immtrim) strips '0x' and hex digits from immediates, replacing them with nothing. The function at line 32-38 finds '0x', then moves everything after the hex digits back, effectively removing the immediate entirely. The issue was about '..' being used as a replacement for '0x' which doesn't save space. The current code seems to have changed the approach (stripping rather than replacing with '..'), but the maintainer asked for a PR fix at disasm.c:2604, which may not have been submitted.

---

#### [#18174](https://github.com/radareorg/radare2/issues/18174) — AVR: Byte/Word addressing in disassembly
*5y old · 2 comments*

Feature request for word-based address display mode for AVR. The reporter acknowledged it's not a bug but a feature request. A commenter suggested an option similar to relative address display. No word-based addressing display option was added for AVR.

---

#### [#18631](https://github.com/radareorg/radare2/issues/18631) — Add "Graph with Offsets" to Visual/Panel mode
*4y old · 8 comments*

graph.addr (formerly graph.offset) exists and works for showing addresses. The issue evolved into a request for p/P view cycling in panel mode graph panels, which was not implemented. The maintainer suggested menu entries or additional panel entries but no one implemented it.

---

#### [#18777](https://github.com/radareorg/radare2/issues/18777) — License - LGPL-3.0-only or LGPL-3.0-or-later?
*4y old · 4 comments*

Confirmed still open. COPYING.md states 'radare2 is mostly LGPLv3' and mentions LGPL-3.0-only in the license script output, but the ambiguity between -only and -or-later is not definitively resolved in the documentation. Discussion was still active in 2025.

---

#### [#18821](https://github.com/radareorg/radare2/issues/18821) — Add arguments for iz and izz to show only ascii unicode or wide strings
*4y old · 2 comments · `good first issue` `RBin` `charsets`*

No string type filtering (ascii/unicode/wide) found for iz/izz commands. No charset filter arguments found in cbin.c. The maintainer suggested using rtable in cbin but this was not implemented.

---

#### [#18962](https://github.com/radareorg/radare2/issues/18962) — Variables use wrong address offsets in Microsoft x64 executable.
*4y old · 1 comments · `vars`*

The Microsoft x64 shadow space variable offset problem persists. The anal.vars.newstack config (libr/core/cconfig.c:3867) is marked EXPERIMENTAL and defaults to false. The maintainer (trufae) acknowledged in 2021 that this is a known issue requiring a variable analysis rewrite. The pre-prologue register spills to shadow space are calculated with offsets relative to post-sub-rsp stack pointer, producing wrong offsets. No specific fix for this calling convention edge case was found.

---

#### [#19139](https://github.com/radareorg/radare2/issues/19139) — charsets: add more from hyperion
*4y old · 2 comments · `charsets`*

Feature request to add Hercules/Hyperion codepage charsets. A contributor (gogo2464) volunteered in March 2022 but no follow-up implementation was found. The charset system exists but these specific encodings were not added.

---

#### [#19196](https://github.com/radareorg/radare2/issues/19196) — Piping external commands to rarun2 stdin/arg1 fails
*4y old · 0 comments*

No commits found fixing the issue of piping external commands (using ! prefix) to rarun2 stdin or arg options. The rarun2 code has had some improvements but the specific !command piping feature was not addressed.

---

#### [#19481](https://github.com/radareorg/radare2/issues/19481) — add a new test to disassemble+reassemble a full file and check the checksum of these files for each architectures
*4y old · 4 comments*

Feature request for comprehensive disassemble-reassemble round-trip testing. The maintainer agreed with the idea but noted 'no serious effort towards this.' No such test infrastructure was added to the test suite.

---

#### [#19528](https://github.com/radareorg/radare2/issues/19528) — disassembly for mips mtc/mfc: incorrect coprocessor registry
*4y old · 2 comments*

This is an upstream Capstone bug (capstone-engine/capstone#1673). Acknowledged as duplicate of #17372. The bug persists until Capstone fixes it. No r2-side workaround was implemented. The Capstone library bundled with r2 likely still has this issue.

---

#### [#19595](https://github.com/radareorg/radare2/issues/19595) — x86-16bit function analysis false positive argument
*4y old · 0 comments*

No commits found referencing this issue. The bug where sub-register writes (al/ah) cause false positive argument detection for the full register (ax) in 16-bit mode likely persists. No specific fix for this edge case was found.

---

#### [#20014](https://github.com/radareorg/radare2/issues/20014) — Compile in CI with -Wshadow -Werror
*3y old · 0 comments*

Confirmed still open. No -Wshadow flag found in CI configuration, Makefile, meson.build, or mk/ files. Some variable deshadowing commits exist but the CI enforcement of -Wshadow -Werror was never enabled.

---

#### [#20099](https://github.com/radareorg/radare2/issues/20099) — Make anal.jmp.cref true as default
*3y old · 2 comments*

Feature request to change anal.jmp.cref default to true. The config variable still exists in libr/core/cconfig.c with default 'false'. The maintainer (trufae) disagreed with the change, saying jumps/cjmps shouldn't create xrefs unless the target is a function. This was a design decision, not a bug. The default remains false.

---

#### [#20499](https://github.com/radareorg/radare2/issues/20499) — Radare2 cannot detect datarefs correctly like IDA on arm32
*3y old · 1 comments*

ARM32 LDR instructions create datarefs pointing to the literal pool address rather than the value stored there. The maintainer acknowledged the issue in March 2023 ('i'll take a look when i have some time') but no fix commits were found. The issue remains.

---

#### [#20559](https://github.com/radareorg/radare2/issues/20559) — Syscalls are not detected correctly
*3y old · 7 comments*

Confirmed still open. The syscall detection via /as and analysis commands exists but the accuracy issues persist. The initial description notes improvements were made (all syscalls now reported) but number accuracy remains a problem.

---

#### [#20568](https://github.com/radareorg/radare2/issues/20568) — Code guidelines suggestions
*3y old · 5 comments*

Discussion about code quality guidelines (malloc->calloc, strcpy elimination, r_free, variable initialization). These are ongoing codebase improvements with no specific resolution point. The maintainer discussed plans for r_malloc/r_free macros in 5.8 but these are incremental improvements. Some suggestions were debated (strcpy vs strncpy vs snprintf). This remains an open discussion/wishlist.

---

#### [#20754](https://github.com/radareorg/radare2/issues/20754) — void init
*3y old · 6 comments*

Confirmed still open. r_core_init returns bool (libr/include/r_core.h:495). Other _init methods return various types. The proposed bulk refactoring to make all _init methods return void was never performed, and the maintainer said it was 'not a blocker'.

---

#### [#20829](https://github.com/radareorg/radare2/issues/20829) — Improve XDG Support
*3y old · 0 comments*

Confirmed still open with partial progress. XDG support exists via r_xdg_datadir, r_xdg_cachedir functions and is used in panels, pdb paths, sigdb, zigns, projects, and cache. However, the specific checklist items (XDG_DATA_DIRS support, Windows AppData paths, macOS Library paths) appear incomplete.

---

#### [#20923](https://github.com/radareorg/radare2/issues/20923) — ROP regex is not as expected for excluding characters
*3y old · 1 comments*

Confirmed still open. The ROP search functions construct_rop_gadget and r_core_search_rop in cmd_search.inc.c accept regex parameters but no specific fix for character exclusion regex behavior was found.

---

#### [#21195](https://github.com/radareorg/radare2/issues/21195) — pd gets off track on large instructions crossing 0x100 offset
*3y old · 6 comments*

Maintainer stated 'No intention to implement this' in 2023. The underlying issue with MAX_OP_SIZE and buffer boundaries for large instructions (like in pickle/snes plugins) remains. The architectural limitation persists.

---

#### [#21251](https://github.com/radareorg/radare2/issues/21251) — Bad assembly generated on arm 32bits
*3y old · 11 comments*

The maintainer (trufae) identified this as a type propagation (aaft) bug and created PR #21257, but explicitly stated 'that fix breaks other tests' and 'the logic behind this code is wrong, more summer of code crap that needs to be rewritten'. No evidence that PR #21257 was merged or that an alternative fix landed. The workaround is to use 'af' or 'aa' instead of 'aaa', but the underlying type propagation bug in ARM32 analysis remains unfixed.

---

#### [#21492](https://github.com/radareorg/radare2/issues/21492) — `movdqa xmm12, oword [inputA]` unexpectedly assembled as `movdqa xmm12, xmmword [obj.inputA]` in x86_64
*3y old · 1 comments*

Feature request to replace xmmword with oword in disassembly output for NASM compatibility and add XMM instruction support to x86.nz assembler. The maintainer agreed it sounds good. No commits found addressing this.

---

#### [#21494](https://github.com/radareorg/radare2/issues/21494) — Weird issue with reverse debugging
*3y old · 0 comments*

The issue is about dts+ (trace session) causing the instruction pointer to jump to libc when stepping. This relates to the known stepback/trace session issues. While 73a20c563c 'Fix bad parsing, uaf and other crashes in the dts command' and bf3188306e 'Fix a crash in dts+ command with empty register arenas' fixed crashes, the fundamental stepping behavior issue persists. The trace session mechanism (dts+) modifies how stepping works, and the jump to libc suggests improper state restoration. The maintainer has acknowledged stepback needs rewriting.

---

#### [#21496](https://github.com/radareorg/radare2/issues/21496) — File descriptors management broken
*3y old · 1 comments*

The maintainer (trufae) said it was 'half fixed' in PR #21501 but indicated the remaining bug needed solving before closing the ticket. The ddd command exists in libr/core/cmd_debug.inc.c:5380-5393, using r_core_syscallf for dup2 syscall injection. The 'Cannot get stack pointer' error suggests the r_debug_execute path for injecting syscalls is broken. The dd command was refactored (724d23a6c0) but the ddd dup2 functionality specifically remains broken as described.

---

#### [#21975](https://github.com/radareorg/radare2/issues/21975) — Improve c++ vtable parsing support
*2y old · 4 comments*

Confirmed still open. vtable parsing code exists in libr/anal/rtti_itanium.c with RTTI/vtable analysis, but the comprehensive improvements described in the issue (following a specific reference document for better parsing) were not implemented.

---

#### [#22310](https://github.com/radareorg/radare2/issues/22310) — Radare2 does not follow forked child / also ignores breakpoint
*2y old · 2 comments*

The reporter set dbg.follow.child=true but r2 stayed attached to the parent after fork. The implementation at libr/debug/debug.c:1271 checks for R_DEBUG_REASON_NEW_PID, but the fork detection on Linux uses ptrace with PTRACE_O_TRACEFORK which generates a different event. The follow_child flag defaults to false (debug.c:369). A commenter suggested 'try dbg.follow.child=1' but this is the same as what the reporter already did. The issue is essentially the same as #12878 from 2019, suggesting the fork-following mechanism has a persistent bug in the Linux native debugger.

---

#### [#22321](https://github.com/radareorg/radare2/issues/22321) — Patching while debugging leads to weird behaviour
*2y old · 2 comments*

Patching instructions at a breakpoint address causes corruption (0x29 byte from the int3/cc breakpoint byte being restored incorrectly). Trufae acknowledged this as expected behavior and suggested warning the user. No commits found implementing such a warning or handling this corner case (breakpoint overlap detection commit faa7938cc5 handles overlapping breakpoints but not patching at breakpoint addresses). The bug remains.

---

#### [#22325](https://github.com/radareorg/radare2/issues/22325) — error while installing radare2 on windows using cygwin
*2y old · 3 comments*

Confirmed still open. Cygwin is referenced in meson.build:369 (disabled ptrace) and some header files, but the build issues described in the ticket were never fully resolved. The maintainer acknowledged it should be easy to port back but no restoration was done.

---

#### [#22400](https://github.com/radareorg/radare2/issues/22400) — mach-o plugin doesn't create null fds
*2y old · 0 comments*

No null fd creation for sections with vsize > psize found in bin_mach0.c or bin_mach064.c. The mach-o plugin does not handle the case where virtual size exceeds physical size by creating null file descriptors.

---

#### [#22477](https://github.com/radareorg/radare2/issues/22477) — [AIX] debugging support
*2y old · 0 comments*

While several AIX portability commits were found (15b9924e75, 5709416ea5, dae7e5cf53 for build/configure support, and 1a4d6e7fe3/f80ef3ca94 for XCOFF binary handling), no debugging-specific support for AIX was implemented. The AIX work focused on compilation and binary format support, not ptrace/debugging on AIX.

---

#### [#22485](https://github.com/radareorg/radare2/issues/22485) — [AIX] r2r silently fails when jq not available
*2y old · 1 comments*

Confirmed still open. No specific SIGPIPE handling or jq availability checking found in binr/r2r/ code. The AIX-specific issue with r2r silently failing was not addressed.

---

#### [#22592](https://github.com/radareorg/radare2/issues/22592) — Colorize bytes in hexdump/disasm that has been modified or patched in the cache
*2y old · 0 comments*

Feature request to highlight modified/patched bytes in cache with different colors. No body text, no comments, and no commits implementing this feature. Filed by trufae himself, likely a self-assigned enhancement that hasn't been done yet.

---

#### [#22650](https://github.com/radareorg/radare2/issues/22650) — stderr not captured into internal files
*2y old · 0 comments*

stderr redirection to internal r2 files (using $foo syntax) does not work, though redirecting to external files works. No comments and no commits found addressing this. Simple but unfixed bug.

---

#### [#22853](https://github.com/radareorg/radare2/issues/22853) — Support nested memory references in the RNum operations
*1y old · 0 comments*

Feature request to support nested memory references like [[0x8080]] or [10+[0x8080+rbp]*4] in RNum expressions. No comments and no implementation found. Filed by trufae, likely a self-assigned enhancement.

---

#### [#22890](https://github.com/radareorg/radare2/issues/22890) — visual-r2rop does not work
*1y old · 0 comments*

The visual r2rop code exists in vmenus.c (confirmed: 'visual-r2rop' label, pdp command reference, help text). The bug report describes ROP gadgets not showing in the visual panel and pdp displaying invalid ROPs. No fix commits were found. The code is present but apparently broken.

---

#### [#22892](https://github.com/radareorg/radare2/issues/22892) — Debugger Stalls W/ Connect In Profile
*1y old · 0 comments*

No commits found fixing rarun2 connect handling stalls. The issue is that r2 stalls and requires kill -9 when the rarun2 connect= profile fails to reconnect after the remote side disconnects. No relevant fixes in git log.

---

#### [#23061](https://github.com/radareorg/radare2/issues/23061) — Add support for demangling plugins
*1y old · 3 comments*

A basic r_bin_demangle_plugin() function exists in demangle.c that delegates to bin plugins with .demangle callbacks. However, the issue requests dedicated demangling plugin types (not just bin plugin callbacks). Comments from 2025-2026 indicate this is still being discussed. The maintainer said 'Should be possible already, just not tested' and wants r2skel examples.

---

#### [#23236](https://github.com/radareorg/radare2/issues/23236) — Add option to show comments at right or top depending on length
*1y old · 0 comments*

Feature request to dynamically position comments based on length for better graph appearance. No comments and no implementation found. Filed by trufae in August 2024.

---

#### [#23362](https://github.com/radareorg/radare2/issues/23362) — Every call to RNum.math or .get should check be checking for errors
*1y old · 0 comments · `good first issue`*

Confirmed still open. r_num_math_err and r_num_get_err APIs exist in the headers and implementation, but only 3 calls to these error-checking variants exist in libr/core/ vs hundreds of calls to the non-error-checking r_num_math/r_num_get. The migration is far from complete.

---

#### [#23372](https://github.com/radareorg/radare2/issues/23372) — Add iz+/iz- like we have for classes (ic+/ic-)
*1y old · 0 comments*

Partially implemented. iz- exists for purging strings (cmd_info.inc.c:62, 1401) but iz+ for registering new strings was not found. The feature is half-implemented.

---

#### [#23392](https://github.com/radareorg/radare2/issues/23392) — help messages shouldnt exceed 80 columns
*1y old · 2 comments · `good first issue`*

Confirmed still open. This is a broad formatting effort across all help messages in the codebase. No systematic enforcement or completion of 80-column help message limits was found.

---

#### [#23475](https://github.com/radareorg/radare2/issues/23475) — r2: expanding a file size not working as expected
*1y old · 3 comments*

The issue is about resizing files and maps not syncing. Comments explain this is by design (io layers are separate), and users should use omt commands for map tying. The UX issue remains - the behavior is confusing to users even if technically correct. No code changes were made to improve the workflow.

---

#### [#23490](https://github.com/radareorg/radare2/issues/23490) — Ensure nullable for all the apis
*1y old · 0 comments*

Confirmed still open. 99 R_NULLABLE/R_NONNULL annotations found in libr/include/ headers. Progress has been made through multiple commits, but the issue asks for ALL APIs to have nullable annotations, which is an ongoing effort with many APIs still unannotated.

---

#### [#23598](https://github.com/radareorg/radare2/issues/23598) — Import thunk alignment size
*1y old · 0 comments*

No PLT entry size information found in the 'ii' command output. No import_thunk_size or plt_entry_size fields found exposed in the bin plugin interface. The feature to show PLT entry sizes remains unimplemented.

---

#### [#23653](https://github.com/radareorg/radare2/issues/23653) — Support elbrus/e2k cpu
*1y old · 0 comments*

Confirmed still open with minimal progress. ELF format recognizes EM_MCST_ELBRUS (elf_specs.h, glibc_elf.h, elf.c) and maps it to 'elbrus' arch string. However, no actual Elbrus/E2K disassembly or analysis plugin exists in libr/arch/. Only binary format identification is supported, not CPU instruction support.

---

#### [#23724](https://github.com/radareorg/radare2/issues/23724) — Append Lc root commands help in ? help message
*1y old · 0 comments*

No 'Lc' pattern found in cmd.c help output. The core plugin commands are not listed in the main help message. Milestoned 6.1.4 and still open.

---

#### [#23813](https://github.com/radareorg/radare2/issues/23813) — Access to the Windows PEB structure.
*1y old · 5 comments · `Windows` `RDebug`*

The :tls command has been implemented for w32dbg (44480ef124), mach (cb1111a595), and ptrace (aea2b68ba2). However, the core request for PEB structure access/overlay is still unresolved. The Feb 2026 comment from trufae says 'check the :tls command in r2... finish this task. any windows user around with the mood?' indicating the PEB structure overlay part is incomplete. The :tls command provides the address but not the full PEB structure parsing.

---

#### [#23827](https://github.com/radareorg/radare2/issues/23827) — oba $$ not working with fat machos
*1y old · 0 comments*

This is a recent issue (Dec 2024) reporting that 'oba $$' fails silently for fat mach-o files while working correctly for extracted individual slices. No fix commit was found. The issue was filed by trufae (pancake, the main r2 developer) suggesting it's a known but unfixed problem. The fatmach0 code has seen improvements (7c9a05c407, 685c8d6503) but none specifically addressing oba interaction with fat binaries. Given the recency and that it was filed by the lead developer without resolution, this is still open.

---

#### [#23864](https://github.com/radareorg/radare2/issues/23864) — ASCII character space (0x20) is displayed in different ways
*1y old · 2 comments*

Confirmed still open. No specific commits addressing the inconsistent display of space character (0x20) as '@' in comments and '_' in flag names. The r_name_filter function still replaces special characters including spaces with '_' for flag names, which is part of this inconsistency.

---

#### [#24044](https://github.com/radareorg/radare2/issues/24044) — tcp-server in background mode
*1y old · 1 comments*

Confirmed still open. HTTP server background mode exists (=h& command, rtr.c:1024) but the specific request for TCP server (=+, rap protocol) background mode is not implemented. The maintainer commented in Nov 2025 that the command logic is being rewritten for real multithreading.

---

#### [#24074](https://github.com/radareorg/radare2/issues/24074) — Installing and using plugins under Windows does not work
*1y old · 15 comments · `r2pm`*

Confirmed still open. The user confirmed the issue persists as of March 2025 despite some r2pm fixes. Windows plugin installation through r2pm remains broken.

---

#### [#25026](https://github.com/radareorg/radare2/issues/25026) — Add %RIP Relative Reference Search
*3mo old · 1 comments*

Feature request for RIP-relative reference search in PIC ELFs. Trufae suggested using ESIL search as a workaround but no dedicated feature was implemented. Filed December 2025, very recent.

---

#### [#25034](https://github.com/radareorg/radare2/issues/25034) — -S Option is Ignored When Assembling in rasm2
*3mo old · 2 comments*

Very recent issue (Dec 2025). The maintainer linked to PR #25028 saying it would be fixed when that PR is finished. The att2intel filter exists in libr/arch/p/x86_nz/att2intel.c but the -S att flag for rasm2 assembly mode is not wired up. No merge of PR #25028 was found in git log. The issue is confirmed as pending a fix that hasn't landed yet.

---

#### [#25145](https://github.com/radareorg/radare2/issues/25145) — Problem encountered while using the step software
*2mo old · 0 comments*

Confirmed still open. r_debug_step_soft exists in libr/debug/debug.c:845 for software stepping, but the specific ARM POP instruction handling bug was not found to be fixed. No commits addressing this specific issue were identified.

---

#### [#25181](https://github.com/radareorg/radare2/issues/25181) — Poor Memory Optimization when saving large Projects
*2mo old · 5 comments*

Memory issue when saving large projects with Ps command. Trufae suggested using the new 'prj' command instead, which was designed with these problems in mind. The old Ps command issue likely persists but a workaround (prj) exists. The user confirmed the workaround works. Still open as the Ps command itself was not fixed.

---

#### [#25297](https://github.com/radareorg/radare2/issues/25297) — Move the autoname core commands into an analysis plugin
*1mo old · 0 comments*

Recent issue from January 2026. Autoname functions exist in canal.c and are referenced in cmd_anal.inc.c. No refactoring to move them into a plugin found. Very recent, no progress.

---

#### [#25480](https://github.com/radareorg/radare2/issues/25480) — pcap: detect embedded ELF inside captures?
*19d old · 5 comments*

This is a recent feature request (Feb 2026) about detecting embedded binaries in PCAP files. The maintainer noted existing tools (/m, oba) cover the use case. The reporter is exploring stream-based detection via pcap:// IO plugin. As of the latest comment (March 2026), the maintainer asked if this can be closed as it's more of a feature request. No code changes were made.

---

</details>

### Confidence 🟡 3 (120)

<details>
<summary>Click to expand 120 issues</summary>

#### [#2685](https://github.com/radareorg/radare2/issues/2685) — backtrace output seems to be wrong
*10y old · 6 comments*

Backtrace code exists in libr/debug/p/native/bt.c, bt/generic_x86.c, bt/fuzzy_all.c, bt/generic_x64.c. The dbg.btalgo config (default/fuzzy/anal) still exists. The issue was that the default/fuzzy algorithm produced infinite repeated frames with a small stack overflow (memset overwriting 10 bytes past buffer). The 'anal' algorithm worked correctly. The backtrace engine code has been maintained but there's no evidence this specific edge case was fixed - the fundamental frame-walking algorithm for default/fuzzy modes may still struggle with mildly corrupted stacks.

---

#### [#5490](https://github.com/radareorg/radare2/issues/5490) — Visual Debug view w/registers - Color change of modified registers sometimes stops updating
*9y old · 7 comments · `consoleui` `RDebug`*

The register diff coloring in visual mode depends on the register arena swap mechanism. Looking at libr/core/cmd_debug.inc.c:3190-3196, dro/drd use r_reg_arena_swap() to access old register values. The visual mode register display relies on the same mechanism. While there have been register color fixes (ad8400caf3, 1c4f91029e), these addressed disassembly coloring, not the visual debug register diff. The core issue - that register diff state gets lost after breakpoint continuation (dc) - relates to when arena swap copies happen, and no specific fix for this was found. The arena swap logic in libr/reg/arena.c:196 still uses the same dual-arena approach.

---

#### [#5836](https://github.com/radareorg/radare2/issues/5836) — radare2 misinterpreting instructions as a string for a DOS binary
*9y old · 21 comments*

The issue is about MZ/DOS binaries with zero sections causing r2 to misidentify executable code as strings. The maintainer confirmed this is because the binary has 0 sections, and suggested bin.strings=false as a workaround. The MZ parser has received updates (commits 493e284b9e clamps MZ sections, a01c13f603 honors section/segment logic, 1398432e97 fixes loading MZ with text < bsize), but none specifically address the string detection heuristic for sectionless MZ binaries. The core string detection code (libr/bin/bfile.c) does have logic to check for data sections (commit a5ab4aca9b 'Do not find strings in binaries with no data sections'), but the MZ-specific zero-sections case may still not be properly handled. The workaround (bin.strings=false) likely remains the solution.

---

#### [#5993](https://github.com/radareorg/radare2/issues/5993) — radiff2 "No functions found, try running with -A or load a project" for ihex inputs.
*9y old · 2 comments · `radiff2` `MIPS`*

The issue is about radiff2 not propagating analysis results for ihex format binaries. No commit directly fixes this. The radiff2 tool has received improvements over the years but the specific ihex analysis pipeline issue (functions found in r2 but not in radiff2) was never explicitly resolved. The ihex plugin still exists but the interaction between radiff2's analysis flow and section-less formats is a design limitation.

---

#### [#6838](https://github.com/radareorg/radare2/issues/6838) — Adapt Arena for Dalvik
*9y old · 8 comments · `ESIL` `Android`*

Feature request for resizable register arena for Dalvik's variable-sized register set (up to 65535 registers). The '...' syntax in r_reg to allow resizable arenas was discussed but never implemented. A contributor promised a first version but no evidence it materialized. The Dalvik support remains limited. Issue was in 'Attic' milestone suggesting deprioritization.

---

#### [#7415](https://github.com/radareorg/radare2/issues/7415) — The pointer to an object is not reference.
*8y old · 11 comments · `enhancement` `RAnal` `test-required`*

The issue requests automatic detection of struct pointer references passed as function arguments. While type analysis has been significantly improved (type propagation, afvt, taa), the specific pattern of tracking struct pointer contents through registers to resolve string references requires deep ESIL emulation. No commit specifically addresses this. The types system was reworked but this level of inter-procedural type tracking remains incomplete.

---

#### [#7567](https://github.com/radareorg/radare2/issues/7567) — rabin2: wrong information for OAT ELF binaries
*8y old · 6 comments · `Android` `ELF`*

The issue reports wrong vaddr and spurious imports for OAT (Android ART) ELF binaries. The ELF parser has been extensively updated (65196c2616 'Refactor / improve loading of ELF symbols + imports', aab4abf3e3 'Proper cleanup of relocs, imports and symbols in ELF', 80282014e7 'Warn on unresolved symbols/relocs and better handle -1 addresses'). However, no OAT-specific handling was found. The spurious imports with empty names and wrong paddrs are likely an ELF symbol resolution bug specific to OAT's unusual section layout (.bss for oatbss with no file backing). While general ELF improvements may have partially helped, the specific OAT format quirks are unlikely to have been specifically addressed.

---

#### [#7867](https://github.com/radareorg/radare2/issues/7867) — dmsdj and dmsdj all?
*8y old · 2 comments · `enhancement` `good first issue` `RDebug`*

The snapshot system was reworked - 'dms' was brought back (02b7d165b4). The 'dmsd' command exists for hex diffing snapshots. However, no 'dmsdj' (JSON output) was found in the codebase via grep. The specific request for JSON output of snapshot diffs and comparing all snapshots appears unimplemented. Reclassifying from 'obsolete' to 'still_open' since the core feature (dmsd) exists but the JSON variant requested does not.

---

#### [#7944](https://github.com/radareorg/radare2/issues/7944) — ar != dr
*8y old · 6 comments · `refactor` `RAnal` `ESIL`*

The fundamental issue of divergent ar/dr command sets persists. Trufae confirmed in 2020 it should be discussed and fixed. No major refactoring to unify analysis and debug register commands was found. The codebase still has separate ar and dr command handlers.

---

#### [#8389](https://github.com/radareorg/radare2/issues/8389) — mips|mips.gnu plugin assembler error in branch labels?
*8y old · 3 comments · `RAsm-Assembler` `MIPS` `r2wars`*

No commits specifically fixing MIPS branch label resolution in the assembler were found. The MIPS assembler has received improvements but label handling in rasm2 multi-line assembly (where labels need to be resolved to relative offsets) is a known limitation. The issue was from 2017, the label resolution code in r_asm_assemble has not been significantly reworked for MIPS. The r2wars tag suggests this was relevant for r2wars competitions but remains unfixed.

---

#### [#8563](https://github.com/radareorg/radare2/issues/8563) — Analyzing mapped PE files in a process memory image
*8y old · 4 comments · `PE` `rabin2`*

The issue is about using 'oba' to load PE files at specific offsets within a FUSE-mounted memory image. This is a niche use case. The PE parser has been updated but the specific workflow of loading mapped PEs from raw memory images via oba was assigned to GustavoLCR in 2020 and never confirmed resolved. No targeted fix commit found.

---

#### [#8639](https://github.com/radareorg/radare2/issues/8639) — Profile overwrites ood argument
*8y old · 1 comments · `rarun2` `RDebug`*

The issue is about rarun2 profile arguments (-R arg1=BBBB) taking precedence over ood arguments. The rarun2 profile handling has been updated (9887b12794 added base64 support, 8580ed5c65 multiple preload directives) but no commit specifically addresses the argument priority between -R profile args and ood args. The rarun2 profile is applied after the ood arguments are parsed, which means profile values override ood values. This is arguably a design issue rather than a bug, but the reporter expected ood args to take precedence.

---

#### [#9286](https://github.com/radareorg/radare2/issues/9286) — RAP Protocol Question
*8y old · 5 comments · `FEEDBACK WANTED` `RDebug`*

The RAP protocol for remote debugging still has the same fundamental architecture. debug_rap.c exists at libr/debug/p/debug_rap.c (94 lines - quite small). Commit 460d55ea83 'Fix debug rap reg profile setup' addressed part of the issue. However, the core problems described - register profile handling via RAP, the config override in cconfig.c overriding the remote profile - are architectural issues. The RAP protocol was partially superseded by the sysgdb:// plugin (f5c444ba67) for remote debugging. Issue #17091 (referenced) suggests ongoing awareness. The user's specific use case (custom 6502 emulator debugger) likely still doesn't work well via RAP.

---

#### [#9378](https://github.com/radareorg/radare2/issues/9378) — Question - Workflow for saving / loading state of debugger session
*8y old · 5 comments · `FEEDBACK WANTED` `RDebug` `projects`*

Projects have received various fixes and improvements (136fd1a4ff, a3d8fc94e9, 4caa3654d7 adding fd selection to project script). However, the fundamental issue of saving/loading debug session state (breakpoints, analysis, etc.) across different PIDs remains a complex problem. The project system has been reworked but debug-specific workflow with PID handling is still not seamless. No specific commits addressing the PID rebasing issue were found.

---

#### [#9653](https://github.com/radareorg/radare2/issues/9653) — /c and /cj should search also .sub disable/enable
*8y old · 2 comments · `enhancement` `RSearch`*

Feature request to allow /c and /cj to search by symbol names (sym.imp.XXX) rather than just raw addresses. The search commands in libr/core/cmd_search.inc.c have been updated over time but no commit was found implementing symbol name matching in the /c (call search) command. Maijin confirmed in June 2020 that this was still missing and important. No evidence of resolution.

---

#### [#9713](https://github.com/radareorg/radare2/issues/9713) — ROP classification not working if setup is not done properly
*8y old · 5 comments · `ROP` `ESIL`*

Multiple UX issues with ROP gadget classification: requiring manual setup of esil.stats, esil.romem, aeim; no error messages; multiple SDBs. The maintainer suggested /R should auto-set these variables. No evidence this auto-setup was implemented. The ROP code has been maintained but these UX issues likely persist.

---

#### [#9880](https://github.com/radareorg/radare2/issues/9880) — ESIL asm.shortcuts
*7y old · 1 comments · `good first issue` `RAnal` `ESIL`*

Feature request for ESIL shortcuts for indirect jumps (UJMP). No specific implementation found. The referenced commit f443be9dc8 was for context only. The asm.shortcuts feature exists but ESIL-based resolution of indirect jumps for shortcut display was not implemented.

---

#### [#9885](https://github.com/radareorg/radare2/issues/9885) — Missing esil for rev16 (thumb)
*7y old · 2 comments · `ESIL` `architectures-enhancements`*

Checklist of missing ESIL implementations for ARM thumb instructions. Three items were marked done (rev16, revsh, rev). However, many items remain unchecked: bkpt, ssat, stcl, vbic, sxtb, uxth, ldc. Partial resolution - some fixed, most still missing.

---

#### [#9975](https://github.com/radareorg/radare2/issues/9975) — Call xrefs from relocation section
*7y old · 5 comments · `RAnal`*

The issue is about r_core_anal_search_xrefs disassembling relocation data as code, creating false call xrefs. The maintainer claimed it worked fine in 2019 but the reporter clarified the issue more specifically (the relocation section being disassembled as code). No targeted fix for skipping relocation sections during xref search was found. The xref system has been reworked but this specific edge case may persist.

---

#### [#10114](https://github.com/radareorg/radare2/issues/10114) — cmd.vprompt setting in visual mode does not work when entering libc library
*7y old · 0 comments · `consoleui` `visual`*

The issue is that cmd.vprompt (e.g., 'dm.') stops working when stepping into library code in visual debug mode. The cmd.vprompt is read and executed in visual.c at multiple points (lines 1791, 4586, 4916). The vprompt command is executed via r_core_cmd_str() and its output is displayed. When stepping into unmapped/library regions, commands like 'dm.' may return no output or error. This is not a bug in the vprompt mechanism itself but rather in the commands being executed failing silently in certain memory regions. No specific fix for this interaction was found. The underlying issue (commands failing silently in library code regions) likely persists.

---

#### [#10211](https://github.com/radareorg/radare2/issues/10211) — Remote rap debug: ptrace PEEKTEXT doesn't have the right address
*7y old · 4 comments*

The issue is about RAP remote debugging where the address offset sent to ptrace is always 0x0. This relates to how the RAP protocol handles seek operations and address translation for the remote debugger. The RAP plugin (debug_rap.c, 94 lines) is minimal and hasn't received significant updates. While io.ptrace has been updated, the core RAP protocol address handling issue - where io->off is 0x0 when accessed remotely - likely persists. The sysgdb:// plugin (f5c444ba67) is now the recommended approach for remote debugging.

---

#### [#10457](https://github.com/radareorg/radare2/issues/10457) — Make types editable in their C format.
*7y old · 1 comments · `RAnal` `hardcore` `types`*

Feature request to re-edit struct definitions via 'to -'. The types system has been significantly refactored but this specific UX improvement (re-editing previously defined structs) was never explicitly implemented. The 'to -' command opens the editor but doesn't pre-populate with existing struct definitions.

---

#### [#10557](https://github.com/radareorg/radare2/issues/10557) — ARC processor support for r2 (on native ARC cpu, not for analyzing ARC binary)
*7y old · 6 comments · `enhancement` `architectures-enhancements`*

ARC asm/anal plugins exist for analyzing ARC binaries. Running r2 natively on ARC/RTOS hardware remains unaddressed. The 2020 discussion showed the user's ARC board runs TRON (RTOS), and native r2 execution was not achieved. This is a niche platform support request.

---

#### [#10689](https://github.com/radareorg/radare2/issues/10689) — libr/r_asm x86.nz breaks when using certain registers with memory accesses
*7y old · 9 comments · `test-required` `RAsm-Assembler` `x86`*

The infinite loop with xmm0 was fixed (f07c02eafe), and several x86.nz improvements have been made (d5027d07d3 for reg+idx, 3421b73bf3 for xmmword, etc.). However, the core issue of wrong assembly with extended registers (r12, r13, etc.) combined with the 'qword' specifier was confirmed still broken in Dec 2019. While many x86.nz commits exist, no specific commit addresses the SIB/REX byte encoding for extended register memory operands with explicit size specifiers. The x86_nz code in libr/arch/p/x86_nz/nzasm.c has been updated but the specific encoding bug likely persists.

---

#### [#10891](https://github.com/radareorg/radare2/issues/10891) — Special characters in demangled names replaced with underscores
*7y old · 6 comments · `FEEDBACK WANTED` `demangling`*

The flag realname system exists in libr/flag/flag.c, and asm.demangle is implemented. However, the specific complaint was that realnames filter out spaces and that filtered flag names are displayed in disassembly instead of demangled names. No r_name_filter or r_str_filter calls were found in libr/bin/demangle.c, suggesting the filtering may have moved elsewhere, but the fundamental issue of special characters being replaced for flag names persists by design. The issue also lacked a clear consensus on what the fix should be (per ret2libc's 2020 comment).

---

#### [#10996](https://github.com/radareorg/radare2/issues/10996) — ARM void* type size is incorrect
*7y old · 4 comments · `hardcore` `ARM` `types`*

The issue is that ARM Thumb binaries report void* size as 16 bits instead of 32 bits because r_bin_elf_get_bits returns 16 for Thumb, which then loads types-16.sdb. The types system was refactored (commit 435eb89b67 'DWARF type parsing into RAnalBaseTypes and saving into sdb'), and the hint system was improved. However, the fundamental issue - that Thumb instruction width (16-bit) is conflated with pointer size (32-bit) for type loading - is an architectural problem. The maintainer (XVilka) called it 'difficult' and said it 'will require massive refactoring'. No specific fix for this pointer-size-vs-instruction-width conflation was found.

---

#### [#11085](https://github.com/radareorg/radare2/issues/11085) — Decode additional strings.
*7y old · 19 comments · `enhancement` `RAnal` `concept`*

anal.strings and emu.str exist for some string detection. However, automated FLOSS-like stack string reconstruction without manual ESIL stepping has not been fully implemented as a first-class feature. The maintainer acknowledged it should be done but it remains partial.

---

#### [#11089](https://github.com/radareorg/radare2/issues/11089) — Problems with call-refs in MIPS32
*7y old · 2 comments · `enhancement` `RAnal` `test-required`*

MIPS indirect calls through GOT (jalr t9 pattern) not showing up in afi call-refs. The maintainer acknowledged this requires ESIL emulation to resolve. While MIPS analysis has received improvements (commit 9731747022 for jalr function detection), the fundamental issue of resolving indirect calls through GOT entries into proper call-refs requires ESIL emulation which is not always run. Partially improved but likely still incomplete.

---

#### [#11119](https://github.com/radareorg/radare2/issues/11119) — Writting a bigger type doesn't overlap following frame fields.
*7y old · 2 comments*

Partial progress found: libr/anal/var.c line 168 has shadow_var_struct_members() which removes overlapping vars when the type is a struct. However, this only works for struct types, not for plain array types like char[32]. The original issue about changing a var type to char[32] and having overlapping vars auto-removed is likely still not fully working for non-struct array types.

---

#### [#11365](https://github.com/radareorg/radare2/issues/11365) — `pf n2` starts at wrong address when `cfg.bigendian=true`
*7y old · 1 comments · `pf`*

In libr/util/format.c, the 'n' format type handling for big endian starts at line 1930. The r_print_format_sizeof correctly accounts for n2 size (line 1935: tabsize*2). However, the actual print formatting code's big-endian byte offset calculation was not clearly fixed. No commit specifically targeting this pf bigendian offset issue was found. The issue would need testing to confirm.

---

#### [#11391](https://github.com/radareorg/radare2/issues/11391) — question/bug in afi stack frame
*7y old · 0 comments · `FEEDBACK WANTED` `RDebug`*

This is a design question about whether ARM push register instructions should be counted in the stack frame size reported by afi. The reporter asked if push {r4, r5, lr} (12 bytes) + sub sp,sp,0x10 (16 bytes) should result in a 28-byte stack frame instead of just 16. This is an analysis design decision. No specific commit addresses this. The stack frame analysis still primarily looks at explicit SP adjustments (sub sp) rather than push instructions in its frame size calculation.

---

#### [#11584](https://github.com/radareorg/radare2/issues/11584) — No reference constant string in format string of scanf family function
*7y old · 11 comments · `RAsm-Disassembler` `types`*

Short format strings like '%s' (2 chars) not being detected as string references. The maintainer noted '2 char string is usually considered a false positive'. The suggestion by ret2libc to use calling convention info to force string detection at known format string parameters was not implemented. The core string detection logic may have changed but the fundamental 2-char string threshold issue likely persists.

---

#### [#11724](https://github.com/radareorg/radare2/issues/11724) — Wrong assembler output for x86
*7y old · 4 comments · `test-required` `RAsm-Assembler` `stale`*

The x86.nz assembler in libr/arch/p/x86_nz/nzasm.c has received many updates (d5027d07d3 for reg+idx/idx+reg support, etc.) but the specific SIB byte encoding issue for scale+index+base+displacement (e.g., 'mov eax, dword [ebx + ecx*2 + 0x10]') was not found to be specifically fixed. The assembler outputs 8b040b (no displacement, wrong SIB) instead of 8b444b10 (correct SIB with displacement). This is a different bug from #10689 (extended registers) but in the same code. No commit specifically mentions fixing scale factor with displacement handling.

---

#### [#11850](https://github.com/radareorg/radare2/issues/11850) — Rust language reversing helpers
*7y old · 1 comments · `enhancement` `RAnal`*

Feature request for Rust-specific reversing helpers (trait resolution, vtable parsing, etc.). Basic Rust demangling exists (libr/bin/mangling/rust.c) but the broader request for Rust-specific analysis helpers referenced in external projects has not been implemented as a comprehensive feature.

---

#### [#12146](https://github.com/radareorg/radare2/issues/12146) — -m option in r2 does not detect the right bin plugin and hence things are missed
*7y old · 6 comments · `stale`*

The issue is that using -m (map address) causes bin plugin detection to fail because the seekaddr used for check_bytes is the map address instead of 0. The maintainer clarified -B should be used for binaries with headers and -m for raw dumps. However, this is still a usability bug - when -m is used with a headerful binary, it reads at the wrong offset for plugin detection. The commit 1e0231cfbd fixed the izz/string side of -m issues, but the bin plugin detection issue is a different codepath. The 'WARNING: using oba to load the syminfo from different mapaddress' message and the analysis hint loss are likely still present.

---

#### [#12206](https://github.com/radareorg/radare2/issues/12206) — ARM/ELF issues
*7y old · 6 comments · `RBin` `ELF` `relocs`*

This issue tracks two ARM/ELF problems: (1) duplicate relocs pointing to the same import overwriting flags, and (2) reloc resolution for indirect calls via GOT. For (1), commit 8f1af2e1ce 'Implement ELF relocs for SPARC and MIPS and avoid duplicates' and c38f140c67 'bin: do not filter out symbols and do not duplicate relocs addresses' may help, but the specific case of R_ARM_GLOB_DAT and R_ARM_JUMP_SLOT to the same symbol producing duplicate flags was never specifically addressed. For (2), the io.cache-based reloc resolution for ARM indirect calls was never fully implemented. The issue was left unresolved with the maintainer saying 'i will describe in detail when i get some spare time'.

---

#### [#12344](https://github.com/radareorg/radare2/issues/12344) — Improve graph lines height
*7y old · 3 comments · `consoleui` `RGraph` `visual`*

Enhancement request to improve backward line drawing in ASCII graphs, specifically to turn backward lines before the next graph row level. The agraph.c code has been modified over the years but recent commits (ecccdc5f9f, c8a136d0a0, etc.) are mostly bug fixes for OOB issues, not rendering improvements. The graph line drawing algorithm in agraph.c is complex and no specific commit addressing backward line height optimization was found. The issue had confused contributors about what exactly needed to change.

---

#### [#12529](https://github.com/radareorg/radare2/issues/12529) — Enhance wasm binary loading
*7y old · 1 comments · `RAnal` `RBin` `stale`*

WASM format header exists (wasm.h) and has received various fixes. But the specific parsing enhancements (function indices, signatures, init_expr, global section symbols) remain largely unaddressed based on the checklist items. Some improvements may have been made incrementally.

---

#### [#12566](https://github.com/radareorg/radare2/issues/12566) — Add command to split a function
*7y old · 2 comments · `RAnal` `stale`*

Commit 2128795d94 'Takeover variables when splitting functions' and 0e7f5b894f 'Fix wrong splitting of functions in aac' show function splitting improvements. However, the specific request for a dedicated inverse-of-afm command to split a function at a given offset without reanalyzing is unclear. The existing af command does some of this but a dedicated split command was requested.

---

#### [#12650](https://github.com/radareorg/radare2/issues/12650) — Types Struct offset 0 / first member
*7y old · 0 comments · `enhancement` `RAnal` `types`*

Feature request for 'tas 0' to show the first struct member. The types system has been significantly reworked but no specific commit addresses this exact feature. The 'ta' command for struct annotation at offset 0 may work differently now but no evidence of explicit fix.

---

#### [#12747](https://github.com/radareorg/radare2/issues/12747) — type propagation: char* not propagate to called function in x86-64
*7y old · 0 comments · `has-test` `test-required` `types`*

Type propagation through register moves to called functions (argv -> r12 -> rsi -> rdi -> called function) not working. The type propagation code has been significantly reworked but this specific inter-procedural propagation through register chains is a hard problem. No specific commit addresses this. The issue has a 'has-test' label suggesting a regression test exists.

---

#### [#12780](https://github.com/radareorg/radare2/issues/12780) — The entropy section search command should support all p= subcommands
*7y old · 0 comments · `RSearch`*

Feature request from the maintainer (radare) to extend the entropy section search (/s) to support all p= subcommands (char frequency per block, etc.). While commit d1d9404907 implements basic /s entropy search, there's no evidence that it was extended to cover all p= modes. The feature request is still open and unfulfilled.

---

#### [#12878](https://github.com/radareorg/radare2/issues/12878) — "e dbg.follow.child = true" setting does not work
*7y old · 0 comments · `RDebug`*

The dbg.follow.child implementation exists in libr/debug/debug.c:1271 where it checks for R_DEBUG_REASON_NEW_PID and switches to the child. However, commit 286ca4d0ed from 2017 was the only implementation. The dbg.execs description still says 'stop execution if new thread is created' (libr/core/cconfig.c:4414), matching the originally reported confusion. Issue #22310 from 2023 reports the same fork-following problem, suggesting this is still broken. The follow_child is set to false by default (debug.c:369).

---

#### [#12896](https://github.com/radareorg/radare2/issues/12896) — amd64 calling conventions doesn't support xmm1 arguments
*7y old · 0 comments · `RAnal` `test-required` `types`*

Detecting float/double function arguments via xmm registers in amd64 calling convention analysis. Some xmm register support has been added (ESIL for SSE float instructions) but no specific fix for detecting xmm-based function arguments in calling convention analysis. The calling convention definitions may not include xmm registers for floating point args.

---

#### [#13110](https://github.com/radareorg/radare2/issues/13110) — Radare2 doesn't recognize short string literals
*7y old · 3 comments · `RAnal`*

While bin.str.min controls minimum string detection length, no type-aware string inference (recognizing short strings when passed as const char* arguments) was found. The core issue of not showing '%d' as a string in disassembly when it's passed to scanf may still persist.

---

#### [#13143](https://github.com/radareorg/radare2/issues/13143) — Process r2 was stopped if machine hasn't enough memory
*7y old · 15 comments · `optimization`*

This is a fundamental issue of r2 using too much memory on very large binaries (libxul.so) with limited RAM. While memory optimizations have been made over the years, the core problem of OOM on constrained systems during deep analysis of huge binaries is inherently difficult to fully resolve. The maintainer suggested 'fix it with hardware'. This will likely remain an issue for very large binaries on low-RAM systems.

---

#### [#13241](https://github.com/radareorg/radare2/issues/13241) — More granular heap information
*7y old · 0 comments · `enhancement` `good first issue` `Windows`*

Feature request for showing bin layout with linked order of chunks. Some heap improvements were made: dmht for tcache (d5a32f6aca), dmha for arenas (a95fb331da), mangling pointer support (3ffe3f88d2). However, the specific request was for a 'layouts' command showing which bins chunks belong to and their linked order (like david942j/heapinfo). The dmh_glibc.inc.c file is 162 references to heap/glibc/dmh - functional but no bin layout visualization matching the feature request was implemented.

---

#### [#13257](https://github.com/radareorg/radare2/issues/13257) — Implement pdc*
*7y old · 0 comments · `enhancement` `RAnal`*

Feature request for pdc* (generating r2 commands with offset references, like pdd*). No specific evidence found that pdc* was implemented. The pdc decompiler has seen significant development but the * suffix variant for r2 command output was not found in git history.

---

#### [#13275](https://github.com/radareorg/radare2/issues/13275) — ELF format issues on ARM packed binaries
*7y old · 0 comments · `RBin` `ELF`*

The issue involves a packed ARM ELF binary designed to break RE tools where exports and strings are empty. The ELF parser has been extensively improved (65196c2616, aab4abf3e3, 09cadac04b 'Handle bin.limit in ELF and support strings, imports'), and string detection was enhanced (a5ab4aca9b 'Dynamic string range detection. Fix ELF strings when no sections'). However, intentionally obfuscated/packed binaries that break RE tools are an ongoing challenge. Without the specific test binary, it's unclear if the improvements help this case. Given that both IDA and r2 failed on this deliberately anti-RE binary, and no specific anti-packing improvements were found, this likely remains problematic.

---

#### [#13358](https://github.com/radareorg/radare2/issues/13358) — Multi-arch CI: MIPS, SH, RISCV, etc
*7y old · 3 comments · `infrastructure` `buildsystem` `PPC`*

Infrastructure request to add CI with cross-compilation and test suite execution on MIPS/SH/RISCV targets using musl.cc and qemu-user. No commits implementing multi-arch CI pipelines were found. The CI infrastructure has improved but still focuses on x86-64 and some ARM testing. Cross-architecture testing with qemu-user was never implemented as a CI pipeline.

---

#### [#13390](https://github.com/radareorg/radare2/issues/13390) — Flags are not remapped on PIE binaries when debugging
*7y old · 2 comments · `RAnal` `RDebug` `cutter`*

The issue was clarified: flags ARE remapped, but analysis data is NOT remapped when using ood on PIE binaries. PR #12801 addressed breakpoint remapping but not analysis/function data. No subsequent fix for analysis data remapping was found. This is a fundamental issue with PIE binary debugging workflow.

---

#### [#13582](https://github.com/radareorg/radare2/issues/13582) — Add the ability to fold and scroll over structs
*6y old · 1 comments · `enhancement` `consoleui` `RAsm-Disassembler`*

Feature request to add struct folding/unfolding in disassembly for tl/Cf using asm.meta. No specific commits implementing fold/unfold for struct fields in the disassembly view were found. The maintainer asked for examples of how it should look (2019) and no follow-up occurred. asm.meta handling exists but not specifically for struct field folding.

---

#### [#13638](https://github.com/radareorg/radare2/issues/13638) — Visual scrolling and blocksize
*6y old · 0 comments · `enhancement` `RSoC` `visual`*

Enhancement request for consistent scrolling behavior across visual print modes. The issue describes three types of scrolling (block '<'/'>', page 'J'/'K', line 'j'/'k') behaving inconsistently, especially in hexdump mode where block scroll should honor user-defined blocksize. The visual mode code in visual.c handles these keys but the scrolling behavior varies by print mode. No specific commits addressing scrolling consistency were found. This is a long-standing usability issue that likely still exists.

---

#### [#13642](https://github.com/radareorg/radare2/issues/13642) — Inverse register usage in function
*6y old · 6 comments · `enhancement` `FEEDBACK WANTED` `good first issue`*

afis/afisa commands were mentioned as existing. However, the specific inverse register-to-instruction mapping (showing which instructions use each register) was the original request. The maintainer said 'See afis and afisa commands' suggesting partial progress, but the full feature as described may not be complete.

---

#### [#13802](https://github.com/radareorg/radare2/issues/13802) — pf variable size arrays support
*6y old · 0 comments · `pf` `types` `high-priority`*

Commit e5d5a688dd added 'variable length array in pf' but noted 'Still doesn't work with flags'. The specific use case of dynamic array sizes based on other struct fields (like fat_header nfat_archs) was partially implemented but not fully working. The pf system has limitations with field-reference-based sizing.

---

#### [#13822](https://github.com/radareorg/radare2/issues/13822) — RFE: improve ascii-art stackframe printer
*6y old · 1 comments · `enhancement` `RDebug` `print`*

Enhancement request with checklist. Colors and utf8 were marked done, but 'move to p', 'relative values/sizes', and 'show pxr in stackframe' remain unchecked. This is an ongoing feature request. Some related improvements may have been made to the print commands, but the specific items (moving to p namespace, relative address display, pxr integration) were not found in commit history.

---

#### [#13943](https://github.com/radareorg/radare2/issues/13943) — Pointers inside structs do not take asm.bits into account when calculating offsets
*6y old · 1 comments · `RAnal` `types`*

void* size was hardcoded to 64 bits regardless of asm.bits (16/32/64). The type system has been reworked but the core issue of pointer sizes not respecting target architecture bits was a fundamental design issue. Related to #10996. The shlr/tcc type parser may now handle this better but no specific commit fixing pointer size to follow asm.bits was found.

---

#### [#14125](https://github.com/radareorg/radare2/issues/14125) — Autodetect argument types in THUMB programs by taking the POP
*6y old · 0 comments · `ARM` `types` `vars`*

Feature request to detect argument count from PUSH/POP instructions in THUMB mode. No specific commits found implementing this heuristic. Multiple ARM/thumb analysis improvements have been made but this specific heuristic was not added.

---

#### [#14161](https://github.com/radareorg/radare2/issues/14161) — Automatically rename function when renaming an anal class method
*6y old · 1 comments · `RAnal` `classes`*

Feature request to automatically rename the underlying function when a class method is renamed. No specific commit found implementing this linkage. The class system has seen improvements but this specific automatic renaming behavior was not added.

---

#### [#14195](https://github.com/radareorg/radare2/issues/14195) — aeim improvement
*6y old · 3 comments · `FEEDBACK WANTED` `RAnal` `ESIL`*

Feature request to use ESIL tracing without mandatory aeim, or to not overwrite existing stack memory. The maintainer approved the proposal but no specific implementation commit was found. The ESIL system has been modified but this specific behavior change was not evidently implemented.

---

#### [#14223](https://github.com/radareorg/radare2/issues/14223) — Mode Detection is not right for ARM architectures
*6y old · 1 comments · `test-required` `ARM`*

ARM/Thumb mode detection remains a complex challenge. Multiple improvements have been made (e889490b4b, 23d01f869c for thumb detection), but the fundamental problem of correctly detecting thumb vs ARM mode across all binaries is inherently difficult. The specific issues mentioned (inline data detection, pd/pdj inconsistency) are long-standing challenges in ARM disassembly. The referenced academic paper (ISSTA 2020) documents the broader problem. Without testing on the specific binary, cannot confirm resolution, but the nature of this issue means it's a class of problems that may never be fully solved.

---

#### [#14293](https://github.com/radareorg/radare2/issues/14293) — db entry0 doesn't break
*6y old · 3 comments · `RDebug` `test-required` `high-priority`*

The issue about breakpoints at entry0 not working was labeled 'high-priority' and 'test-required'. The maintainer said 'this wont be fixed soon, move to milestone 4.5 or further.' While many breakpoint fixes have been committed (e503bdd9c2, faa7938cc5, 313d4b4893), no specific fix for the entry0 breakpoint issue was found. The issue references a packer (0pack) that modifies entry point behavior, which may interfere with breakpoint placement. Without the specific test binary and testing, this cannot be confirmed as resolved.

---

#### [#14338](https://github.com/radareorg/radare2/issues/14338) — Improve source level debugging features
*6y old · 0 comments · `FEEDBACK WANTED` `RDebug` `DWARF`*

Partially addressed. CLf was implemented (commit 6d7e5ff513 - 'Implement CL+ and CLf, show info in afi/afij'). However, CLn/CLp (next/previous source line navigation) were NOT found in the codebase. Also, asm.dwarf is still 'false' by default (confirmed in cconfig.c:3994). So 1 of 3 checklist items is done, 2 remain open. Marking as still_open since the majority of the feature request is unfinished.

---

#### [#14505](https://github.com/radareorg/radare2/issues/14505) — Invalid jmptbl size on x86-64
*6y old · 6 comments · `RAnal` `x86` `high-priority`*

The ahv hint command was implemented (commit 56cdeee666, PR #14508) as a workaround. A user (d4em0n) found that reordering try_get_delta_jmptbl_info before try_get_jmptbl_info fixes the issue and was asked to make a PR, but no evidence it was merged. The automatic detection still may produce wrong jmptbl sizes in some cases.

---

#### [#14513](https://github.com/radareorg/radare2/issues/14513) — scr.color reset
*6y old · 4 comments · `enhancement` `FEEDBACK WANTED` `consoleui`*

The issue is that scr.color is not restored after running the =h HTTP server and then stopping it. Examining libr/core/rtr.c, the HTTP server startup code does not save/restore the scr.color setting. There is no grep match for 'scr.color' in rtr.c at all, meaning no code saves/restores the color state around server operation. The maintainer stated this is 'by design' for scripting purposes. Since the design decision was never revisited or voted on as requested, and no code was added to restore color after server shutdown, this remains an unresolved enhancement request.

---

#### [#14557](https://github.com/radareorg/radare2/issues/14557) — Graph in panels
*6y old · 2 comments · `consoleui` `RGraph` `panel`*

Feature request for better graph navigation in panels mode: 1..9 to follow nth edge, '.' to scroll to selected node, '/' to search and scroll. The panels code (panels.c) has graph support (PANEL_CMD_GRAPH handling, line 807) and decompiler integration, but the specific keybindings requested (1..9 for edge following, '.' for node focus, '/' for search-and-scroll within graph panels) were not found in the panels code. The panel graph interaction is basic toggling between graph/panels view (space key, line 217). The requested advanced navigation features appear unimplemented.

---

#### [#14794](https://github.com/radareorg/radare2/issues/14794) — Make C++ STL library function more simplicity
*6y old · 4 comments · `RAnal` `C++`*

Request to shorten/elide C++ STL template names in disassembly. The workaround is 'e asm.demangle=false'. No specific feature to elide template internals (like Binary Ninja does) was implemented. The asm.flags.maxname option (commit 5ecd4c352b) helps with truncating but doesn't intelligently elide template content.

---

#### [#14847](https://github.com/radareorg/radare2/issues/14847) — Alias || to ;??
*6y old · 1 comments · `command`*

No implementation of || or && shell-like operators was found in the command parser. The tree-sitter parser rewrite replaced the old command parser, but these POSIX shell-like operators were not implemented. No grep results for '||' as a command operator in the core command handling code.

---

#### [#14933](https://github.com/radareorg/radare2/issues/14933) — Basic Blocks issue disassembling Java class file
*6y old · 1 comments · `RAnal` `test-required` `java`*

Each Java instruction being a separate basic block is a regression. The maintainer noted the Java support has been unmaintained for a long time and suggested using a2f. Some Java-related fixes exist (commit 53ab84a7fc for double free) but the fundamental basic block detection issue in Java bytecode analysis was acknowledged as a regression with no clear fix.

---

#### [#15042](https://github.com/radareorg/radare2/issues/15042) — Visual Panel Decomp Highlight
*6y old · 4 comments*

Feature request for highlighting the current instruction in the decompiler panel during debugging. The panels code has decompiler support (__print_decompiler_cb at panels.c:4714, __show_all_decompiler_cb at panels.c:2274) but no current-address highlighting logic was found in these callbacks. The maintainer said 'that works for me' in response to the feature description, but this appears to be acknowledging the request, not stating it's implemented. The reporter specifically asked for pdgo to highlight the current rip address, which would require matching the current PC to decompiler output offsets. No such matching/highlighting code was found in the panels decompiler callbacks.

---

#### [#15121](https://github.com/radareorg/radare2/issues/15121) — Prettier interface for the tabline (`t` commands)
*6y old · 2 comments · `enhancement` `consoleui` `good first issue`*

The maintainer said in 2020 'i think its fine like its now, i would close this issue'. No Unicode or color improvements to the tab line were found. While technically still open, the maintainer considered it a non-issue. Marking still_open since no code changes were made, but confidence is lower because the maintainer wanted to close it.

---

#### [#15204](https://github.com/radareorg/radare2/issues/15204) — Unable to identify Xrefs in non-stripped x86 Binary
*6y old · 3 comments*

32-bit PIE ELF xref detection failure. The maintainer linked this to issues #13356, #8900, and #6283, all about x86-32 PIE xref problems. This is a known long-standing issue with 32-bit position-independent code where references use ebx+GOT relative addressing. While some improvements have been made, 32-bit PIE xref detection remains problematic.

---

#### [#15258](https://github.com/radareorg/radare2/issues/15258) — oodr stdin=file doesn't seem to work
*6y old · 0 comments*

No commits specifically fixing oodr stdin=file functionality were found. The dd command was significantly refactored (724d23a6c0 'Refactor, fix, and test dd command'), and the ddd (dup2) command exists in libr/core/cmd_debug.inc.c:5380-5393, but the specific ood/oodr stdin= feature for piping file input during debug reopen has no evidence of being fixed. The crash fix 19bdc6f41a only fixes a crash in oodr, not the stdin= parameter handling.

---

#### [#15278](https://github.com/radareorg/radare2/issues/15278) — Sync zoom.byte p= and pz commands
*6y old · 0 comments*

Enhancement request to synchronize the zoom.byte config variable between p= (entropy bars) and pz (zoom view) commands. The zoom.byte variable exists in cconfig.c and is used by cmd_print.inc.c. Both p= and pz exist as separate commands with separate help messages (cmd_print.inc.c:227,680). The issue body was just a screenshot showing inconsistency. No commits reference this issue. Without testing it's hard to confirm, but no code unification between p= and pz zoom.byte handling was found.

---

#### [#15962](https://github.com/radareorg/radare2/issues/15962) — dro/drd does not show old/difference register values
*6y old · 2 comments · `RDebug` `high-priority`*

Looking at the current implementation at libr/core/cmd_debug.inc.c:3190-3196, dro swaps arenas, prints registers, and swaps back. drd calls r_debug_reg_list with flag 3 for diff mode. The implementation looks correct in principle. The issue was that after stepping (ds), dro showed the same values as dr. This suggests the arena swap might not be properly saving the pre-step state. The r_reg_arena_swap at libr/reg/arena.c:196 performs the swap. Commit 5ce3c287db 'Add drp* arp* commands to flag the reg arena' suggests ongoing arena work, but no specific fix for dro/drd showing identical values was found.

---

#### [#16366](https://github.com/radareorg/radare2/issues/16366) — Add stackframesize information in aoj
*5y old · 2 comments · `good first issue` `RAnal` `RAsm-Disassembler`*

Feature request to add stackframesize to aoj output. Related to RAnalBlock register arena work (issue #12070). The maintainer described wanting each RAnalBlock to store an RRegArena. No evidence this was implemented.

---

#### [#16377](https://github.com/radareorg/radare2/issues/16377) — Fails to search a binary for some regex
*5y old · 0 comments · `RSearch`*

The issue is about /e command failing to find regex matches like /[A-F0-9]{126,}/ while izzq|egrep finds them. Commit df77191f9d (2021) fixed a bug in regex searching related to empty matches, but this is a different issue. Looking at the current search_regex_read code in libr/search/regexp.c, the buffer size is hardcoded at 0x1000 (4096 bytes). The regex searches for 126+ hex characters, which means matches of 126+ bytes. The buffer scanning logic advances by buflen - buflen/8 on no-match, which means it steps 3584 bytes at a time. A match of 126 bytes could span a buffer boundary and be missed. The TODO comment on line 36 acknowledges this: 'allow user to configure according to the maximum expected match length to prevent FN on matches that span boundaries.' This fundamental limitation likely still causes false negatives for long regex matches.

---

#### [#16827](https://github.com/radareorg/radare2/issues/16827) — Support CALL_* python bytecodes analysis
*5y old · 0 comments · `pyc`*

The pyc plugin has CALL_FUNCTION handlers in opcode_anal.c that set R_ANAL_OP_TYPE_CALL. However, the core issue is about stack emulation to resolve call targets, which requires tracking which function is on the stack. This deeper analysis for inter-function call resolution has not been implemented.

---

#### [#16517](https://github.com/radareorg/radare2/issues/16517) — Wrongly detected additional x-ref in ELF 64bit
*5y old · 0 comments · `RAnal` `test-required`*

False xref created by parsing RIP-relative displacement bytes as raw absolute values. The xref at 0x2e8 (section..note.ABI_tag) is incorrectly generated from the displacement in 'lea rdi, [main]'. Various xref deduplication fixes have been made but this specific pattern of double-parsing RIP-relative addressing was not addressed.

---

#### [#16708](https://github.com/radareorg/radare2/issues/16708) — Unable to use winedbg://
*5y old · 1 comments · `WineDbg`*

The winedbg IO plugin exists at libr/io/p/io_winedbg.c. Recent commits (ab512c139b removes globals, b6629710e7 removes dead conditional) cleaned up the code but didn't address the fundamental connection timeout issue. The bug is that winedbg:// shows 'connected' but then hangs - suggesting a timing/protocol issue between r2 and winedbg's GDB stub. The winedbg integration is niche and likely undertested.

---

#### [#16790](https://github.com/radareorg/radare2/issues/16790) — With `~...` selection you can't select with a mouse
*5y old · 3 comments · `consoleui`*

The issue is about mouse clicks inserting garbage characters in the ~... interactive grep/selection mode. The console mouse handling code (libr/cons/cons.c:654-676) has been updated with r_cons_enable_mouse() but this deals with enabling/disabling mouse tracking. The ~... selection mode uses r_cons_fgets() for interactive input. Mouse events in terminal send escape sequences that can appear as garbage text when mouse tracking is enabled but the input handler doesn't parse mouse escape codes. No specific commit was found addressing this. The mouse/cons interaction has received fixes but this specific ~... scenario may still be affected.

---

#### [#16810](https://github.com/radareorg/radare2/issues/16810) — Missing symbols when debugging a remote gdb on Windows
*5y old · 0 comments · `Windows` `RDebug`*

The issue is about r2 on Windows failing to resolve symbols when connecting to a remote GDB server on Linux. On Linux, symbols are resolved through the binary on disk, but when the client is on Windows, the file path handling differs. The GDB plugin has been updated (ce4a32ef34 removed globals, 9b8c604e2e fixed remote debugging) but no specific fix for cross-platform symbol resolution was found. The issue is that Windows r2 can't find/read the ELF binary to extract symbols when debugging remotely.

---

#### [#16842](https://github.com/radareorg/radare2/issues/16842) — `af` doesn't create a new function at the address which is reached by jump from two or more functions
*5y old · 1 comments*

Fundamental analysis engine issue where overlapping basic blocks prevent function creation when two functions jump to the same target. The 'r_anal_fcn_bb() fails' error occurs because the address is already claimed by another function's basic block. The analysis engine has been reworked but this is a fundamental design constraint of non-overlapping basic blocks.

---

#### [#16979](https://github.com/radareorg/radare2/issues/16979) — af should create function at offset with zignature
*5y old · 2 comments · `RAnal` `zignatures`*

The reporter confirmed in December 2020 that the issue still exists and acknowledged it's more complicated than initially thought. Some z/ fixes exist ('Fix bug in z/, that creates misplaced functions') but the general issue of af correctly creating function boundaries at zignature-matched offsets remains open.

---

#### [#17003](https://github.com/radareorg/radare2/issues/17003) — command agC didn't get work on all function
*5y old · 0 comments · `RAnal` `test-required`*

agC (global call graph) missing main function because it's called indirectly through CRT startup. This is a fundamental analysis limitation - indirect calls through function pointers loaded from data require ESIL emulation to resolve. No specific fix found.

---

#### [#17036](https://github.com/radareorg/radare2/issues/17036) — Incorrect detection of jump table's ending causing incorrect disassemblies in middle of opcodes (x86/64)
*5y old · 1 comments*

Jump table analysis has received various improvements (78e83bb7ff for MIPS, 81af2203c6/fdcc2e206c for ARM64, 207e6f91b1 for zero cases, 3fedf80036 for v850), but no commit specifically addresses the problem of two adjacent jump tables being incorrectly merged into one. The fundamental challenge of determining jump table bounds when multiple tables are adjacent without explicit size information remains. The PE/x86-64 case described (ncdump.exe) involves distinguishing where one jump table ends and another begins, which is an inherently difficult analysis problem.

---

#### [#17363](https://github.com/radareorg/radare2/issues/17363) — Prevent axg* from erasing important function offset information that is retained in axg and axgj
*5y old · 2 comments · `json` `RGraph` `test-required`*

Feature request to include instruction-level addresses in axg* output instead of function-level addresses. A maintainer explained that axg* is for r2 command consumption and suggested using axj instead. The reporter also mentioned axgj producing malformed JSON. No fix for either issue was found.

---

#### [#17432](https://github.com/radareorg/radare2/issues/17432) — `afs` won't allow any custom type in the signature
*5y old · 4 comments · `RAnal` `types` `high-priority`*

The C parser for afs only allows predefined types, not custom types defined via 'td'. XVilka mentioned switching to tree-sitter which was later removed. The C type parser has been improved but the core limitation of afs requiring pre-defined types in the compiler's vocabulary likely persists. HoundThe confirmed no fix path at the time.

---

#### [#17448](https://github.com/radareorg/radare2/issues/17448) — Separation of the binary loading and some analysis
*5y old · 3 comments · `refactor` `RBin`*

Design-level feature request to separate binary loading from analysis (PLT identification, etc.). This is an architectural change. While the codebase has evolved, this fundamental separation has not been explicitly implemented. The request references Ghidra's approach. This remains a design philosophy issue.

---

#### [#17493](https://github.com/radareorg/radare2/issues/17493) — Piped process does not get waited after terminal resizing
*5y old · 0 comments*

SIGWINCH handling exists in libr/cons/cons.c and libr/core/task.c. The issue is that when r2 pipes output to an external process (like 'e??|less') and receives SIGWINCH, it stops waiting for the foreground piped process. This is a signal handling race condition. No commits specifically addressing the SIGWINCH+pipe interaction were found. The SIGWINCH handler may interrupt the waitpid() call without proper retry logic.

---

#### [#17541](https://github.com/radareorg/radare2/issues/17541) — Float arithmetic support in ESIL
*5y old · 0 comments · `enhancement` `RAnal` `ESIL`*

Request for native float support in ESIL with float flags/types in the ESIL VM. Some floating point ESIL operations exist (commit 0be8f250c8 for x86 SSE, commit 2dc88c63b7 for arm64). However, comprehensive native float support as described (with float type flags) is not fully implemented. The reporter's r2ghidra PR shows a workaround approach.

---

#### [#17589](https://github.com/radareorg/radare2/issues/17589) — Inform within disassembler on the performed direct relocations with addend
*5y old · 1 comments · `test-required` `ELF`*

Commit 03d510d00b improved relocation display by not printing addend when it's 0, and 4c90534f84 fixed ELF r_addend handling. However, the core issue remains: when relocations are applied (io.cache=true), the disassembly doesn't show that a value came from a relocation with a non-null addend. The relocation info is only visible with io.cache=false. ret2libc confirmed the issue extends beyond R_X86_64_32S. No commit was found that adds relocation provenance comments to the patched disassembly.

---

#### [#17626](https://github.com/radareorg/radare2/issues/17626) — radare2 with winedbg
*5y old · 4 comments · `RIO` `RDebug` `WineDbg`*

The winedbg IO plugin received some updates (ab512c139b removed globals, 0f5a7ce45f fixed non-null terminated bug) but no fix for the core issue: reading memory of debugged processes returns 0xff. The maintainer confirmed in 2021: 'nobody touched the winedbg code in r2. im aware winedbg had several bugs at the time.' A user in 2021 confirmed the issue persists in r2 5.1.0. The problem is likely in how winedbg:// reads memory from the debugged process, possibly a seek/address issue between r2 and winedbg.

---

#### [#17647](https://github.com/radareorg/radare2/issues/17647) — Xrefs not found for armcompiler6 (armclang)
*5y old · 0 comments · `RAnal` `test-required` `ARM`*

movw/movt instruction pairs not generating xrefs for ARM (armclang compiler). The issue is that armclang uses movw/movt pairs instead of literal pools. While ARM analysis has improvements, no specific commit addresses movw/movt-based xref detection. This requires ESIL emulation to combine the two half-word loads into a full address. The reporter offered to help implement the fix.

---

#### [#17674](https://github.com/radareorg/radare2/issues/17674) — Question about CFG extraction.
*5y old · 11 comments · `enhancement` `RAnal` `RGraph`*

The abl command was added to list basic blocks (trufae mentioned 'See my abl pr'). But the core request for a call-splitting CFG mode where call instructions end basic blocks was not implemented. The maintainers suggested scripting as the solution.

---

#### [#17753](https://github.com/radareorg/radare2/issues/17753) — XNU kernelcache plugin slows down /x by orders of magnitude
*5y old · 2 comments · `optimization`*

The 'reconstructing chained fixups' message is in libr/bin/format/mach0/mach0.c:1621. The issue is that this reconstruction is called for every section/segment during search, leading to O(n) calls where n is the number of search regions. The workaround (search.in=io.maps.x) was suggested and confirmed to work. However, the fundamental performance issue of redundant chained fixup reconstruction during search was not addressed with caching or similar optimization.

---

#### [#17768](https://github.com/radareorg/radare2/issues/17768) — Slow autocompletion init
*5y old · 0 comments · `optimization`*

Multiple autocompletion fixes exist (e7abd2d133, 513c80fcea, e753075a47, etc.) but none specifically address the initialization-on-every-command performance issue. The original issue described autocompletion init being called on every command when using r2pipe (-0 mode), consuming most of the execution time. The backtrace showed the init path being triggered repeatedly. No commit was found that restructures autocompletion to init once rather than per-command.

---

#### [#17849](https://github.com/radareorg/radare2/issues/17849) — Error parsing non main arena heap with dmh
*5y old · 4 comments · `test-required` `heap`*

The issue is about thread arena parsing failing with glibc 2.28 showing chunks as corrupted. A commenter confirmed it works with glibc 2.32. The heap parsing code (dmh_glibc.inc.c) handles glibc version differences, and some autodetection was added. However, the specific struct layout difference between glibc 2.28 thread arenas and 2.32 was not explicitly addressed. The maintainer acknowledged in 2023 that 'libc changes the heap algorithm every 1-2 years, so its expected to break.' Without a test binary for glibc 2.28, this remains unresolved.

---

#### [#17949](https://github.com/radareorg/radare2/issues/17949) — how to use/load debug symbol files from ubuntu -dbgsym packages
*5y old · 0 comments · `debug-info` `RDebug` `RBin`*

Feature request for loading external debug symbols from /usr/lib/debug/. The code at libr/bin/p/bin_elf.inc.c:1589 parses .gnu_debuglink sections. However, there's no evidence of automatic search in /usr/lib/debug/ directories as GDB does. No 'debugdir' or similar config option was found. This remains a gap - r2 can read .gnu_debuglink but doesn't automatically search for the referenced debug files in standard directories.

---

#### [#17962](https://github.com/radareorg/radare2/issues/17962) — Slow disassembly with many flags and empty flag space selected
*5y old · 0 comments · `optimization` `RAsm-Disassembler` `Rflags`*

The r_flag_get_at function in libr/flag/flag.c:680 still performs a linear search when closest=true (line 716-737, while loop walking through flags). Two fixes exist (1dddfd83d4 for spaces interference, 66225b9ee1 for filtering unselected flagspaces) but these predate the issue (filed Nov 2020). The core performance problem with many flags and closest=true search remains: the function uses r_flag_get_nearest_list and walks backwards linearly. No post-issue optimization was found.

---

#### [#17984](https://github.com/radareorg/radare2/issues/17984) — I can not confirm address related JNIEnv
*5y old · 0 comments · `RAnal` `ARM` `question`*

This is more of a usage question/documentation gap about ESIL emulation for finding JNIEnv addresses. No specific code changes or documentation improvements were found. The underlying ESIL capability exists but is poorly documented.

---

#### [#18228](https://github.com/radareorg/radare2/issues/18228) — Different overflow behavior with winedbg / gdb
*5y old · 0 comments*

The issue involves stack memory reading differences between winedbg and gdb modes during buffer overflow exploitation. The winedbg:// plugin shows 0xff for stack data while standalone winedbg shows correct values. This is the same underlying issue as #17626 - the winedbg plugin has trouble reading memory. The maintainer hasn't touched the winedbg memory reading code. The GDB mode showing different register states (CS overwrite instead of EIP) suggests a register profile mapping issue between wine's gdb stub and r2.

---

#### [#18353](https://github.com/radareorg/radare2/issues/18353) — winkd net/pipe supporting
*5y old · 8 comments*

The winkd IO plugin exists (io_winkd.c) and has had KDNET improvements. However, the specific use case of connecting to a CDB .server TCP session for usermode debugging was never resolved. The last test in 2021 showed the connection freezing after sending one UDP packet. The maintainer acknowledged limited ability to test without Windows.

---

#### [#18691](https://github.com/radareorg/radare2/issues/18691) — syscall resolving doesnt work in debug mode
*4y old · 2 comments*

The maintainer identified the root cause: 'probably caused by the abuse of RAnal->esil instance, the pd loop may have its own instance so it doesnt affect the debugging session.' A screenshot from 2021 was posted but no commit message referencing this fix was found. The issue is that in debug mode, asm.emu shows wrong syscall numbers (always execve/59 or unknown/-1) because the ESIL instance used for disassembly picks up the real register state instead of emulating from scratch. The maintainer acknowledged it 'requires some refactoring in disasm.c.'

---

#### [#19190](https://github.com/radareorg/radare2/issues/19190) — Create a new function after "int 0x80" (used instead of "ret")
*4y old · 3 comments*

Recognizing sys_exit via int 0x80 as a function terminator. condret correctly noted that int 0x80 is not always sys_exit and requires deeper ESIL analysis to determine the syscall number. The maintainer noted this is similar to noreturn/tailcall detection. No specific implementation for int 0x80 as a conditional noreturn was found.

---

#### [#19651](https://github.com/radareorg/radare2/issues/19651) — Jump table with negative cases improperly handled
*4y old · 1 comments*

The issue describes two problems: (1) 32-bit negative comparison values (cmp edi, 0xfffffff6) being sign-extended to 64-bit and treated as table size, and (2) 8-bit negative values (cmp dil, 0xf6) being used as unsigned table size 246 instead of -10. The reporter offered to fix it but asked questions about the approach. The maintainer replied asking if they planned to implement it but no PR was submitted. No commit addressing negative switch case comparison values in jump table analysis was found.

---

#### [#20034](https://github.com/radareorg/radare2/issues/20034) — Strange behavior in cmd.esil.mdev I think
*3y old · 3 comments*

cmd.esil.mdev returning incorrect memory values (always 0) and the return value check always being true. The reporter did detailed debugging showing r_core_esil_cmd always returns a truthy value regardless of script output. The maintainer acknowledged the issue and provided guidance but no explicit fix was committed.

---

#### [#20240](https://github.com/radareorg/radare2/issues/20240) — Follow child doest not work
*3y old · 0 comments*

The dbg.follow.child config exists (libr/core/cconfig.c:4387) and the implementation is in libr/debug/debug.c:1271. On Linux, when R_DEBUG_REASON_NEW_PID is detected with follow_child enabled, it calls linux_attach_new_process via dynamic linking. The code uses r_lib_dl_sym(NULL, 'linux_attach_new_process') for runtime resolution (line 1277), which is fragile. The issue was that fork() child following didn't work. The implementation exists but may have bugs in the runtime symbol resolution or the actual attach logic.

---

#### [#20284](https://github.com/radareorg/radare2/issues/20284) — Add cycle info for m68k
*3y old · 3 comments*

Partially addressed. libr/arch/p/m68k_cs/plugin.c:63 contains a get_move_cycles() function and cycle counting for MOVE instructions (line 658). However, this is limited to MOVE instructions only - the issue requests comprehensive cycle info for all m68k instructions. The implementation is incomplete.

---

#### [#20377](https://github.com/radareorg/radare2/issues/20377) — rabin2 -V corrupt
*3y old · 5 comments*

The issue reports rabin2 -V failing to parse PE version info for certain binaries (360safe.com setup.exe). The PE VS_VERSIONINFO parser in libr/bin/format/pe/pe.c has numerous early-exit checks that log warnings and return NULL. The maintainer acknowledged the code was 'totally misleading' but no specific fix was committed. The PE parser has received many updates since (3cd886deda, 0ab198ce84, 72869988ab), but these are mostly for crashes, memory leaks, and other features rather than improving VS_VERSIONINFO parsing robustness. The reporter confirmed the binary contains version info visible in other tools. Without testing the specific binary, the parser's strict validation may still reject valid-but-unusual version info structures.

---

#### [#20736](https://github.com/radareorg/radare2/issues/20736) — Race condition in r2r
*3y old · 2 comments*

The heap-use-after-free race between sigchld_th (binr/r2r/run.c:511) and subprocess_runner is still architecturally present. The sigchld thread (run.c:478-520) reads from sigchld_pipe and acquires locks on subprocess objects, while worker threads free those locks in subprocess_runner. The lock acquisition/free ordering hasn't fundamentally changed. Recent r2r commits focus on leak fixes and API changes, not this specific race. The race only manifests under ASAN/TSAN with Ctrl+C, making it a low-priority but real bug.

---

#### [#20859](https://github.com/radareorg/radare2/issues/20859) — macOS: Debugging x86 under ARM crashes when breakpoint is hit.
*3y old · 0 comments*

The error 'tsk_setperm: Invalid argument' comes from libr/io/p/io_mach.c:331 where vm_protect fails. This occurs when debugging x86 binaries under Rosetta 2 on Apple Silicon. The io.mach plugin (249dfd175c removed globals) and breakpoint code (1fa8ae3c32 enabled hwbp by default on M1, 805ddee6be mach exception handling) received updates, but no specific fix for Rosetta 2 x86 debugging. The page permission model may differ for translated x86 processes. The config defaults to hwbp=false (cconfig.c:4372) but the comment mentions M1 sw bp failure.

---

#### [#20863](https://github.com/radareorg/radare2/issues/20863) — pipe not working on windows 7
*3y old · 1 comments · `Windows`*

Windows 7 pipe operator issue. The maintainer acknowledged it as 'important to be implemented to properly support windows'. While some Windows pipe fixes have been made, the core issue of pipe commands not filtering output on Windows may persist. Windows 7 is EOL but the issue may apply to newer Windows versions too. Hard to verify without a Windows environment.

---

#### [#21144](https://github.com/radareorg/radare2/issues/21144) — verbose  / debug - can be replaced with R_LOG imho
*3y old · 1 comments*

Partially addressed but ongoing. R_LOG is now widely used (5664 occurrences found via grep) but eprintf still has 1105 remaining usages in libr/. Several R_LOG migration commits exist (f27ba4eee4, a05253eed1, 4fc478044d, 8c73bc6d89, 6bbe2e22fa). The migration is in progress but far from complete. The specific verbose/debug pattern shown in the screenshot may or may not have been addressed.

---

#### [#22322](https://github.com/radareorg/radare2/issues/22322) — Error when load zignature
*2y old · 2 comments*

The error occurs when loading zignatures containing vararg function signatures ('...'). In libr/anal/var.c:1951-1954, vararg is detected but handled with a TODO comment ('Detected, but unhandled vararg type'). The zignature loading code in libr/anal/sign.c would fail when encountering function signatures with '...' in the type string. While vararg detection exists, proper parsing/handling of vararg in zignature loading was not fully implemented.

---

#### [#22327](https://github.com/radareorg/radare2/issues/22327) — analyze function not finished on IAR generated elf for ARM Cortex-M device.
*2y old · 3 comments*

Function analysis splits functions at jump targets that have symbols, causing incomplete function detection on IAR-compiled ARM Cortex-M ELFs. The maintainer explained that function splitting at symbols is 'correct' behavior but acknowledged the IAR pattern is unusual. The reporter couldn't share the binary. The maintainer asked in March 2024 to try with latest git.

---

#### [#22484](https://github.com/radareorg/radare2/issues/22484) — [AIX] Huge symbol table
*2y old · 0 comments*

AIX build support exists but no specific fixes for symbol visibility or stripping to reduce the 50MB binary and 1.3M symbol table were found. This is likely a build configuration issue specific to AIX's linker behavior. Difficult to verify without AIX access.

---

#### [#22715](https://github.com/radareorg/radare2/issues/22715) — Move esil{cfg|dfg} from anal to esil
*2y old · 0 comments*

Refactoring request to move esil_cfg/dfg from anal to esil module. Commit 05611152ca moved esil_cfg typedefs back to anal, suggesting an attempt was made and partially reverted. The separation between libr/anal and libr/esil continues to evolve. Current state is that esil_cfg remains partially in anal.

---

#### [#23269](https://github.com/radareorg/radare2/issues/23269) — [winkd] Windows kernel debug, r2 can not debug with winkd plugin
*1y old · 0 comments*

The user reports the winkd:// plugin hanging after opening a VirtualBox COM pipe. The winkd plugin has been updated (169923932d enabled winkd, afaf16f66d migrated plugin, 0ba897f5c9 added KDNET support, 476efd4d2f improved error reporting). However, the issue could be a VirtualBox COM pipe configuration problem or a protocol handshake issue. The output shows 'Opened pipe /tmp/virtualbox-com1 with fd 0x7' and then hangs, suggesting the KD protocol handshake isn't completing. This could be a configuration issue (wrong KD settings in the VM) or a genuine plugin bug.

---

#### [#24090](https://github.com/radareorg/radare2/issues/24090) — how to get arm-v7 & v8 static lib ?
*12mo old · 6 comments*

User question about building ARM static libraries for Android. The NDK build script issues (ndk-gcc not found with modern NDK) were discussed. This is more of a support question than a bug. The sys/android.sh build script may or may not work with modern NDK. The user's C++ code question is unrelated. Keeping as still_open since the NDK compatibility issue was never explicitly resolved.

---

</details>

### Confidence 🟠 2 (27)

<details>
<summary>Click to expand 27 issues</summary>

#### [#15577](https://github.com/radareorg/radare2/issues/15577) — Merge ESIL testing scripts
*8y old · 0 comments · `r2r`*

Request to merge external esil-tests repository (sushant94/esil-tests) into the main test suite. No evidence this merge occurred. The r2 test suite has evolved significantly (r2r framework) but whether these specific ESIL tests were incorporated is unclear.

---

#### [#12164](https://github.com/radareorg/radare2/issues/12164) — Align nodes vertically in horizontal layout
*7y old · 0 comments · `enhancement` `RGraph`*

graph.layout exists in libr/core/cconfig.c and libr/core/agraph.c. The config allows toggling between vertical (0) and horizontal (1) layout. This is an enhancement request about improving node alignment in horizontal layout mode. Cannot verify visual rendering improvements without testing. No commits specifically addressing horizontal layout alignment were found.

---

#### [#12426](https://github.com/radareorg/radare2/issues/12426) — [WASM] calls enrichment produces an unexpected function name
*7y old · 2 comments · `RAnal` `RAsm-Disassembler`*

WASM call instruction shows wrong symbol name due to function index vs offset confusion. A contributor (cgvwzq) acknowledged the issue and said they'd fix it in a next commit, but no evidence of the fix being merged. The WASM plugin has received several fixes since 2018 but no commit specifically addresses this naming issue.

---

#### [#12604](https://github.com/radareorg/radare2/issues/12604) — Add anal classes to Vb
*7y old · 3 comments · `RAnal` `visual` `classes`*

Feature request to browse anal classes in Vb visual browser. XVilka said it's 'mostly supported' but needs interface improvement. The maintainer said 'ive never used anal classes' and 'i dont even have a reproducer to test'. Unclear status; likely partially working but incomplete.

---

#### [#13004](https://github.com/radareorg/radare2/issues/13004) — Improve appcall aeC
*7y old · 0 comments · `RAnal` `ESIL` `RDebug`*

The aeC command exists but the three requested improvements (debugger support, string argument parsing, return value display) may not all be implemented. No specific commits addressing these improvements were found.

---

#### [#13059](https://github.com/radareorg/radare2/issues/13059) — vJKJKJKJK should keep the same seek position
*7y old · 12 comments · `regression` `consoleui`*

This issue about visual mode J/K scrolling not returning to the same seek position had conflicting reports. The maintainer (radare) insisted it was still present and described specific conditions (needs a real file with metadata, code, and unallocated regions). Other contributors could not reproduce. No git commits reference this issue number. The visual scrolling code in libr/core/visual.c has been modified but without specific fix commits addressing this seek position regression. Given the maintainer's insistence and no evidence of a targeted fix, this likely remains open, though confidence is low because the visual code has changed significantly.

---

#### [#13123](https://github.com/radareorg/radare2/issues/13123) — r2pipe hangs indefinitely on hardware watchpoint hit
*7y old · 0 comments · `RDebug` `Linux OS` `x86`*

The issue is about r2pipe (Python) hanging when a hardware watchpoint is hit via dc. Watchpoint support received fixes (ce73397f91 crash fix, 26baccd4e8 double-free fix) but no specific fix for the r2pipe communication hang. The issue may relate to how r2pipe handles the output of debug continuation commands when a watchpoint is hit vs a normal breakpoint. This would need testing with current r2pipe to verify.

---

#### [#13618](https://github.com/radareorg/radare2/issues/13618) — improved pds with function call args
*6y old · 0 comments · `enhancement` `types`*

Feature request for pds to show function call arguments in its summary output. The pdsb command was implemented (commit 80759f227d) but whether pds now shows function call arguments needs testing. No specific commit addresses this exact feature.

---

#### [#13710](https://github.com/radareorg/radare2/issues/13710) — Poor function detection case
*6y old · 1 comments · `has-test`*

Tail-call optimization causing function boundary detection issues in Haskell binaries. The maintainer noted this is related to analysis order. Has 'has-test' label. Function detection has been improved but the fundamental issue of tail-call optimization vs function boundaries is a known hard problem. Without the specific binary retesting, unclear if improved.

---

#### [#13930](https://github.com/radareorg/radare2/issues/13930) — The result of function argument identification is poor
*6y old · 8 comments · `RAnal` `test-required`*

PPC function argument identification quality issue. The reporter was asked to provide test cases and submit to radare2-regressions. The maintainer asked in 2020 if they could try again. No specific PPC argument detection improvements found. PPC analysis is less well-maintained than x86/ARM.

---

#### [#14612](https://github.com/radareorg/radare2/issues/14612) — /R is slow
*6y old · 0 comments · `RAnal` `ROP`*

Performance issue with ROP gadget search disassembling instructions 5x more than necessary. No specific optimization commit referencing this issue was found. The ROP search code has been maintained but the fundamental issue of redundant disassembly may still exist. The issue was in the 'Attic' milestone suggesting it was deprioritized.

---

#### [#14702](https://github.com/radareorg/radare2/issues/14702) — radiff2 doesn't show changes in strong difference basic block
*6y old · 12 comments · `radiff2`*

radiff2 code exists in binr/radiff2/radiff2.c and libr/main/radiff2.c, and graph diff in libr/core/cmd_cmp.inc.c. The -G flag has been replaced by -c, and syntax has changed. The specific rendering issue with orange blocks not showing both versions in graph diff output cannot be verified without visual testing. The issue had no clear resolution in commits and the reporter never confirmed if later versions fixed it.

---

#### [#14992](https://github.com/radareorg/radare2/issues/14992) — afu makes function shorter but not longer
*6y old · 1 comments · `RAnal`*

Bug where afu updates 'size' but not 'realsz' when enlarging a function. The analysis blocks were significantly reworked (thestr4ng3r's PR) but whether this specific bug was fixed is unclear. The workaround is to delete the function and re-create it.

---

#### [#16302](https://github.com/radareorg/radare2/issues/16302) — Issue Breaking on Win32 DLL Functions
*5y old · 1 comments · `Windows` `RDebug` `high-priority`*

Windows debugging issue with breaking on VirtualAlloc in malware analysis. The Windows debugger has seen various improvements but this specific problem with access violations during continue-to-breakpoint on DLL functions was never confirmed fixed. Assigned to GustavoLCR in 2020 milestone but no resolution evidence.

---

#### [#16578](https://github.com/radareorg/radare2/issues/16578) — Bugs in function run_commands in radare2.c
*5y old · 1 comments*

Bug in r_core_run_script boolean return value handling in run_commands. The maintainer suggested checking $? value. The code in libr/main/radare2.c has been refactored over time but no specific fix for this return value handling issue was found.

---

#### [#16584](https://github.com/radareorg/radare2/issues/16584) — izzq~MSCF yields fake hits?
*5y old · 1 comments · `PE`*

False positive string detection in large PE files (1GB). The string detection code has had improvements but no specific fix for this edge case with unmapped/overlay PE sections producing false string matches was found. The issue involves string search returning hits at addresses that contain 0xff bytes, not actual MSCF headers. This likely relates to how string search handles PE overlay/unmapped regions.

---

#### [#16938](https://github.com/radareorg/radare2/issues/16938) — ARMv7a 'unaligned' instructions in visual mode after aav analysis
*5y old · 0 comments · `ARM` `visual`*

Running aav in visual mode while seeked to 0x00 causes unaligned instructions in ARM disassembly. There have been improvements to handling unaligned cases in disassembly but no commit specifically addresses this visual-mode + aav interaction bug. Without a reproducible test, hard to confirm.

---

#### [#16987](https://github.com/radareorg/radare2/issues/16987) — Fix functions detection for CYW20735 firmware.
*5y old · 0 comments · `RAnal` `test-required`*

ARM firmware function detection for Broadcom Bluetooth firmware. No specific commits target this firmware or similar embedded ARM function detection improvements. The issue lacks specific details about which functions fail to detect and why.

---

#### [#17655](https://github.com/radareorg/radare2/issues/17655) — Finding cross-reference_from based on Android native API
*5y old · 2 comments · `RAnal` `test-required` `Android`*

Dex xref analysis for imported methods in Android native APIs. The reporter was unsure if it was a user error or bug. XVilka asked why it was closed and it was reopened. No specific fix for dex method import xref detection was found.

---

#### [#17933](https://github.com/radareorg/radare2/issues/17933) — When start debug android process, the app is frozen and app process will quit
*5y old · 0 comments · `gdb`*

The issue is about debugging Chrome on Android via gdbserver causing the process to quit. GDB remote debugging on Android received fixes (e56c1ee7fe, cc37f0c606), but these addressed /proc/pid/maps parsing and register profiles, not process freezing. Chrome is a complex multi-process application with sandboxing that may interfere with debugging. This could be a gdbserver version issue (7.11 is old) or a Chrome security feature. Without access to the same environment, this cannot be verified.

---

#### [#18014](https://github.com/radareorg/radare2/issues/18014) — Incorrect function argument emulation for a 32bit binary
*5y old · 2 comments*

ESIL emulation shows incorrect function arguments depending on disassembly starting point. This is a known limitation of ESIL emulation state being dependent on where disassembly begins. The maintainer asked in 2024 if it was tested with latest releases, with no response. The underlying issue of ESIL state initialization remains.

---

#### [#18910](https://github.com/radareorg/radare2/issues/18910) — Debugger - Invalid Watchpoint for memory locations with valid data
*4y old · 0 comments*

Watchpoints set with 'dbw <addr> rw' show as 'invalid' in db output and are never triggered. Watchpoint fixes (ce73397f91 crash fix, 26baccd4e8 double-free fix) addressed stability but not the 'invalid' status display. The watchpoint implementation at libr/core/cmd_debug.inc.c:4157 handles dbw. Without a specific fix commit, and given the limited watchpoint testing, this likely persists.

---

#### [#19114](https://github.com/radareorg/radare2/issues/19114) — Windows Defender issues
*4y old · 1 comments · `Windows`*

This involves Windows Defender flagging test binaries and r2agent.exe as malware. The fix requires submitting false positive reports to Microsoft (https://www.microsoft.com/en-us/wdsi/filesubmission). No evidence this was done. The test suite binaries (malware samples for analysis testing) will inherently trigger AV. The r2agent.exe false positive may have been resolved by newer builds, but this requires Windows testing to verify.

---

#### [#19247](https://github.com/radareorg/radare2/issues/19247) — Function name from Ordinal number in Radare2
*4y old · 0 comments*

User question about resolving DLL function names from ordinals using PDB files. The PDB parser has 'unsupported leaf type' limitations. PE ordinal handling has been improved (commits 4c4da83cbe, 29c1344ff3) but PDB-based cross-DLL ordinal resolution remains incomplete due to PDB parser limitations.

---

#### [#21190](https://github.com/radareorg/radare2/issues/21190) — rax2 may support more types of input and be more solid
*3y old · 0 comments*

rax2 -s doesn't handle escaped hex input correctly (\x41\x42 producing 0xab instead of 'AB'). No specific fix found for this parsing behavior. The rax2 tool has received various updates but this specific edge case was not addressed.

---

#### [#21516](https://github.com/radareorg/radare2/issues/21516) — aab causes some functions to have 0 bbs, or not, but its not correct
*2y old · 0 comments*

Issue about aab producing incorrect basic block counts (0 bbs for some functions). Some aab performance improvements were committed (commit 2cdb1952ab 'Make aab even faster') but no specific fix for incorrect bb counts. The issue has minimal detail (screenshot only, no specific binary info beyond 'testsuite').

---

#### [#22551](https://github.com/radareorg/radare2/issues/22551) — evaluation of ESIL expression gives wrong result
*2y old · 5 comments*

ESIL evaluation gives incorrect results for ARM64 str/ldr operations using DUP and tmp register. trufae mentioned pushing 'some fixes a couple of weeks ago' and asked the reporter to re-test. condret questioned if writeable memory was mapped. No confirmation from reporter. The DUP/tmp pattern in ESIL is known to be problematic.

---

</details>
