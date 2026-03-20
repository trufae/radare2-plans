# r4ghidra Open Issues Review

*Generated 2026-03-18 — 8 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 0 | 0% |
| 🗑️ obsolete | 1 | 12% |
| 🔧 still_open | 7 | 87% |
| **Total** | **8** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🔵 4 | 0 | 1 | 1 |

## 🗑️ Obsolete (1)

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#1](https://github.com/radareorg/r4ghidra/issues/1) — do an r2-to-ghidra exporter/importer based on the ida pro plugin
*7y old · 0 comments*

This issue is from 2019 and references an IDA Pro plugin ZIP on ghidra.re. The r4ghidra project has since evolved into a full Ghidra-to-r2 bridge with REPL, JS scripting, decompilation, and analysis commands. The original request to base an exporter/importer on the IDA Pro plugin is obsolete as the project took a different architectural direction. The linked ZIP URL is likely dead. No activity in over 6 years.

---

</details>

## 🔧 Still Open (7)

### Confidence 🔵 4 (6)

<details>
<summary>Click to expand 6 issues</summary>

#### [#4](https://github.com/radareorg/r4ghidra/issues/4) — Implement session registration for radare2
*1y old · 0 comments · `enhancement`*

The issue requests implementing the =l? command by creating session files. Searching the codebase for 'session' or '=l' finds no relevant implementation. The R4GhidraState and R2REPLImpl do not contain session registration logic. This feature enhancement has not been addressed.

---

#### [#8](https://github.com/radareorg/r4ghidra/issues/8) — install dependabot
*8mo old · 0 comments*

No dependabot configuration was found in the repository. No .github/dependabot.yml file exists. The issue also mentions updating the Ghidra CI action for API limits, which also does not appear to have been addressed.

---

#### [#16](https://github.com/radareorg/r4ghidra/issues/16) — Enable r2pipe shell
*8mo old · 0 comments · `enhancement`*

Searching for 'r2pipe' or 'pipe' in the Java source shows only R2JsCommandHandler and R2REPLImpl references, neither of which implements spawning r2 instances from Ghidra for r2js scripting. This feature enhancement to combine analysis results from both sides has not been implemented.

---

#### [#17](https://github.com/radareorg/r4ghidra/issues/17) — Running instance
*8mo old · 0 comments · `enhancement`*

The request is to register r4ghidra as a running instance so r2 can find it without passing the URL. No session/instance registration code was found in the codebase. This is closely related to issue #4 (session registration) and neither has been implemented.

---

#### [#20](https://github.com/radareorg/r4ghidra/issues/20) — Unsafe JSON escaping
*7mo old · 1 comments*

The issue points to R2JsCommandHandler.java line 159 for unsafe JSON escaping. The current code at line 156 still uses simple string replace for quotes: ((String) obj).replace("\"", "\\\""). This is indeed inadequate as it does not handle other special characters (newlines, tabs, backslashes, unicode). The maintainer acknowledged the issue and asked for a fix, but none has been submitted.

---

#### [#23](https://github.com/radareorg/r4ghidra/issues/23) — Document settings
*7mo old · 0 comments*

The issue requests moving settings documentation from a comment in R2EvalCommandHandler to a more accessible location. The R2EvalCommandHandler file exists and contains inline documentation but there is no external settings documentation file, README section, or wiki page covering the available settings. The repo README does not document configuration options.

---

</details>

### Confidence 🟡 3 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#14](https://github.com/radareorg/r4ghidra/issues/14) — decouple the code from ghidra
*8mo old · 1 comments · `enhancement`*

While R4GhidraState does reference FlatProgramAPI and some abstraction exists, the codebase still directly uses Ghidra types extensively throughout command handlers. The decoupling requested (replaceable mock API for testing/CLI) has not been fully implemented. There is no mock or test framework in place. The v-p-b comment from July 2025 outlines approaches but no PR followed.

---

</details>
