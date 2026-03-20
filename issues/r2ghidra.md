# r2ghidra Open Issues Review

*Generated 2026-03-18 — 20 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 5 | 25% |
| 🗑️ obsolete | 1 | 5% |
| 🔧 still_open | 14 | 70% |
| **Total** | **20** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🔵 4 | 4 | 1 | 5 |
| 🟡 3 | 1 | 0 | 1 |

## ✅ Likely Resolved (5)

### Confidence 🔵 4 (4)

<details>
<summary>Click to expand 4 issues</summary>

#### [#94](https://github.com/radareorg/r2ghidra/issues/94) — Build the sleigh files with meson
*3y old · 0 comments*

The meson.build now includes sleighc_exe from the ghidra subproject (line 131: 'sleighc_exe = ghidra.get_variable("sleighc_exe")'), indicating that sleigh file compilation has been integrated into the meson build system. This addresses the original request to build sleigh files with meson instead of requiring separate make commands.

---

#### [#99](https://github.com/radareorg/r2ghidra/issues/99) — Imstallation issue in termux
*3y old · 8 comments*

The installation issue was due to missing build dependencies (g++, wget, git, patch). The solution was provided in comments: 'pkg install build-essential binutils wget git' then 'r2pm -ci r2ghidra'. This is a user configuration issue, not a code bug. The instructions have been shared and the package manager approach works.

---

#### [#130](https://github.com/radareorg/r2ghidra/issues/130) — Why can r2ghidra not output the same code as ghidra in cutter
*2y old · 14 comments*

The r2ghidra 6.0.0 release significantly improved decompiler output quality. The collaborator satk0 demonstrated that the output now matches both rz-ghidra and ghidra for the test binary provided. A user confirmed 'It's much better now.' The ghidra backend was updated, local variable analysis improved, and the output quality gap was substantially closed. Minor issues like missing space after comma were noted as remaining.

---

#### [#154](https://github.com/radareorg/r2ghidra/issues/154) — W64 release of 5.9.8 is empty
*11mo old · 16 comments*

The Windows build was restored with radare2 6.0.0 release, as confirmed by the collaborator satk0 in August 2025: 'with radare2 6.0.0 release we've managed to restore windows support for r2g.' The empty zip issue for 5.9.8 is moot now that newer releases exist. However, some remaining tasks for meson Windows builds were noted (building analysis/asm libraries, installing libraries).

---

</details>

### Confidence 🟡 3 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#133](https://github.com/radareorg/r2ghidra/issues/133) — 5.9.0 Entry point not found
*1y old · 1 comments*

This was reported for r2ghidra 5.9.0 on Windows, with a follow-up noting 5.9.4 hangs with EXCEPTION_ACCESS_VIOLATION. Since Windows support was reworked for 6.0.0 with significant fixes, this specific version-locked issue is likely resolved in current releases, though Windows stability remains an ongoing concern.

---

</details>

## 🗑️ Obsolete (1)

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#58](https://github.com/radareorg/r2ghidra/issues/58) — Build failure when compiling x86-64.slaspec
*4y old · 16 comments*

This 2021 issue was specific to building from tarballs on a musl-based Linux distro (Source Mage) with GCC 10.3.0. The maintainer could not reproduce on Alpine (also musl). The ghidra-native code and sleighc have been significantly updated since then (ghidra backend updated for 6.0.0). The specific version combination (r2ghidra 5.5.2, ghidra-native 0.1.3) is long obsolete.

---

</details>

## 🔧 Still Open (14)

### Confidence 🔵 4 (5)

<details>
<summary>Click to expand 5 issues</summary>

#### [#41](https://github.com/radareorg/r2ghidra/issues/41) — .fidb File Support
*4y old · 1 comments*

Ghidra's Function ID Database (.fidb) files are not supported in r2ghidra. The maintainer noted this should be implemented in r2 itself (as signature-based symbol detection) rather than in the decompiler plugin. No fidb support has been added to either r2 or r2ghidra.

---

#### [#106](https://github.com/radareorg/r2ghidra/issues/106) — Allow using system pugixml for build
*3y old · 4 comments*

The meson.build still bundles pugixml as a subproject rather than allowing system pugixml. A meson wrap file exists (subprojects/pugixml.wrap) but the maintainer explicitly stated preference for using r2's XML parser instead (issue #138). Neither approach has been fully implemented - system pugixml is not an option, and r2's XML parser hasn't replaced pugixml yet.

---

#### [#138](https://github.com/radareorg/r2ghidra/issues/138) — Remove pugixml dependency
*1y old · 0 comments*

pugixml is still used in the codebase. CodeXMLParse.cpp includes pugixml headers, and the build system has pugixml as a subproject (subprojects/pugixml.wrap, subprojects/pugixml.mk). The goal was to replace it with r2's own XML APIs, but this has not been done.

---

#### [#150](https://github.com/radareorg/r2ghidra/issues/150) — MIPS32 function resolution (loc._gp)
*1y old · 1 comments*

This is a known Ghidra limitation for MIPS global pointer-relative function resolution. The maintainer acknowledged it as a ghidra backend issue and suggested using decai (LLM-based decompiler) as a workaround. The underlying ghidra C++ code has not been updated to handle this case, and r2ghidra cannot easily fix upstream ghidra limitations.

---

#### [#192](https://github.com/radareorg/r2ghidra/issues/192) — W64 release of 6.0.4 is missing
*4mo old · 5 comments*

The latest comment from March 2026 shows the issue persists for 6.1.0. The maintainer asked for relevant logs but the Windows 64-bit release remains problematic. The Windows build pipeline has had chronic issues with r2ghidra.

---

</details>

### Confidence 🟡 3 (9)

<details>
<summary>Click to expand 9 issues</summary>

#### [#52](https://github.com/radareorg/r2ghidra/issues/52) — x86 - missing strings in decompilation
*4y old · 6 comments*

String resolution fails for x86 binaries, showing raw addresses (0x2000) instead of string literals in decompiler output. The roprop config variable can help but doesn't fully resolve it. The issue is related to how ghidra handles low virtual addresses that overlap with physical addresses. A high base address workaround was suggested. The underlying issue in string/constant resolution persists.

---

#### [#55](https://github.com/radareorg/r2ghidra/issues/55) — Topic: types for variables, flags, and function signatures
*4y old · 0 comments*

This is a design discussion about implementing a proper type system within r2ghidra independent of radare2's type system. The argument is that ghidra's type handling (including IR-only variables that cannot be mapped to r2) warrants a separate type system in the plugin. This remains an open architectural discussion with no resolution. The r2 type system is being rewritten but the r2ghidra-specific type handling hasn't been independently developed.

---

#### [#112](https://github.com/radareorg/r2ghidra/issues/112) — RISC-V on windows
*3y old · 14 comments*

Originally about RISC-V support (which works fine on non-Windows), this became a Windows-specific heap corruption issue. The maintainer confirmed it works on Windows 11 but the reporter had issues on Windows 10. The core Windows heap corruption problems were partially addressed in 6.0.0 but Windows remains the least stable platform for r2ghidra.

---

#### [#123](https://github.com/radareorg/r2ghidra/issues/123) — Different behavior when opening local vs remote files.
*2y old · 3 comments*

String literal resolution differs between local file and RAP (remote) access modes. The reporter found a 10 MiB .data section size limit for string recognition. This is likely related to how r2ghidra reads memory through the r2 IO layer vs direct file access, and the size limits on data section scanning. No specific fix found.

---

#### [#134](https://github.com/radareorg/r2ghidra/issues/134) — operation result dereferenced by memory address
*1y old · 0 comments*

Ghidra decompiler showing '*0x2068' (memory dereference) instead of the actual computed constant value. This is related to how the decompiler handles read-only data section references. The roprop config variable exists to control read-only propagation levels (0-4), which may help in some cases, but the underlying issue in the ghidra backend persists.

---

#### [#135](https://github.com/radareorg/r2ghidra/issues/135) — SBORROW4 wrongly used in condition
*1y old · 0 comments*

This is a Ghidra decompiler bug where SBORROW4 (signed borrow) is incorrectly used in loop conditions, making the decompiled condition always false when the original code works correctly. This is an upstream ghidra decompiler issue in its intermediate representation simplification. No specific fix found.

---

#### [#136](https://github.com/radareorg/r2ghidra/issues/136) — type casted value recovered as 0
*1y old · 0 comments*

This is a Ghidra decompiler backend bug where type-casted constant values (like (int)(9876543)) are incorrectly recovered as 0. This is likely an issue in the ghidra native decompiler code. The ghidra backend has been updated in 6.0.0 but it's unclear if this specific constant folding bug was fixed. No direct fix found in r2ghidra source.

---

#### [#141](https://github.com/radareorg/r2ghidra/issues/141) — Problem regarding r2ghidra installation on Windows
*1y old · 11 comments*

This long-running Windows issue involves heap corruption crashes when loading the plugin. The root cause was identified as a free() issue with std::string taking ownership of strdup'd memory on Windows. While a partial fix was applied and Windows builds were restored for 6.0.0, multiple users continued reporting issues on different versions (5.9.4, 5.9.6). The maintainer suggested consolidating all Windows issues into one ticket. The fundamental Windows compatibility remains fragile.

---

#### [#166](https://github.com/radareorg/r2ghidra/issues/166) — The names of called functions are not resolved if `aa` is not invoked prior to invoking `pdg`
*8mo old · 0 comments*

This is about function name resolution in the decompiler output for vmlinux targets. The decompiler shows raw addresses like func_0xffffffff... instead of resolved symbol names when analysis (aa/aaa) has not been run first. This is somewhat by design since r2ghidra relies on r2's analysis data, but could be improved with automatic analysis or better symbol resolution. No specific fix found.

---

</details>
