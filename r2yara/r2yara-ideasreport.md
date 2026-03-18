# r2yara Ideas Report

Date: 2026-03-18

## What the project already does well

`r2yara` already has two strong foundations:

- A native `yr` command inside radare2 to load rules, list them, scan the current file or maps, and generate simple rules from strings, hex patterns, function signatures, and referenced strings.
- An `import "r2"` YARA module that lets rules use higher-level binary metadata such as imports, sections, resources, exports, libraries, hashes, and general file info.

That combination is interesting because it moves YARA from pure byte matching into binary-aware hunting. In real workflows, that matters when:

- Malware hides behavior behind dynamic imports, packers, or thin loaders.
- Firmware and non-PE formats need richer structure than the stock PE/ELF modules expose.
- Analysts want to match semantic properties discovered during reversing, not just raw strings.
- Rules need to scale from one-off interactive triage to repeatable offline scanning.

## Main product direction

The best long-term direction is to treat `r2yara` as three products that share the same engine:

1. An interactive radare2 plugin for reverse engineers.
2. A YARA module that enriches rules with radare2-derived metadata.
3. A rule-management and automation toolchain for repeatable hunting.

If those three layers share one extraction core and one rules/cache layer, the project can cover both interactive reversing and batch threat hunting without duplicating logic.

## GitHub issues reviewed

I reviewed these open issues:

- Issue #2: https://github.com/radareorg/r2yara/issues/2
- Issue #13: https://github.com/radareorg/r2yara/issues/13
- Issue #16: https://github.com/radareorg/r2yara/issues/16
- Issue #20: https://github.com/radareorg/r2yara/issues/20
- Issue #21: https://github.com/radareorg/r2yara/issues/21
- Issue #43: https://github.com/radareorg/r2yara/issues/43

## Issue-by-issue ideas

### Issue #43: Run YARA rules on the output of a command

Issue: https://github.com/radareorg/r2yara/issues/43

This is one of the highest-value ideas in the whole tracker. In real malware analysis, a lot of the most useful evidence is easier to detect in decompiler output, disassembly text, string tables, import summaries, and analysis reports than in raw bytes.

Examples of real use cases:

- Scan `pdc` output for decompiler artifacts that reveal crypto constants, mutex names, command IDs, registry paths, or encoded PowerShell fragments.
- Scan `pdf`, `pdr`, or `pD` output to detect characteristic instruction sequences after normalization.
- Scan `izzq` or `izzz` output to find suspicious string clusters scoped to functions or sections.
- Scan command output from analysis plugins to detect capabilities, not only literals.

Recommended implementation:

- Add a new scan family that treats command output as a virtual text buffer.
- Start with a simple API such as:
  - `yrsc <r2cmd>` to scan the output of one command.
  - `yrscj <r2cmd>` for JSON output.
  - `yrscf <r2cmd>` to scan one output blob per function when used with `@@f`.
- Reuse the existing in-memory YARA scan path instead of inventing a second scanner.
- Add match metadata that records which command produced the match, and optionally which function/basic block/offset it came from.

Important design choice:

- Raw text scanning should be versioned and normalized, otherwise rules become fragile across formatting changes.
- For decompiler/disassembly scanning, add a normalization mode:
  - strip comments
  - collapse whitespace
  - lowercase mnemonics
  - optionally replace registers, addresses, and immediates with placeholders

Best first milestone:

- `yrsc pdc @ sym.main`
- `yrsc pD 256 @ entry0`
- `yrsc izzq`

Stretch goal:

- Add "structured command scanning" where JSON output is converted into canonical text before scanning, so rules are stable even if pretty-print formatting changes.

Why this matters:

- This feature lets YARA rules reason about reverse-engineered meaning, not just bytes.
- It also complements Issue #21 because constants extracted from code can be matched both as encoded bytes and as textual artifacts.

### Issue #21: Extract constants from function and register them as byte patterns

Issue: https://github.com/radareorg/r2yara/issues/21

This is a very strong idea for malware family hunting. Many families reuse magic values, crypto round constants, protocol IDs, syscall IDs, dispatch tables, or bitmasks even when strings change.

The important part in the issue is correct: constants should be extracted as encoded bytes, not as abstract integer values. That is what makes them useful for raw YARA matching.

Recommended implementation:

- Add a generator command focused on immediates, for example:
  - `yrgc` for constants in current function
  - `yrgC` for constants in selected range
  - `yrgci` to print the extracted immediates before adding them
- Use radare2 analysis APIs or JSON outputs that expose decoded operands and operand sizes.
- For each immediate:
  - determine operand width
  - encode in endianness-aware byte order
  - skip values that are obviously relocation addresses, stack offsets, or trivial small constants unless the user asks for them

Useful filters:

- minimum width: 2, 4, 8 bytes
- entropy or rarity threshold across the binary
- ignore common constants like 0, 1, -1, page sizes, alignment masks
- only keep constants referenced at least N times
- scope by function, section, namespace, or current analysis selection

Useful output forms:

- Raw byte strings for YARA
- Hex strings with wildcards where relocations or instruction prefixes vary
- A "constant profile" report before generating the final rule

Best first milestone:

- Support x86/x64 and ARM64 immediates first.
- Focus on current function only.
- Generate one hex string per encoded immediate sequence.

Stretch goal:

- Mine repeated constant combinations from a function and generate "N of them" conditions automatically.
- Add an option to pair constants with nearby API names or strings, which is often stronger than constants alone.

Why this matters:

- This produces more resilient signatures than string-only generation.
- It is especially useful against stripped samples, loaders, shellcode, and obfuscated malware with low string visibility.

### Issue #20: Implement native `yara` and `yarac` commands inside r2

Issue: https://github.com/radareorg/r2yara/issues/20

This issue is partly conceptual because `r2yara` already embeds YARA for `yr`, but it still does not feel like a full `yara`/`yarac` environment inside radare2. The opportunity here is to expose the embedded compiler and scanner in a more complete and scriptable way.

Recommended implementation:

- Keep `yr` as the concise interactive command.
- Add full-form commands for parity with normal YARA workflows:
  - `yara add <file>`
  - `yara scan`
  - `yara check <file>` for syntax-only validation
  - `yarac <input> <output>` to compile and cache a rule pack
- Support:
  - namespaces
  - include paths
  - external variables
  - precompiled rule bundles
  - JSON output

Important idea:

- The native `yarac` support should be treated as a cache layer, not only a compatibility alias.
- Analysts often reuse the same packs across many samples, so compiled-rule caching can save a lot of startup time.

Best first milestone:

- Syntax check and compile-to-cache inside `~/.local/share/radare2/plugins/rules-yara3` or a dedicated cache directory.
- Add `yrsj` to produce machine-readable match output.

Stretch goal:

- Make compiled rule sets usable from r2pipe and scripts without shelling out.
- Expose `yr` operations as library-friendly commands for automation.

Why this matters:

- It removes friction in environments where the analyst is already inside r2.
- It also sets up the infrastructure needed by the future CLI tool in Issue #2.

### Issue #2: Add an `r2yara` CLI tool

Issue: https://github.com/radareorg/r2yara/issues/2

This is the best ecosystem play in the tracker. In the real world, analysts rarely maintain one flat folder of YARA rules forever. They curate rule packs by source, family, platform, trust level, and freshness.

The issue body and comments suggest two related goals:

- Manage rule databases from multiple sources.
- Use r2 to help generate or consume those rules.

Recommended scope for the CLI:

- `r2yara rules search`
- `r2yara rules install`
- `r2yara rules update`
- `r2yara rules list`
- `r2yara rules enable/disable`
- `r2yara rules lint`
- `r2yara rules compile`
- `r2yara scan <sample>`

Recommended architecture:

- Local metadata index in JSON or SQLite.
- Rule sources as pluggable backends:
  - Git repositories
  - HTTP endpoints
  - vendor APIs
  - local directories
- Profiles:
  - `malware`
  - `pe`
  - `elf`
  - `macho`
  - `packed`
  - `crypto`
  - `firmware`

What would make it actually useful:

- Tag rule packs with source, family, target platform, confidence, and update date.
- Allow "trusted only" or "local only" modes.
- De-duplicate identical or conflicting rules.
- Compile packs once and reuse compiled bundles.
- Keep third-party content separate from user-authored rules.

Best first milestone:

- Support local Git-based rule sources and a lockfile-style manifest.
- Install packs into a per-user directory.
- Compile them into one cached bundle consumable by the plugin.

Stretch goal:

- Support authenticated backends like VirusTotal.
- Add reputation/trust metadata and selective synchronization.

Why this matters:

- This turns `r2yara` from a plugin into a workable analyst environment.
- It also makes distribution and updates much easier for teams.

### Issue #13: Windows CI builds

Issue: https://github.com/radareorg/r2yara/issues/13

This is more than packaging hygiene. If `r2yara` wants adoption in malware analysis, Windows support is mandatory.

Recommended implementation:

- Add GitHub Actions matrix builds for:
  - Windows x64
  - Linux x64
  - macOS x64/arm64 if possible
- Build YARA as part of the workflow or vendor the supported configuration.
- Produce installable artifacts for r2pm consumption, not only test binaries.

Specific technical work:

- Decide whether Windows will use static YARA, bundled DLLs, or a vendored subproject.
- Make sure the plugin install path matches the radare2 plugin path on Windows.
- Add CI tests that actually load the plugin in r2 and run:
  - `yr?`
  - load rule
  - scan test binary

High-value follow-up:

- Add regression fixtures for PE resources, imports, exports, and hashes because Windows is where those use cases matter most.

Why this matters:

- It is hard to argue for Windows-focused malware hunting with no Windows artifact pipeline.

### Issue #16: macOS package

Issue: https://github.com/radareorg/r2yara/issues/16

This is mostly packaging work, but it matters because macOS analysts and iOS/macOS malware researchers are a relevant audience for radare2.

Recommended implementation:

- Follow the existing radare2 macOS packaging template, as the issue suggests.
- Build universal binaries where practical.
- Test both Intel and Apple Silicon install paths.
- Verify that r2pm installs the plugin, docs, default rules, and any dependent libraries into consistent locations.

Useful extra idea:

- Ship a small default rules pack aimed at Mach-O and Apple ecosystem hunting:
  - suspicious entitlements strings
  - Objective-C selector patterns
  - DYLD and launch agent indicators

Why this matters:

- Packaging is not glamorous, but it removes one of the highest-friction adoption barriers.

## Additional ideas that are not explicit GitHub issues yet

### 1. JSON output for all scan modes

This should be added early.

Suggested commands:

- `yrsj` for current-file scan results
- `yrsJ` for verbose results with rule, string identifier, address, bytes, map, tag, namespace

Why:

- JSON output makes `r2yara` useful from r2pipe, CI, notebooks, and pipelines.
- It also makes it much easier to test.

### 2. Rule generation from analysis artifacts, not only bytes

The existing generator is a good start, but it can go much further:

- imported API combinations
- section traits
- resource language/type fingerprints
- function call graph motifs
- string xrefs per function
- hashes of sections or functions
- normalized basic block fingerprints

This would let analysts create "behavior-shaped" rules faster.

### 3. Function-scoped and section-scoped scanning

Real reverse engineering often asks:

- which functions match this pattern?
- which sections contain the matched strings?
- which imported capability cluster belongs to this function?

Suggested additions:

- `yrsf` scan each function as a separate unit
- `yrsb` scan each basic block
- `yrss` scan each section

This is a natural fit for triage and clustering.

### 4. Better offline report workflow for the `r2` YARA module

The module already supports JSON reports, but the old helper scripts are dated.

Ideas:

- Replace the Python 2 helper with a maintained Python 3 or C tool.
- Define a stable report schema version.
- Add validation and schema tests.
- Allow partial reports for faster use:
  - imports only
  - sections plus hashes
  - strings plus xrefs

This would make the module more practical for batch processing.

### 5. Externals and environment-aware scans

Many production YARA rules rely on external variables. `r2yara` should expose that cleanly.

Examples:

- target family
- OS
- architecture
- analyst-selected mode
- sample source

That would make it easier to reuse one ruleset across multiple workflows.

### 6. Rule QA and explainability tools

This is very useful when analysts generate rules interactively.

Ideas:

- `yr explain` to show why a rule matched
- `yr lint` to detect weak generated rules
- `yr diff` to compare match sets between two rule packs
- `yr profile` to show slow or noisy rules

This would reduce false positives and make generated rules more trustworthy.

### 7. Threat-hunting packs by domain

Instead of only shipping sample rules, curate small high-signal packs for:

- packers
- crypto usage
- loaders and droppers
- suspicious import clusters
- malware persistence artifacts
- firmware/embedded traits
- shellcode indicators

The plugin becomes much easier to adopt when useful packs exist on day one.

## Recommended roadmap

### Phase 1: Stabilize the current core

Before large feature work:

- fix crash-prone JSON handling in the YARA module
- modernize the report-generation helper
- add tests for malformed inputs and negative paths
- add JSON output for matches

This phase improves trust and makes later features easier to build.

### Phase 2: Make the plugin automation-friendly

- implement native `yara`/`yarac` parity features from Issue #20
- add compiled-rule caching
- expose clean JSON outputs
- improve r2pipe friendliness

This phase makes the project usable in scripts and pipelines.

### Phase 3: Add the high-value reverse-engineering features

- implement constant extraction from Issue #21
- implement command-output scanning from Issue #43
- add function-scoped and section-scoped scanning

This phase is where `r2yara` becomes genuinely differentiated from plain YARA.

### Phase 4: Solve distribution and ecosystem adoption

- Windows CI and packaging from Issue #13
- macOS packaging from Issue #16
- CLI and ruleset management from Issue #2

This phase helps the project reach more users and makes rule sharing realistic.

## Concrete feature ideas by user persona

### Malware analyst

- scan decompiler output for protocol commands
- generate rules from API sets plus immediates
- ship packer/loader/persistence starter packs

### DFIR / triage analyst

- batch-scan many files with cached compiled rules
- produce JSON and CSV outputs
- support offline report generation without opening samples interactively

### Reverse engineer

- generate YARA from current function, section, or call graph neighborhood
- inspect why a rule matched at a given function or offset
- convert interesting analysis artifacts directly into a draft rule

### Team / CI owner

- reproducible rule packs
- pinned external sources
- fast cached scans
- Windows/macOS/Linux artifacts

## Final recommendation

If only three ideas are implemented in the near term, they should be:

1. Stabilize and modernize the extraction/reporting core.
2. Implement Issue #21 and Issue #43 as the signature reverse-engineering features.
3. Build the rule cache and CLI/package layer around Issue #20 and Issue #2.

That combination would make `r2yara` useful both as a serious analyst tool and as a platform for advanced YARA workflows.

## Sources

- Local project docs and source tree in this repository.
- GitHub issues page: https://github.com/radareorg/r2yara/issues
- Issue #2: https://github.com/radareorg/r2yara/issues/2
- Issue #13: https://github.com/radareorg/r2yara/issues/13
- Issue #16: https://github.com/radareorg/r2yara/issues/16
- Issue #20: https://github.com/radareorg/r2yara/issues/20
- Issue #21: https://github.com/radareorg/r2yara/issues/21
- Issue #43: https://github.com/radareorg/r2yara/issues/43
