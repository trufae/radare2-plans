**Spec Summary**  
Summary: The core `yr` command layer is mostly straightforward, but the embedded YARA module has several crash-level null/error-path bugs, and the rule generator/CLI wrapper has a few user-visible regressions.  
Verdict: ❌ Request Changes

@@@task
# 🟠 `scanS` is wired to quiet mode
`yara scanS` is documented as the mode that prints matching strings, but it is dispatched as `cmd_yara_scan(..., "q")`, which turns string printing off instead.  
Severity: 🟠 medium  
Suggested fix: Route `scanS` to a dedicated "show strings" path and keep `"q"` only for quiet scans.  
Ref: [src/core_r2yara.c#L933](/Users/pancake/prg/r2yara/src/core_r2yara.c#L933)
@@@

@@@task
# 🟠 `yara show` can crash on non-YARA-X builds
`cmd_yara_show` passes `name` straight into `r_str_casestr`, but `yara show` without an argument leaves `name == NULL`. That is safe in the YARA-X branch, but it dereferences null in the legacy YARA branch.  
Severity: 🟠 medium  
Suggested fix: Treat a missing filter as "show all" before calling `r_str_casestr`.  
Ref: [src/core_r2yara.c#L476](/Users/pancake/prg/r2yara/src/core_r2yara.c#L476)
@@@

@@@task
# 🟠 Generated YARA rules are not escaped safely
The generator inserts config metadata and extracted strings directly into quoted YARA literals, only replacing newlines. Quotes and backslashes from the binary or config will therefore produce invalid rules or inject unintended syntax.  
Severity: 🟠 medium  
Suggested fix: Escape `"`, `\`, and other YARA-special characters before appending them to rule text.  
Ref: [src/core_r2yara.c#L681](/Users/pancake/prg/r2yara/src/core_r2yara.c#L681)  
Ref: [src/core_r2yara.c#L755](/Users/pancake/prg/r2yara/src/core_r2yara.c#L755)
@@@

@@@task
# 🔴 Module predicates dereference missing JSON fields
Several helpers call `strcasecmp` or `yr_re_match` on values returned by `json_string_value(...)` without checking for `NULL`, so an incomplete or malformed report can crash the scan. This pattern repeats across imports, sections, resources, and exports.  
Severity: 🔴 high  
Suggested fix: Guard every extracted field before comparing it and treat missing fields as a non-match.  
Ref: [yara-r2-module/src/r2.c#L203](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L203)  
Ref: [yara-r2-module/src/r2.c#L728](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L728)
@@@

@@@task
# 🔴 `module_load` continues after failed `r2`/JSON setup
If `r2p_open` fails or any `json_loadb` call returns null, the code still falls through into `json_array_foreach(...)` and `json_object_get(info, ...)` on uninitialized pointers. That turns tool/runtime failures into process crashes instead of returning `ERROR_INVALID_FILE`.  
Severity: 🔴 high  
Suggested fix: Validate every required JSON object before use and abort `module_load` cleanly when any dependency fails.  
Ref: [yara-r2-module/src/r2.c#L1523](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L1523)  
Ref: [yara-r2-module/src/r2.c#L1627](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L1627)  
Ref: [yara-r2-module/src/r2.c#L1714](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L1714)
@@@

@@@task
# 🔴 Hash collection writes through null or empty responses
The standalone module path assumes every `ph` command returns a non-empty string and immediately does `hash_value[strlen(hash_value) - 1] = '\0'`. A failed command or empty output will underflow or dereference null.  
Severity: 🔴 high  
Suggested fix: Check `hash_value != NULL` and `*hash_value != '\0'` before trimming, and handle failures as missing hashes.  
Ref: [yara-r2-module/src/r2.c#L1603](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L1603)
@@@

@@@task
# 🟡 JSON ownership in the module is inconsistent and leaks
When a report is provided, the parsed root `json` object is never stored on `module_object->data`, but `module_unload` later decrefs borrowed children and also misses `sections` because it asks for `"section"`. That leaks report-backed scans and makes the ownership model brittle.  
Severity: 🟡 low  
Suggested fix: Keep the root JSON on `module_object->data`, `incref` child objects only if needed, and fix the unload key to `"sections"`.  
Ref: [yara-r2-module/src/r2.c#L1509](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L1509)  
Ref: [yara-r2-module/src/r2.c#L1814](/Users/pancake/prg/r2yara/yara-r2-module/src/r2.c#L1814)
@@@

**Bad Practices**  
- The helper scripts are still Python 2-era code under `#!/usr/bin/python`, so they are likely broken on modern Python 3 systems: [generate_report.py#L23](/Users/pancake/prg/r2yara/yara-r2-module/generate_report.py#L23), [launch_tests.py#L243](/Users/pancake/prg/r2yara/yara-r2-module/launch_tests.py#L243).  
- `cmd_yara_scan` pushes the `yara` flag space and never pops it, so scans leave persistent UI state behind: [src/core_r2yara.c#L447](/Users/pancake/prg/r2yara/src/core_r2yara.c#L447).

**Missing Features**  
- Add maintained negative-path tests for malformed JSON, missing fields, failed `r2` startup, and empty hash outputs.  
- Finish the commented/planned module coverage for bins/fields/strings/symbols so rules can query more than imports/sections/resources/exports/hash/info.  
- Add structured output for scan results (`yrsj`/JSON) so other r2 tooling can consume matches without screen scraping.  
- Extend the rule generator with proper literal escaping plus options like `ascii`, `wide`, `nocase`, `fullword`, and custom condition templates.

Static review only; I didn’t build or run the plugin.
