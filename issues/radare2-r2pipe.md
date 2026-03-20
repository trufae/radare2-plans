# radare2-r2pipe Open Issues Review

*Generated 2026-03-18 — 15 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 5 | 33% |
| 🗑️ obsolete | 1 | 6% |
| 🔧 still_open | 9 | 60% |
| **Total** | **15** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🔵 4 | 2 | 1 | 3 |
| 🟡 3 | 3 | 0 | 3 |

## ✅ Likely Resolved (5)

### Confidence 🔵 4 (2)

<details>
<summary>Click to expand 2 issues</summary>

#### [#4](https://github.com/radareorg/radare2-r2pipe/issues/4) — Modernize r2pipe.js to use more ES6
*9y old · 2 comments · `nodejs`*

Request to use let/const, arrow functions, and modern JS features. The current index.js uses 'const' for require statements and 'let' in functions. Arrow functions are used in examples. The codebase has been largely modernized. Some 'var' usage may remain but the main modernization goals (let/const, arrow functions) have been achieved.

---

#### [#64](https://github.com/radareorg/radare2-r2pipe/issues/64) — TypeError: a bytes-like object is required, not 'str'
*7y old · 2 comments*

Python 3 compatibility issue where strings needed to be encoded to bytes for os.write(). The current open_sync.py properly encodes strings before writing: 'self.process.stdin.write((cmd + "\n").encode("utf8"))' at line 140. The fix suggested in the comments (using .encode()) has been applied in the current codebase.

---

</details>

### Confidence 🟡 3 (3)

<details>
<summary>Click to expand 3 issues</summary>

#### [#14](https://github.com/radareorg/radare2-r2pipe/issues/14) — R2Pipe Python can't load project files without binary
*9y old · 1 comments*

Request to support opening r2 with project files. The current open_base.py has normalize_open_target() which handles file_args, and open_sync.py passes additional arguments to the r2 process via Popen. Users can pass flags via the second argument to open(). However, it's unclear if project files specifically work without a binary argument.

---

#### [#70](https://github.com/radareorg/radare2-r2pipe/issues/70) — r2pipe python-async seems much slower than previous sync version
*7y old · 0 comments*

Report from 2018 about the async r2pipe being 300x slower than the sync version. The codebase now has separate open_sync.py and open_async.py modules, with open_sync.py being the default for Python 3. The sync version is available and used by default, so users are no longer forced into the async path. The issue was about the forced migration to async; the separation into sync/async modules likely resolves the performance concern.

---

#### [#148](https://github.com/radareorg/radare2-r2pipe/issues/148) — ioplugin and map file
*3y old · 3 comments*

User question about mapping virtual files with r2pipe. The maintainer (trufae) confirmed it is possible and explained how to use the 2nd argument of open() to pass arguments to the r2 process. This was a usage question rather than a bug. The user was given a working solution. The functionality exists.

---

</details>

## 🗑️ Obsolete (1)

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#5](https://github.com/radareorg/radare2-r2pipe/issues/5) — Fix the process.nextTick issue
*9y old · 0 comments · `nodejs`*

References a 2016 article about process.nextTick in Node.js. The Node.js r2pipe code has been significantly updated since then, using modern patterns (const/let, arrow functions). The specific nextTick issue from Node.js v4 era is no longer relevant to modern Node.js versions. The issue is 10 years old with no specifics on what actually breaks.

---

</details>

## 🔧 Still Open (9)

### Confidence 🟢 5 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#76](https://github.com/radareorg/radare2-r2pipe/issues/76) — Not working examples of using r2 via web interface
*7y old · 3 comments*

Examples reference cloud.radare.org and cloud.rada.re which are dead URLs. Searching the current codebase reveals these URLs still appear in many files: swift/main.swift, python/examples/batch.py, java examples, nodejs examples, dotnet examples, nim examples, r2core-js, and more. The URLs were never updated or removed despite explicit agreement to do so. At least 19 files still reference these non-functional URLs.

---

</details>

### Confidence 🔵 4 (3)

<details>
<summary>Click to expand 3 issues</summary>

#### [#2](https://github.com/radareorg/radare2-r2pipe/issues/2) — Write proper mocha testsuite for NodeJS
*9y old · 0 comments · `nodejs`*

Request for a proper mocha test suite for the Node.js r2pipe. The current package.json shows the test script is just 'node examples/test-channels.js' - not a proper test framework. No mocha dependency exists. The test directory contains simple scripts, not structured test suites. No mocha or jest tests were ever written.

---

#### [#63](https://github.com/radareorg/radare2-r2pipe/issues/63) — Implement rap:// protocol handling in r2pipe.py and .js
*7y old · 0 comments*

Request to implement the rap:// (radare remote protocol) in r2pipe Python and JavaScript bindings. Searching the codebase for 'rap://' yields no results. The Python r2pipe supports http://, tcp://, ccall://, and spawn modes, but not rap://. This feature was never implemented.

---

#### [#160](https://github.com/radareorg/radare2-r2pipe/issues/160) — r2pipe.py have lots of linting issues
*2y old · 0 comments*

Detailed pylint output from January 2024 showing numerous code quality issues in the Python r2pipe module: redefined builtins, missing docstrings, bare excepts, deprecated arguments (loop parameter in asyncio), eval usage, bad indentation, duplicate code. Current source still shows the same patterns - class named 'open' (not PascalCase), super() with arguments, no docstrings, etc. These linting issues have not been addressed.

---

</details>

### Confidence 🟡 3 (5)

<details>
<summary>Click to expand 5 issues</summary>

#### [#32](https://github.com/radareorg/radare2-r2pipe/issues/32) — Handle batched commands with python API
*8y old · 4 comments*

Bug report about semicolon-separated commands causing desync in the pipe protocol because r2 sends a NULL after each sub-command but r2pipe only expects one. The current open_sync.py shows the same pipe-based communication with no special handling for multiple commands. The issue is inherent to the pipe protocol design.

---

#### [#36](https://github.com/radareorg/radare2-r2pipe/issues/36) — Timeout commands
*8y old · 4 comments*

Request for timeout support when running r2 commands via r2pipe (e.g., timeout for 'aaaa' analysis). The user reported anal.timeout gets stuck after showing the timeout message. No timeout mechanism has been added to r2pipe itself. The anal.timeout is a radare2 core feature, not r2pipe. R2pipe still has no built-in command timeout.

---

#### [#61](https://github.com/radareorg/radare2-r2pipe/issues/61) — Examples of using r2pipe to write plugins?
*7y old · 3 comments*

User asked for examples of using r2pipe to write radare2 plugins (assembler, IO plugin, syscall handler) as mentioned in the README. The maintainer pointed to radare2-bindings. The README still claims r2pipe can be used for writing plugins but no clear examples exist in this repository. The documentation is misleading.

---

#### [#77](https://github.com/radareorg/radare2-r2pipe/issues/77) — Multiple commands injection into cmd/cmdj while implementing automation scripts
*7y old · 3 comments*

Security concern about newlines in strings being interpreted as command separators when passed to r2 via r2pipe. The maintainer acknowledged the issue and suggested using base64 encoding for comments (CCa subcommand). However, the fundamental design where r2pipe passes commands as text and newlines act as separators has not changed. The current r2pipe code still uses newline as command terminator. This is by design in the r2 protocol but remains a footgun for users.

---

#### [#161](https://github.com/radareorg/radare2-r2pipe/issues/161) — r2pipe link breaks after ^C in r2ai
*1y old · 0 comments*

Report from June 2024 by trufae (pancake) showing r2pipe connection breaks after pressing Ctrl+C in r2ai. No comments or discussion. The issue is likely related to signal handling in the r2pipe protocol when interrupted during an r2ai session. No evidence of a fix in the repository. Given the reporter is a core maintainer, this is a known active issue.

---

</details>
