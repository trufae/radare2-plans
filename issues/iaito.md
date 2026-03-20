# iaito Open Issues Review

*Generated 2026-03-18 — 20 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 4 | 20% |
| 🗑️ obsolete | 1 | 5% |
| 🔧 still_open | 15 | 75% |
| **Total** | **20** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🟢 5 | 2 | 0 | 2 |
| 🔵 4 | 1 | 1 | 2 |
| 🟡 3 | 1 | 0 | 1 |

## ✅ Likely Resolved (4)

### Confidence 🟢 5 (2)

<details>
<summary>Click to expand 2 issues</summary>

#### [#153](https://github.com/radareorg/iaito/issues/153) — Iaito show string "%#010I64x" instead addresses
*2y old · 5 comments*

Windows-specific bug where format strings were displayed instead of formatted addresses. A blind fix was applied by the maintainer, and two users independently confirmed it works correctly in the fixed build. One user showed before/after screenshots proving addresses now display properly.

---

#### [#267](https://github.com/radareorg/iaito/issues/267) — Unable to rename function (x86 DOS executable)
*1mo old · 0 comments*

The exact bug was that getThingUsedHere() only checked for the 'offset' JSON key from the 'anj' command, but r2 returns 'addr' instead. The current source code in DisassemblyContextMenu.cpp (line 441-444) now checks both 'offset' and 'addr' keys, with a fallback: 'if (obj.contains("offset")) { offset = obj["offset"]... } else if (obj.contains("addr")) { offset = obj["addr"]... }'. This directly fixes the reported issue.

---

</details>

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#191](https://github.com/radareorg/iaito/issues/191) — Insert input data in debugger console, putting data from outside tools like "echo" or "python print"
*1y old · 3 comments*

The maintainer submitted PR #192 adding rarun2 profile support for specifying program input. The DebugActions.cpp file references rarun2/rr2 functionality. The feature was implemented quickly after the request and merged.

---

</details>

### Confidence 🟡 3 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#256](https://github.com/radareorg/iaito/issues/256) — Crash (SIGABRT) after trying to load a PE32 i386 file
*3mo old · 2 comments*

The maintainer mentioned fixing 'a couple of relevant bugs' and asked the reporter to try building from git. The reporter used a Debian package with mismatched versions (iaito 6.0.4 labeled as 6.0.7 package). The maintainer could not reproduce. This was likely a version mismatch issue between iaito and radare2, and the relevant fixes were applied in git before the 6.0.8 release.

---

</details>

## 🗑️ Obsolete (1)

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#125](https://github.com/radareorg/iaito/issues/125) — Disassembly window showing x86 code when debugging ARM program
*3y old · 4 comments · `bug`*

The reporter was trying to debug an ARM binary on x86 Linux using qemu-user, which transparently JIT-compiles to x86. The maintainer correctly explained this is not possible through ptrace (qemu-user doesn't support it). The workaround is to use qemu-system or native ARM hardware. This is not an iaito bug but a fundamental limitation of the debugging approach.

---

</details>

## 🔧 Still Open (15)

### Confidence 🟢 5 (3)

<details>
<summary>Click to expand 3 issues</summary>

#### [#180](https://github.com/radareorg/iaito/issues/180) — Custom switch analysis panel
*1y old · 0 comments*

Feature request for a GUI panel to perform custom switch table analysis, similar to IDA's switch analysis dialog. No switch analysis widget exists in the iaito source code. Only generic thread/process widgets reference switch-related patterns.

---

#### [#211](https://github.com/radareorg/iaito/issues/211) — Highlighting respective Disassembly-Decompiler code for multiline selection
*1y old · 0 comments*

Feature request for bidirectional multi-line highlighting between disassembly and decompiler views. Currently single-line sync exists but multi-line selection does not propagate between views. No implementation found.

---

#### [#212](https://github.com/radareorg/iaito/issues/212) — vars/args/regs/mem values popups when mouse over
*1y old · 0 comments*

Feature request for IDA-like hover tooltips showing variable/register/memory values during debugging/emulation. This would require implementing QToolTip integration with r2's debug/emulation state. No implementation found in the codebase.

---

</details>

### Confidence 🔵 4 (6)

<details>
<summary>Click to expand 6 issues</summary>

#### [#149](https://github.com/radareorg/iaito/issues/149) — Add full proper terminal widget
*2y old · 0 comments*

Feature request to integrate QVTerminal (a Qt terminal widget) for a proper terminal experience within iaito. The ConsoleWidget.h exists but does not use QVTerminal. No integration with the referenced project has been done.

---

#### [#181](https://github.com/radareorg/iaito/issues/181) — New panel to manage files/maps
*1y old · 1 comments*

Feature request for an IO map management panel similar to rwx GUI or Ghidra's memory map view. Would allow creating/managing memory maps, IO banks, and file mappings. No dedicated map management widget was found in the iaito source.

---

#### [#188](https://github.com/radareorg/iaito/issues/188) — WebServer support
*1y old · 0 comments*

Feature request for web server settings panel (sandbox, HTTP port, bind address) and server toggle. No webserver-related widgets or settings were found in the iaito source beyond some Makefile references. This remains unimplemented.

---

#### [#216](https://github.com/radareorg/iaito/issues/216) — debug mode with custom IO-debug plugin
*1y old · 1 comments*

Using a custom Python IO debug plugin works with r2 command line but fails when launching iaito from within r2 in debug mode. The error 'File does not have executable permissions' suggests iaito's debug launcher doesn't properly handle custom IO URI schemes. No fix found.

---

#### [#217](https://github.com/radareorg/iaito/issues/217) — add command to add new menu entries
*1y old · 0 comments*

Feature request from the maintainer to allow core plugins to add custom menu entries to iaito's GUI. No implementation found. This would require a plugin API extension for menu registration.

---

#### [#220](https://github.com/radareorg/iaito/issues/220) — select range+data conversion only changes 1 word
*11mo old · 0 comments*

When selecting a range in the disassembly view and pressing 'd' to rotate data types, only the last cursor position is changed rather than the entire selection. This is a UI behavior bug filed by the maintainer (trufae). No fix was found in the codebase.

---

</details>

### Confidence 🟡 3 (6)

<details>
<summary>Click to expand 6 issues</summary>

#### [#68](https://github.com/radareorg/iaito/issues/68) — Debug not working
*4y old · 6 comments*

Long-running issue about debugging not working, originally due to r2/iaito version mismatches and thread safety issues. The maintainer acknowledged iaito needs thread-safe r2 usage and planned fixes for r2-5.7.0. A user reported the issue persisting on iaito 5.9.6 in October 2024. The fundamental thread safety improvements are ongoing but debugging remains unreliable for many users.

---

#### [#120](https://github.com/radareorg/iaito/issues/120) — auto refresh content
*3y old · 3 comments · `bug`*

When a new file is mapped via r2 commands, the disassembly view stays stale until manually refreshed via View->Refresh Contents. The maintainer noted it's a one-line fix and asked for reproduction steps and a patch, but no follow-up was provided. The auto-refresh on IO map changes has not been implemented.

---

#### [#122](https://github.com/radareorg/iaito/issues/122) — Open project when passing a project directory as argument
*3y old · 2 comments · `enhancement`*

Feature request to open r2 project files directly from the command line. The maintainer mentioned using the Pz command from r2-5.8.4 for project zip support. SaveProjectDialog.ui exists but no command-line project loading was found. The r2 project system has evolved but iaito's integration remains incomplete.

---

#### [#172](https://github.com/radareorg/iaito/issues/172) — Debugging hangs
*1y old · 4 comments*

Debug button click causes the UI to hang with an unresponsive dialog that cannot be cancelled. Multiple users confirmed the issue across different versions (5.9.2, 5.9.6) and distributions (Manjaro, Ubuntu). The underlying cause is likely related to r2's thread safety issues that the maintainer has been working on. No definitive fix found.

---

#### [#186](https://github.com/radareorg/iaito/issues/186) — show file, timestamp instead of project name twice
*1y old · 0 comments*

UI bug where the project name is shown twice instead of showing file path and timestamp. The screenshot shows the issue clearly. No specific fix was identified in the source code, though the issue was updated in January 2025 suggesting some activity.

---

#### [#255](https://github.com/radareorg/iaito/issues/255) — Discord link (https://discord.gg/6RwDEJFr) is expired or invalid
*3mo old · 6 comments*

The Discord invite links keep expiring. The maintainer maintains a redirect at https://radare.org/discord but even that redirected to an expired link. A contributor created a 'never expires' link but the maintainer was skeptical it would last. This is a recurring operational issue with Discord invite management, not a code bug.

---

</details>
