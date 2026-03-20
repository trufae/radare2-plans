# radare2-book Open Issues Review

*Generated 2026-03-18 — 38 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 10 | 26% |
| 🗑️ obsolete | 2 | 5% |
| 🔧 still_open | 26 | 68% |
| **Total** | **38** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🔵 4 | 1 | 1 | 2 |
| 🟡 3 | 8 | 1 | 9 |
| 🟠 2 | 1 | 0 | 1 |

## ✅ Likely Resolved (10)

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#218](https://github.com/radareorg/radare2-book/issues/218) — Add documentation about zooming
*6y old · 0 comments · `bug` `enhancement` `chapter` `paragraph`*

The commandline/zoom.md file exists and provides comprehensive documentation about the zoom feature including the `pz` command, `Z` visual mode toggle, zoom configuration variables (zoom.maxsz, zoom.from, zoom.to, zoom.byte), various zoom byte modes (entropy, printable chars, flags, etc.), and examples. The visual mode chapter is referenced in SUMMARY.md. This issue appears well-addressed.

---

</details>

### Confidence 🟡 3 (8)

<details>
<summary>Click to expand 8 issues</summary>

#### [#237](https://github.com/radareorg/radare2-book/issues/237) — Document `tpv` and `@v`
*6y old · 4 comments · `paragraph` `examples`*

The `tpv` command is documented in analysis/types.md (line 31: `tpv <type> @ [value] - Show offset formatted for given type`) and in commandline/types.md. However, the `@v:` modifier (value modifier for temporary offset override) does not appear to be documented anywhere in the book. The issue is partially addressed - tpv is listed but @v is missing.

---

#### [#220](https://github.com/radareorg/radare2-book/issues/220) — Document more about the analysis graphs
*6y old · 0 comments · `enhancement` `chapter` `examples`*

The analysis/graphs.md file now contains extensive documentation of graph commands including `agc` (function callgraph), `agC` (global callgraph), and various graph types and output formats. The `axg` command for finding paths between functions is documented in analysis/code_analysis.md. The issue asked for callgraph between functions and finding paths - both are now covered, though more examples could always be added.

---

#### [#163](https://github.com/radareorg/radare2-book/issues/163) — More graph commands and examples
*7y old · 0 comments · `paragraph` `examples` `illustration`*

The analysis/graphs.md now contains comprehensive documentation of graph commands including all `ag` variants. The `axg` and `axg*` commands are documented in code_analysis.md with examples. The issue specifically requested `axg*` documentation and more examples/illustrations. The commands are now documented, though illustrations are not present (the book is text-based markdown).

---

#### [#162](https://github.com/radareorg/radare2-book/issues/162) — Fix r2lang plugins documentation after radare2 refactoring of RBin and RAnal+RAsm merge
*7y old · 0 comments*

The plugins section has been substantially updated. The plugins/python.md shows r2lang plugin examples that reference the merged architecture (r_arch). The ragg2 lang.md mentions 'r_arch interface (r_asm and r_anal merged into r_arch)'. The plugin documentation appears to reflect the post-merge state of radare2, suggesting this refactoring has been accounted for.

---

#### [#160](https://github.com/radareorg/radare2-book/issues/160) — Add "Analysis by default" in the book
*7y old · 0 comments · `chapter`*

The analysis/code_analysis.md contains discussion of default analysis behavior, the `aa`/`aaa`/`aaaa` command sequence, and when to use different analysis depths. While it may not directly copy the blog post content, the concept of analysis-by-default and the different analysis levels are covered in the code analysis chapter.

---

#### [#177](https://github.com/radareorg/radare2-book/issues/177) — Add a installation for Android/iOS
*7y old · 0 comments · `paragraph` `examples`*

The install/android.md file exists with detailed Termux-based installation instructions for Android. However, there is no iOS installation guide. The issue requested both Android and iOS. Android is covered; iOS remains missing. Partially resolved.

---

#### [#140](https://github.com/radareorg/radare2-book/issues/140) — Add info on Syscalls in r2book
*7y old · 10 comments · `chapter`*

The analysis/syscalls.md file now exists with substantial content covering: syscall search with `/c` and `/as`, `asl` for listing syscalls, `asm.emu` for emulated syscall display, `aae`/`aaaa` for flagspace integration, `dcs`/`dcs*` for debugging syscalls, and `asl`/`asn` utilities. However, some items from the checklist remain unaddressed: `e emu.write`, emulation of syscalls, analysis of syscalls in depth, and how to add new syscall databases. Substantial progress but not fully complete per the original checklist.

---

#### [#115](https://github.com/radareorg/radare2-book/issues/115) — Add UI Section
*7y old · 0 comments · `section` `screenshots` `illustration`*

The book now has an intro/ui.md section referenced in SUMMARY.md. Iaito (formerly Cutter) is mentioned with a download link. The install section covers Flatpak and Snap installation for iaito. The webserver is not extensively documented in a dedicated section, but the UI section exists. The issue also requested Android/iOS app info which is partially covered. Not fully comprehensive but the UI section exists.

---

</details>

### Confidence 🟠 2 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#185](https://github.com/radareorg/radare2-book/issues/185) — Missing new commands
*7y old · 0 comments · `enhancement` `chapter` `paragraph` `examples`*

The issue lists several commands: c1, vbg, vbz, aao, aaF, aflm, /ci. Checking the book: `c1` is documented in comparing_bytes.md, `aflm` is mentioned in code_analysis.md. However, `vbg`, `vbz`, `aao`, `aaF`, and `/ci` do not appear to be documented. Only partial coverage achieved. Many of these commands may also have changed or been removed in newer radare2 versions, making this partially obsolete.

---

</details>

## 🗑️ Obsolete (2)

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#424](https://github.com/radareorg/radare2-book/issues/424) — On the ARMV8 architecture, how to print floating-point registers
*10mo old · 0 comments*

This is a support question about using radare2, not a documentation issue for the book. The user asks how to use `dr s0` to print floating-point registers on ARMv8. This belongs as a radare2 core issue or on a support forum, not as a book issue. The book already documents registers in the debugger/registers section.

---

</details>

### Confidence 🟡 3 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#165](https://github.com/radareorg/radare2-book/issues/165) — Analysis commands changes
*7y old · 0 comments · `good first issue`*

This issue from 2018 references radare2 issue #12393 about analysis command changes. The book's analysis section has been significantly rewritten since then. The referenced issue is from the old 'radare' org (not radareorg). Given the age (7+ years) and the significant evolution of both radare2 and the book, these specific command changes have likely been superseded by multiple subsequent changes.

---

</details>

## 🔧 Still Open (26)

### Confidence 🟢 5 (5)

<details>
<summary>Click to expand 5 issues</summary>

#### [#400](https://github.com/radareorg/radare2-book/issues/400) — Add a sumy job to sumarize the book for r2ai
*1y old · 0 comments*

No evidence of a sumy integration anywhere in the repository. No Makefile target, no CI job, no reference to the sumy tool. The feature request remains completely unaddressed.

---

#### [#241](https://github.com/radareorg/radare2-book/issues/241) — Add a links validator in the CI
*5y old · 0 comments*

No evidence of a link validator in CI. No references to remark-lint, mdbook-linkcheck, or any link validation tool in the repository. The CI configuration does not include link checking.

---

#### [#197](https://github.com/radareorg/radare2-book/issues/197) — Add a "control questions" at the end of the chapters.
*6y old · 0 comments · `enhancement`*

No evidence of control questions or exercises at the end of any chapter in the book. This pedagogical feature has not been implemented.

---

#### [#105](https://github.com/radareorg/radare2-book/issues/105) — Put all used example binaries in the "examples/"
*7y old · 0 comments · `examples` `refactor`*

No `examples/` directory exists in the repository root. Example binaries referenced throughout the book are not centralized in a dedicated directory for reproducibility.

---

#### [#76](https://github.com/radareorg/radare2-book/issues/76) — Convert book into "literate radare"
*8y old · 6 comments · `enhancement`*

No evidence of a literate programming system, documentation test runner, or automated code example verification in the repository. The code examples in the book remain static markdown blocks with no mechanism to run or verify them. The PoC mentioned in comments was external (a gist) and was never integrated.

---

</details>

### Confidence 🔵 4 (11)

<details>
<summary>Click to expand 11 issues</summary>

#### [#423](https://github.com/radareorg/radare2-book/issues/423) — Add system-specific sections
*11mo old · 0 comments*

The issue requests adding Windows, iOS, and Android system-specific sections. The book's SUMMARY.md has no dedicated system-specific sections for Windows, iOS, or Android debugging/reversing. There is an install/android.md and install/windows.md for installation, and a debugger/windows_messages.md, but no comprehensive system-specific reversing sections as requested. All three checkboxes remain unchecked.

---

#### [#221](https://github.com/radareorg/radare2-book/issues/221) — Interactive ASCII/Unicode radiff2 usage
*6y old · 1 comments · `paragraph` `illustration`*

The radiff2 section covers data diffing, code diffing, headers diffing, and binary diffing, but there is no documentation of interactive ASCII/Unicode diff usage. The datadiff.md mentions `radiff2 -x` for hex+ascii format but not interactive usage as requested. No illustrations of interactive diff output exist.

---

#### [#188](https://github.com/radareorg/radare2-book/issues/188) — Document `tfc` command for calling conventions
*7y old · 1 comments · `chapter`*

The calling_conventions.md documents `afc` family of commands but does not mention `tfc` specifically. The types.md lists `tc` for C output format but not `tfc`. The calling convention manipulation description referenced in issue #166 has basic coverage but not the `tfc` command specifically.

---

#### [#179](https://github.com/radareorg/radare2-book/issues/179) — Document lldb integration
*7y old · 0 comments · `chapter`*

There is no mention of LLDB integration in the debugger section or anywhere else in the book. The debugger/migration.md file covers IDA, GDB, and WinDBG migration but not LLDB. The remote debugging section covers GDB and WinDbg protocols only.

---

#### [#176](https://github.com/radareorg/radare2-book/issues/176) — Document tracing features
*7y old · 1 comments · `chapter`*

The debugger section mentions `dt` (display instruction traces) in the command listing in intro.md, and `dbt` for backtrace. However, there is no dedicated tracing documentation covering `dt++`, `graph.trace`, trace-based analysis as described in the issue comments. The specific workflow of marking traced basic blocks and reflecting them in the graph is not documented.

---

#### [#157](https://github.com/radareorg/radare2-book/issues/157) — Document change on section `_end` and `$e`
*7y old · 1 comments · `paragraph`*

No mention of `_end` section or `$e` variable found in the expressions or any other chapter of the book. The referenced PR #16194 in radare2 changed section naming. This specific documentation about the `_end` section and `$e` evaluable variable has not been added.

---

#### [#148](https://github.com/radareorg/radare2-book/issues/148) — Document kernelcache and dyldcache in Symbols section
*7y old · 0 comments · `chapter`*

The only mentions of dyldcache in the book are in plugins/troubles.md (troubleshooting plugin loading) and first_steps/commandline_flags.md (the -X flag). There is no documentation of kernelcache or dyldcache usage in the Symbols section or anywhere else that explains how to work with these formats.

---

#### [#147](https://github.com/radareorg/radare2-book/issues/147) — Document `adf` and `adfg`
*7y old · 1 comments · `enhancement` `paragraph` `illustration`*

No documentation of `adf` or `adfg` commands found anywhere in the book. These commands for marking data references and finding gaps between basic blocks remain undocumented.

---

#### [#128](https://github.com/radareorg/radare2-book/issues/128) — Mention radare2ida migration scripts
*7y old · 0 comments · `chapter`*

The debugger/migration.md covers migration from IDA/GDB/WinDBG in terms of command mapping, but does not mention the radare2ida migration scripts repository or tools for importing/exporting data between radare2 and IDA.

---

#### [#35](https://github.com/radareorg/radare2-book/issues/35) — Add documentation for BITS esil command
*9y old · 0 comments*

No documentation of the BITS ESIL command (to change core->bits) found in the emulation section or anywhere else in the book. The ESIL documentation covers many operations but not the BITS command specifically.

---

#### [#19](https://github.com/radareorg/radare2-book/issues/19) — dsi conditionnals
*10y old · 0 comments*

The `dsi` command appears only in the debugger/migration.md comparison table (as 'How to step until condition is true') but is not documented with its conditional syntax, usage, or examples anywhere in the book. No dedicated documentation exists.

---

</details>

### Confidence 🟡 3 (10)

<details>
<summary>Click to expand 10 issues</summary>

#### [#278](https://github.com/radareorg/radare2-book/issues/278) — Printing file headers
*5y old · 1 comments · `paragraph` `good first issue`*

The issue requests documenting the `pfo` and `pf.elf_*` workflow for printing file headers in the print_modes chapter. The print_modes.md file mentions `pf.` format definitions but does not include the specific ELF header parsing workflow described in the issue (using `pfo elf64`, `pf.elf_ident`, `pf.elf_header`). The detailed example from the issue has not been added to the book.

---

#### [#240](https://github.com/radareorg/radare2-book/issues/240) — More explanations on calling conventions
*5y old · 0 comments · `chapter`*

The book has an analysis/calling_conventions.md file that documents `afc` commands and basic calling convention concepts. However, it lacks the detailed explanations of `fastcall`, `thiscall`, etc. with examples and radare2 commands as requested. The existing content covers the command interface but not the calling convention details themselves.

---

#### [#238](https://github.com/radareorg/radare2-book/issues/238) — Fix and improve Plugins and Python sections
*6y old · 0 comments*

The plugins/python.md file exists and documents r2lang Python plugins with examples for asm, anal, and bin plugins. The plugins section has been expanded with multiple plugin types (IO, Asm, Analysis, Bin, Charset, R2JS, Python, r2ai). Some improvement has been made but the issue is vague (filed with template only, no specifics). Partial progress but likely still room for improvement.

---

#### [#227](https://github.com/radareorg/radare2-book/issues/227) — Bindings introduction
*6y old · 1 comments · `enhancement` `section` `examples`*

The scripting section covers r2pipe and rlang, and the plugins section covers Python plugins. However, there is no dedicated 'Bindings introduction' section that explains the three kinds of bindings (r2pipe, rlang, API bindings) as described in the issue body. The valabind/radare2-bindings approach and r2pipe-api are not mentioned. The content from the issue description has not been incorporated as a cohesive introduction.

---

#### [#166](https://github.com/radareorg/radare2-book/issues/166) — Improve types C output commands description
*7y old · 1 comments · `chapter` `examples` `refactor`*

The analysis/types.md documents `tc` command for listing types in C output format and covers type-related commands. The calling_conventions.md covers `afc` commands. However, `tsc` is not documented, and the calling convention manipulation description could use more examples as noted in the issue comments. Partially addressed but the requested improvements to C output commands and calling convention descriptions are incomplete.

---

#### [#156](https://github.com/radareorg/radare2-book/issues/156) — Better chapter on evaluable variables
*7y old · 0 comments · `chapter`*

The config/evars.md file exists (183 lines) and documents common configuration variables with some use cases. However, it is relatively brief and does not cover all variables or provide common use cases for each as the issue requests. The chapter exists but could use significant expansion to meet the request for explaining 'possible or common use cases' for each variable.

---

#### [#153](https://github.com/radareorg/radare2-book/issues/153) — More information on Python r2lang plugins
*7y old · 0 comments · `enhancement` `chapter` `examples`*

The plugins/python.md documents r2lang Python plugins with examples for asm, anal, and bin plugins. However, the issue requests thorough function explanations with examples and complex API explanations (e.g. for relocations). The current documentation provides basic plugin scaffolding but lacks the in-depth API coverage and relocation examples requested.

---

#### [#136](https://github.com/radareorg/radare2-book/issues/136) — Add ROP search chapter
*7y old · 0 comments · `chapter`*

The `/R` command for ROP gadgets is listed in the search command reference (search/intro.md and refcard/intro.md) and mentioned in rasm2/disassemble.md and debugger/memory_maps.md. However, there is no dedicated ROP search chapter with examples, workflows, and explanations as requested. Only brief mentions exist.

---

#### [#116](https://github.com/radareorg/radare2-book/issues/116) — How to create tiny bins?
*7y old · 0 comments · `enhancement`*

The ragg2 section mentions creating 'tiny binaries' and the ragg2 tool compiles to tiny binaries. However, `rabin2 -C` for creating tiny bins is not documented. The specific workflow of creating minimal/tiny binaries using rabin2 is not covered.

---

#### [#75](https://github.com/radareorg/radare2-book/issues/75) — Add documentation on forensics
*8y old · 0 comments · `section`*

The rahash2 section covers block-based hashing with forensic use cases (`rahash2 -B -b 1M -a sha256`). The search section documents `/m` for magic file search and `/M` for filesystem mounting. The visual mode reference mentions `M` for walking mounted filesystems. However, there is no dedicated forensics chapter or section that ties these together as a cohesive forensics workflow. The individual commands are scattered across different sections.

---

</details>
