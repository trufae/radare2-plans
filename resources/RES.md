# RBin Resources in radare2: State of the Art, Gaps, and a Proposal

`rabin2 -U` (alias: the `iR` command inside r2) is the entry point into the
"resources" subsystem. In radare2 today this subsystem is *almost exclusively*
a PE feature: the `.rsrc` directory of Windows executables is parsed, cached in
SDB, and then re-emitted in plain text, JSON or radare-script form. Everything
else — NE, LE/LX, Mach-O asset catalogs, classic Mac resource forks, APK
`resources.arsc`, ELF GResource sections, BeOS/Haiku attributes — is either
parsed and thrown away, or not parsed at all.

This document inventories what exists, shows where the code lives, describes
what other binary formats call "resources", and proposes how we can turn
`rabin2 -U` from a PE-only curiosity into a general, format-agnostic resource
viewer/extractor that is actually useful for reverse engineers, malware
analysts, localisers and forensics people.


## 1. Mental model: what "resource" means

A resource, for the purposes of this document, is **auxiliary, typed,
structured data embedded inside an executable or object file that is consumed
at runtime by the binary itself (or by a shell/loader)**. It is not code,
though it may be keyed by an ID, a type tag, a name and often a language. The
canonical taxonomy comes from Windows:

* **UI resources** — icons, cursors, menus, dialogs, accelerators, fonts.
* **Localisable text** — string tables, message tables, version strings.
* **Metadata** — version info (VS_VERSIONINFO), manifest XML (UAC, dpiAware,
  SxS), CLR .NET metadata.
* **Opaque blobs** — RCDATA (user-defined), HTML, bitmaps, arbitrary files.

Resources predate the Unix model of "everything is a separate file on disk" —
on Mac, Windows 1.0/NT, OS/2 and Palm OS the *fat binary is the bundle* — and
that is why they remain a rich source of forensic evidence on Windows, iOS and
Android to this day: they are convenient places to stash payloads, drop files,
hide licence keys, persist default data, or carry identity.


## 2. Current wiring in radare2 (as of master @ 2026-04-20)

### 2.1 CLI and command plumbing

The short-flag mapping:

* `binr/rabin2/` is built from `libr/main/rabin2.c`.
* `libr/main/rabin2.c:99` — help string: `" -U              resoUrces\n"`.
* `libr/main/rabin2.c:851` — `case 'U': set_action (R_BIN_REQ_RESOURCES);`
* `libr/main/rabin2.c:1304` — `run_action ("resources", R_BIN_REQ_RESOURCES,
  R_CORE_BIN_ACC_RESOURCES);`

The action bit lives in the public header:

* `libr/include/r_bin.h:72` — `#define R_BIN_REQ_RESOURCES 0x10000000`
* `libr/include/r_core.h:885` — `#define R_CORE_BIN_ACC_RESOURCES 0x100000`

Inside the r2 shell the command is `iR`:

* `libr/core/cmd_info.inc.c:161` — help row: `"iR", "", "list the resources"`.
* `libr/core/cmd_info.inc.c:2848-2849` —
  `case 'R': RBININFO ("resources", R_CORE_BIN_ACC_RESOURCES, NULL, 0);`

### 2.2 Core dispatcher

Everything funnels through `libr/core/cbin.c`:

```c
// libr/core/cbin.c:4876
static bool bin_resources(RCore *core, PJ *pj, int mode) {
    const RBinInfo *info = r_bin_get_info (core->bin);
    if (!info || !info->rclass) { ... return false; }
    if (!strncmp ("pe", info->rclass, 2)) {
        bin_pe_resources (core, pj, mode);
    } else {
        bin_no_resources (core, pj, mode);
    }
    return true;
}
```

This is the single largest architectural limitation of the whole subsystem: the
dispatcher is a `strncmp("pe", ...)`. No other rbin plugin can hook in. Adding
Mach-O `__TEXT.__cstring` resource dumping, APK `resources.arsc`, classic Mac
resource forks, ELF GResource, or NE (which *is* already parsed — see 2.5)
requires changing this dispatcher.

`bin_no_resources()` at `libr/core/cbin.c:4869` is a one-line stub that emits
an empty JSON array for all non-PE binaries.

### 2.3 The PE path

`bin_pe_resources()` lives at `libr/core/cbin.c:4793-4867`. It **does not
re-read the binary**: it reads a pre-populated SDB namespace at
`"bin/cur/info/pe_resource"` using keys shaped like `resource.%d.timestr`,
`resource.%d.vaddr`, `resource.%d.size`, `resource.%d.type`,
`resource.%d.language`, `resource.%d.name`. For each index it prints one of:

* Plain text (default) — human-readable key/value list plus `r_num_units()`
  size formatting.
* `-r` / `iR*` — radare script that creates flags:
  `f resource.0 <size> 0x<vaddr>` inside the `resources` flag space.
* `-j` / `iRj` — JSON array with `{name, index, type, vaddr, size, lang,
  timestamp}`.

Set mode (when invoked from inside `r_core_bin_info` during load) pushes flags
directly via `r_flag_set()` under the `R_FLAGS_FS_RESOURCES` flag space.

### 2.4 PE resource parser

The heavy lifting happens at load time, far from the CLI. The three key
functions live in `libr/bin/format/pe/pe.c`:

1. `PE_(bin_pe_parse_resource)` at line 3333 — entry point, called from
   `bin_pe_init()` at line 3515. Walks the three-level resource directory
   tree (type → name/id → language) starting from
   `pe->resource_directory_offset`, with a `HtUU` hash table of visited
   offsets as a belt-and-braces loop detector (malformed malware PEs love
   pointing resource subdirectories back at themselves).

2. `_parse_resource_directory()` at line 3173 — recursive walker. For each
   leaf it allocates a `r_pe_resource` struct and appends it to
   `pe->resources`. Named entries are UTF-16LE and are narrowed to ASCII
   byte-by-byte (see line 3206-3214) — this drops multibyte characters in
   named resources, which is something we should improve.

3. `_store_resource_sdb()` at line 3303 — flattens `pe->resources` into the
   SDB namespace.

The on-disk struct parsed into:

```c
// libr/bin/format/pe/pe.h:82
typedef struct _PE_RESOURCE {
    char *timestr;   // formatted timestamp
    char *type;      // "ICON", "VERSION", "MANIFEST", ...
    char *language;  // "LANG_ENGLISH", ...
    char *name;      // numeric id as string, or named entry
    Pe_image_resource_data_entry *data; // RVA, Size, CodePage
} r_pe_resource;
```

Attached to the PE object:

```c
// libr/bin/format/pe/pe.h:142
struct PE_(r_bin_pe_obj_t) {
    ...
    RList *resources; // RList<r_pe_resource>
    ...
};
```

### 2.5 Resource types currently recognised (PE)

Hard-coded in `libr/bin/format/pe/pe_specs.h:563-583` and stringified in
`libr/bin/format/pe/pe.c:3067`:

| ID | Constant | Meaning |
|----|----------|---------|
| 1  | CURSOR | Mouse cursor image |
| 2  | BITMAP | DIB bitmap |
| 3  | ICON | Icon image (referenced by GROUP_ICON) |
| 4  | MENU | Menu template |
| 5  | DIALOG | Dialog box template |
| 6  | STRING | Localised string table (16-per-block) |
| 7  | FONTDIR | Font directory |
| 8  | FONT | Font data |
| 9  | ACCELERATOR | Keyboard accelerator table |
| 10 | RCDATA | Arbitrary user-defined binary data |
| 11 | MESSAGETABLE | FormatMessage() lookup table |
| 12 | GROUP_CURSOR | Cursor directory |
| 14 | GROUP_ICON | Icon directory (enumerates ICONs + sizes) |
| 16 | VERSION | VS_VERSIONINFO |
| 17 | DLGINCLUDE | (compiler hint, `#include` fragment) |
| 19 | PLUGPLAY | Plug & Play resource |
| 20 | VXD | VxD driver data |
| 21 | ANICURSOR | Animated cursor |
| 22 | ANIICON | Animated icon |
| 23 | HTML | Embedded HTML |
| 24 | MANIFEST | Side-by-side / UAC manifest (XML) |

Languages: ~80 entries in `_resource_lang_str()` at `pe.c:2968`. The code
masks the entry with `0x3ff` to extract the 10-bit primary language (same
encoding as Windows `MAKELANGID()` / `PRIMARYLANGID()`).

### 2.6 The special case: VERSIONINFO

Version info gets a second, deeper parse because it has its own nested
format. When the top-level type equals `PE_RESOURCE_ENTRY_VERSION` (16) the
code at `pe.c:3243-3282`:

1. Translates the RVA to a file offset via `PE_(va2pa)`.
2. Requires 4-byte alignment (otherwise it bails with a warning — this is
   correct per spec but also rejects some weird packers; worth checking).
3. Loops calling `Pe_r_bin_pe_parse_version_info()` and
   `Pe_r_bin_store_resource_version_info()`, each producing an SDB
   sub-namespace like `VS_VERSIONINFO0`, `VS_VERSIONINFO1`, ...
4. Attaches everything under `pe->kv / vs_version_info`.

This is re-exposed to the user through a completely separate code path:
`bin_info -> version info` prints `CompanyName`, `ProductName`, `FileVersion`,
etc. via `iV` — not `iR`. Users frequently ask why the two commands disagree
about version resources; this is the reason.

### 2.7 Where "resources" are parsed but *not* surfaced

#### New Executable (NE) — 16-bit Windows/OS/2 1.x

`libr/bin/format/ne/ne.c:178-279`. A full parser exists:

* `__resource_type_str()` at line 178 — same types as PE, plus `NAMETABLE` at
  id 15.
* `__ne_get_resources()` at line 225 — walks
  `bin->ne_header->ResTableOffset`, reads `NE_image_typeinfo_entry` and
  `NE_image_nameinfo_entry` records, applies the shift/alignment factor, and
  builds `RList<r_ne_resource>` with nested entries.

The result lives in `bin->resources` (see `libr/bin/format/ne/ne.h:30`), but
the NE plugin (`libr/bin/p/bin_ne.c`) never forwards that list to any RBin
callback that `cbin.c:bin_resources` would pick up. **NE resources are dead
code from the user's perspective.**

#### Linear Executable (LE/LX) — 32-bit OS/2, VxD

`libr/bin/format/le/le_specs.h:13-35` defines `LE_resource_type` with 21 type
constants (`LE_RT_POINTER`, `LE_RT_BITMAP`, ..., `LE_RT_HELPTABLE`,
`LE_RT_FD`). The header also declares `ut32 rsrccnt` at line 190. But
`libr/bin/p/bin_le.c:78-79` only prints:

```
Resource Table: 0x%04x
Resource Count: %u
```

No individual entries are walked. LE support is effectively
informational-only.

#### QNX LMF

`libr/bin/format/qnx/qnx.h` defines `lmf_resource`, and `bin_qnx.c:50-94`
reads one `lmf_resource` header per record to compute the vsize for a
section. This is plumbing, not an end-user resource list.

### 2.8 Summary of user-visible gaps

* Only PE flows through `iR` / `-U`.
* PE resources are listed but never **extracted** (no `rabin2 -U -o out/`).
* Named resource entries are ASCII-narrowed; non-ASCII names get truncated.
* GROUP_ICON / ICON relationships are not reassembled into
  ready-to-open `.ico` files.
* VS_VERSIONINFO is parsed twice for different commands (inefficient, and
  sometimes inconsistent).
* MANIFEST XML bytes are never shown — the user gets an offset and size but
  has to `pd` the bytes manually to read the XML.
* No search/filter by type, language or name.
* No hashing (md5/sha1/sha256) per resource — which is exactly what malware
  triage tools want.
* NE data is parsed but invisible; LE is stubbed.


## 3. What resources look like in other formats

A quick tour of the landscape, so we can see how far we are from generality.

### 3.1 Classic Mac OS "resource fork"

Every Mac file historically had two forks: data and resource. The resource
fork is a structured archive of typed records, each keyed by:

* `OSType` — a 4-character type (e.g. `ICN#`, `cicn`, `PICT`, `snd ` (trailing
  space), `STR `, `STR#`, `MENU`, `DLOG`, `DITL`, `WIND`, `CODE`, `vers`,
  `BNDL`, `FREF`).
* A signed 16-bit ID.
* An optional name.

Storage is HFS/HFS+ metadata. On non-HFS systems forks are carried via
**AppleSingle** (one file combining data + resource forks) or
**AppleDouble** (sidecar `._filename`). Magic bytes `00 05 16 00` mark
AppleSingle; AppleDouble uses `00 05 16 07`. On macOS today
`com.apple.ResourceFork` is an extended attribute.

**Notable types:**

* `ICN#` — 128-byte B/W icon + 128-byte mask (32×32).
* `cicn` — colour icon.
* `PICT` — QuickDraw pictures.
* `snd ` — System 7 sound resource.
* `STR `, `STR#` — Pascal strings / string lists.
* `MENU`, `DLOG`, `DITL`, `WIND` — UI templates.
* `CODE` — **actual executable code segments** (!). In 68k Mac binaries the
  program itself lives in the resource fork, not the data fork. This is
  relevant to radare2 because old Mac binaries may fail to load today precisely
  because their "code" is parsed as "resource data" everywhere else.
* `vers` — version info, conceptually similar to VS_VERSIONINFO.
* `BNDL` / `FREF` / `ICN#` triples — file-type associations.

Prior art worth emulating: `rsrcdump` (Jorio), `resource_dasm`
(fuzziqersoftware), `ForkView` (kainjow), and the Kaitai spec at
`formats.kaitai.io/resource_fork/`.

### 3.2 Android APK — `resources.arsc`

Not strictly a "binary format resource" in the PE sense (it's a sidecar file
inside the APK, not a section of the DEX or ELF), but the *concept* maps
perfectly: it is a string-pool-based, chunk-oriented table of typed resources
(strings, drawables, dimens, colors, layouts) keyed by a 32-bit resource ID
(`package << 24 | type << 16 | entry`). Resource names, resolved values and
per-configuration overrides are all there.

Files of interest for us in radare2 context:
* `resources.arsc` — the resource table (main payload).
* `AndroidManifest.xml` — binary XML, encoded in the same chunk format.
* `classes.dex` — where the code lives (radare2 already reads this).

Radare2's `bin_dex.c` intentionally doesn't touch `resources.arsc` (only a
single comment mentioning "resources" in the context of freeing memory —
`bin_dex.c:2079`). If `-U` learned to delegate to a helper that can open the
APK zip and interpret `resources.arsc`, we'd expose a huge amount of useful
data (app name, version, icon paths, localised strings) without requiring
`aapt`.

### 3.3 iOS / macOS asset catalogs — `Assets.car`

Compiled asset catalogs. Inside they are a BOM (Bill of Materials) container
holding typed entries: image sets (including `@2x` / `@3x` variants), colour
sets, data sets, app icons. Not part of the Mach-O itself — they live
alongside it in the `.app` bundle — but for someone doing `rabin2 App.app/App`
today, the resources truly live in `Assets.car` and they get nothing.

Mach-O *does* have section-level conventions that are resource-ish and we
could start extracting opportunistically:

* `__TEXT.__cstring` — C string literals (already accessible via `rabin2
  -z`, but semantically adjacent).
* `__TEXT.__cfstring` — CoreFoundation string references.
* `__TEXT.__info_plist` — embedded `Info.plist` (common in command-line
  tools, XPC services).
* `__TEXT.__launchd_plist` — embedded launchd plist.
* `__TEXT.__entitlements` — code-signing entitlements blob.
* `__DATA.__objc_classlist`, `__objc_methlist`, etc. — ObjC runtime metadata.

None of these are exposed today under `iR`. Several are partially accessible
via other subsystems (entitlements via `rabin2 -e`, `__info_plist` via a
one-off command).

### 3.4 ELF + GResource (GNOME)

GNOME embeds resource bundles into ELF binaries using `glib-compile-resources
--generate-source` (producing a `.c` blob linked into the binary) or a
standalone `.gresource` file. The bundle is a GVariant-formatted indexed
table; `gresource list /path/to/binary` and `gresource extract /path/to/binary
/org/gnome/app/foo.ui` operate directly on the ELF's read-only data.

There is no standard ELF section name for this ("resources" live in
`.rodata` alongside everything else), but the magic header
(`GVARSC` / specific GVariant typecode) is distinctive enough that a scanner
could find them. This would let `rabin2 -U` on GNOME/GTK apps expose embedded
`.ui` files, CSS, icons and translations.

### 3.5 NE / LE / LX — other Microsoft / IBM families

Covered above under current state. LX (OS/2 Warp) and LE (Windows 3.x VxD)
use the same resource types as Windows with extra entries specific to Presentation
Manager (`LE_RT_HELPTABLE`, `LE_RT_KEYTBL`, `LE_RT_FDDIR`).

### 3.6 Rich Header (PE)

Not technically a "resource" but often requested in the same breath. The Rich
Header is a Microsoft linker artefact sitting between the DOS stub and the PE
header, XOR-encoded with a key placed just after the `Rich` signature. It
records which tool versions (linker, C runtime, import/export lib versions)
assembled the binary. Its forensic value is well established — Dumitras et
al. 2017 showed 71% of 964K malware samples retain the Rich Header, and it
survives most packers. Radare2 has partial Rich parsing already in
`libr/bin/format/pe/pe.c` (grep `Rich`), but it is not surfaced under `iR`.
Arguably it shouldn't be — but a family like `iRR` or a dedicated `iRich`
would be natural.

### 3.7 Palm OS `.prc` / `.pdb`

Records are typed (`appl`, `tFRM`, `tSTR`, `tbmp`, `Tbsb`, `code`). Already
loadable by radare2 as data, but without any notion of "this record is a
resource of type X". Old Palm binary reversing is still a niche but real
use-case.

### 3.8 Executable wrappers with embedded payloads

PE `RCDATA` is routinely used to carry:
* Compressed installers (NSIS, Inno Setup).
* Decoy documents for spear-phishing.
* Encrypted second-stage malware.
* Electron/NW.js compressed `.asar` bundles.
* PyInstaller frozen archives (with their own TOC at the end).

Identifying these via magic bytes inside RCDATA is something we could do
today if `-U` dumped payloads.


## 4. Typical use cases

### 4.1 Malware analysis and triage

* **Identify droppers** — dump RCDATA, hash each, run `file`/`yara` on it.
* **Find decoy documents** — most spear-phish executables stash a PDF/DOC in
  RCDATA to display while the second stage unpacks.
* **VERSIONINFO lies** — mismatch between `CompanyName` / `FileDescription`
  and the actual signer is a classic red flag.
* **Manifest checks** — `requestedExecutionLevel=requireAdministrator` plus
  a suspicious name is an instant UAC-prompt elevation play.
* **Icon reuse** — icons are routinely stolen from legitimate software; hash
  each ICON and cross-reference against a corpus.
* **Language hint** — resources in `LANG_RUSSIAN` or `LANG_CHINESE` on a
  supposed US-vendor binary is a strong signal.

### 4.2 Localisation / i18n

* Dump all `STRING` and `MESSAGETABLE` resources.
* Group by language.
* Diff translations across builds.

### 4.3 Reverse engineering / forensics

* Recover missing app icons from an exe.
* Extract embedded HTML/JS to understand UI logic.
* Rebuild a `.rc` file from a compiled binary.
* Pull the manifest XML to understand UAC/SxS behaviour.
* Extract certificates embedded as RCDATA.

### 4.4 Supply-chain / compliance

* Verify VERSIONINFO fields match the SBOM.
* Hash each resource blob and include in SBOM (the rest of rabin2 already
  does this for sections via `-s --hashes`).


## 5. What radare2 already has in its toolbelt

Most of the building blocks are there:

* **PE loader + resource parser** (see §2.4).
* **SDB** — easy to stash typed, namespaced data so other commands can query
  it without re-parsing.
* **JSON via `PJ`** — already the canonical output format.
* **Flag spaces** — `R_FLAGS_FS_RESOURCES` is already created.
* **`r_bin_file_hash_new`** — same helper `iS,cs` uses to hash sections; can
  hash a `[vaddr,size)` range of resource bytes.
* **`RBinFile::extract`** — already used for fat/universal; a sibling
  `RBinFile::extract_resource(name, id) -> RBuffer*` would fit naturally.
* **`r_bin_string_*`** — can be re-used to surface STRING tables.


## 6. Concrete improvement proposal

Aim: make `-U` a real, format-generic resource browser and extractor.

### 6.1 Promote `resources` to a first-class RBinPlugin method

Add to `RBinPlugin` (in `libr/include/r_bin.h`):

```c
RList /*<RBinResource *>*/ *(*resources)(RBinFile *bf);
```

Where `RBinResource` is:

```c
typedef struct r_bin_resource_t {
    char *type;        // "ICON", "STRING", "manifest", "assets.car/png", ...
    char *name;        // human-readable name or id-as-string
    char *language;    // "en-US", "LANG_NEUTRAL", or NULL
    ut64 vaddr;        // virtual address, UT64_MAX if not mapped
    ut64 paddr;        // physical offset in the file
    ut32 size;         // raw size
    ut32 codepage;     // for text resources
    char *timestr;     // ISO-8601 timestamp or NULL
    char *comment;     // free-form "contains UAC=requireAdmin" etc.
    ut32 index;        // per-plugin ordinal
    // Optional hashes filled by -h variant
    char *md5, *sha1, *sha256;
} RBinResource;
```

Refactor `bin_resources()` in `cbin.c` to iterate `plugin->resources(bf)` and
drop the `strncmp("pe",…)` dispatch. Provide a default implementation that
synthesises resources from NE (already parsed, see §2.5), LE (stubbed),
Mach-O (§3.3), and ELF GResource (§3.4).

### 6.2 Enable PE resource extraction

Add a new rabin2 flag `-U -o <dir>` or an r2 command `iRo <dir>` that, for
each resource, writes `dir/<index>_<type>_<name>.bin`. For known wrapper
types, do the conversion:

* `ICON` + `GROUP_ICON` → reassemble into `.ico` (prepend ICONDIR header).
* `CURSOR` + `GROUP_CURSOR` → reassemble into `.cur`.
* `BITMAP` → prepend BITMAPFILEHEADER (the compiled resource only has the
  DIB payload starting at BITMAPINFOHEADER).
* `MANIFEST` → write as `.manifest` (already XML).
* `HTML` → write as `.html`.
* `STRING` — decode the 16-block UTF-16 table and dump as JSON/po.
* `RCDATA` — sniff magic at offset 0 and use the correct extension
  (`.zip`, `.pdf`, `.exe`, `.asar`, `.pyz`, else `.bin`).

Reference implementations to mimic: `wrestool` (icoutils), NirSoft
`ResourcesExtract`, and Jorio's `rsrcdump` for Mac.

### 6.3 Show manifest XML inline

When the type is `MANIFEST`, pretty-print the XML under `iR` verbose mode
(e.g. `iRR` or `iRv`). Surface interesting attributes to a `i`-level summary:

```
requestedExecutionLevel: requireAdministrator (uiAccess=false)
dpiAwareness: PerMonitorV2
Windows compatibility: 10, 8.1, 8, 7
assemblyIdentity: Microsoft.Windows.Common-Controls v6.0.0.0
```

### 6.4 Hash and fingerprint each resource

`-U --hashes` / `iR,md5,sha256` — include md5 / sha1 / sha256 of each
resource's raw bytes in the JSON output and the plain-text output. Same
machinery as `-s --hashes` uses for sections (see the recent commit
`639de8da43 Reuse section hashing code to reduce LOCs`). This is the single
most useful addition for malware triage and correlates well with the Rich
Header — together they make a strong provenance signal.

### 6.5 Wire up NE

`libr/bin/p/bin_ne.c` already has `bin->resources`. Implement the new
`plugin->resources()` method to walk it and return `RBinResource`s. Costs
~30 lines.

### 6.6 Wire up LE/LX

Expand `libr/bin/p/bin_le.c` to actually walk the resource table using
`rsrccnt` and the already-defined `LE_resource_type` enum. One-time cost,
benefits the OS/2 and VxD communities.

### 6.7 Mach-O: opportunistic resources

Populate `RBinResource` entries from:

* `__TEXT.__info_plist` — show the plist directly.
* `__TEXT.__launchd_plist` — same.
* `__TEXT.__entitlements` — show the property list (already in `-e`, but as
  a resource it should show up in `iR` too).
* Sidecar detection — if the Mach-O path is inside a `.app` bundle, probe
  for `Contents/Resources/Assets.car`, `Info.plist`, `*.nib`, `*.storyboardc`
  and emit them as external resources (with `paddr = UT64_MAX, external =
  true`). This mirrors how `rabin2` already walks fat binaries.

### 6.8 APK: resources.arsc as an external provider

When the loaded file is an APK (detectable via zip magic + `AndroidManifest.xml`
presence — `bin_dex` already has enough to know it), read
`resources.arsc` and `AndroidManifest.xml` (binary XML) via a small
chunk-parser and emit the typed entries as `RBinResource`s. Start with:

* Application label (from AndroidManifest → `resources.arsc` string pool).
* Version code and name.
* Icon resource name.
* Permission list.
* Declared activities/services.

`android-arscblamer` (Google) and `apktool` show the format is small enough
to handle in-tree without an external dep.

### 6.9 ELF GResource scanning

Scan read-only segments for the GResource magic (GVariant typecode
`"a{sv}"` framing, reachable from `g_resources_register` entry points or
via exported symbol names `*_resource_get_resource`). This is best-effort; a
`-U --scan` flag could enable it on demand.

### 6.10 Classic Mac resource fork and AppleSingle/AppleDouble

A new rbin plugin `bin_rsrc` that accepts:

* Raw resource fork data (loaded via the macOS xattr reader or a sidecar
  `._` file).
* `AppleSingle` (magic `00 05 16 00`).
* `AppleDouble` (magic `00 05 16 07`).

Emit entries with `type` set to the 4-char OSType, `name` from the resource
name table, `language` left NULL. For type `CODE`, flip the entry to also be
a section so r2's disassembler picks it up — this finally lets radare2
properly open 68k Mac applications.

### 6.11 Output ergonomics

* `iR~icon` — grep by type (already works, `r_cons_grep`).
* `iRq` — quiet mode, `type:name` one per line.
* `iRc` — count by type.
* `iRt <type>` — filter by type.
* `iRl <lang>` — filter by language.
* `iRx <idx>` — hex-dump resource.
* `iRx.` — hex-dump resource at current seek.
* `iRo <dir>` — extract all.

### 6.12 SDB cleanup

Version info is stashed at `bin/cur/info/vs_version_info` and `pe_resource`
independently; they should be nested so `iR` can link each resource to its
parsed structured form where available.

### 6.13 Documentation

Update `doc/intro.md` and `doc/bin.md` (if present) with a table of which
formats support resources, what types they expose, and what commands map to
what.


## 7. Bugs and micro-issues spotted during this audit

These are concrete, independent, small fixes worth landing first:

1. **`pe.c:3206-3214` narrows UTF-16 named resources to ASCII.** Any
   non-ASCII byte is converted to zero and the string is freed — meaning
   named resources with accented characters become unnamed. Replace with
   `r_str_utf16_to_utf8()` (already in `libr/util/utf8.c`).

2. **`bin_no_resources()` emits `pj_a` for JSON but nothing for plain/rad
   modes.** Non-PE formats print nothing instead of a hint like "no
   resources for format <rclass>". Low-value fix but improves UX.

3. **`cbin.c:4858` — `R_FREE (lang)` is missing a trailing `;`** (it
   compiles because `index++;` supplies the terminator, but it's ugly).

4. **`bin_resources()` returns `false` when `info->rclass` is NULL.** For
   unknown binaries this is probably right, but `-U` on something that is
   recognised but has no resources currently returns `false` too, which
   propagates up through `run_action` and can mess with exit codes in
   scripted pipelines. Should be `true` with empty output.

5. **`R_PE_MAX_RESOURCES` is a hard cap.** A legitimate installer can carry
   thousands of resources. Worth auditing the actual value and either
   raising it or making it a config variable.

6. **`is_dos_time()` heuristic (`pe.c:3165`).** Compares high 16 bits of
   two 32-bit timestamps. Fails on builds produced around 1980-01-01 or
   across DST shifts. Not a common issue but a source of occasional wrong
   timestamps.

7. **LE plugin never walks the resource table** despite having the offsets
   in the header (see §2.7). Low-effort fix.

8. **NE resources are dead code** (§2.7). Medium-effort fix.

9. **Version info alignment strictness** (`pe.c:3260`). Some packers
   deliberately pad incorrectly. Consider warning but still attempting a
   best-effort parse.

10. **No fuzzing surface for rbin resources.** There's plenty of parser code
    that trusts on-disk offsets. Given that malware PE resource dirs are
    adversarial input, it'd be wise to add an oss-fuzz target that feeds
    random bytes into `PE_(bin_pe_parse_resource)`. Loop guards exist
    (`HtUU *dirs`) but recursion depth is not capped.


## 8. Implementation roadmap (suggested order)

1. **Land micro-fixes** (§7.1, §7.2, §7.3, §7.4).  — 1 afternoon.
2. **Introduce `RBinResource` and `plugin->resources()`** without breaking
   callers: keep `bin_pe_resources()` reachable, add a fallback that pulls
   from the SDB namespace for now. — 1 day.
3. **Wire NE through the new API.** — 2 hours.
4. **Add extraction (`-U -o dir`) with conversion for ICON, CURSOR, BITMAP,
   MANIFEST.** — 1 day.
5. **Hashes (`-U --hashes` / `iR,md5,sha256`).** — 2 hours (reuse section
   hashing).
6. **Wire LE.** — 0.5 day.
7. **Mach-O opportunistic (`__info_plist`, entitlements, bundle
   sidecars).** — 1 day.
8. **APK `resources.arsc` reader.** — 2-3 days depending on how deep we go.
9. **ELF GResource scanner.** — 1 day opportunistic, more to make robust.
10. **`bin_rsrc` plugin for classic Mac.** — 2-3 days. High marginal value
    for the retro-reversing community.
11. **Ergonomics pass (`iRt`, `iRl`, `iRx`, `iRc`, `iRq`).** — 0.5 day.
12. **Documentation update.** — 0.5 day.

Total: roughly two weeks of focused work for a complete overhaul.


## 9. Quick reference — current command cheatsheet

| Command | What it does | Status |
|---------|--------------|--------|
| `rabin2 -U foo.exe` | Plain text list of PE resources | Works |
| `rabin2 -Uj foo.exe` | JSON list | Works |
| `rabin2 -Ur foo.exe` | Radare script with flags | Works |
| `rabin2 -V foo.exe` | VS_VERSIONINFO only | Works (separate path) |
| `iR` | Resource list in r2 shell | Works (PE only) |
| `iRj` | JSON | Works |
| `iR*` | Radare script | Works |
| `iV` | VS_VERSIONINFO | Works |
| `fs resources; f*` | Iterate resource flags | Works if loaded with `e bin.info=true` |

After the proposed changes:

| Command | What it would do |
|---------|------------------|
| `iRt ICON` | Filter by type |
| `iRl LANG_ENGLISH` | Filter by language |
| `iRx <idx>` | Hex-dump resource contents |
| `iRo <dir>` | Extract all resources to disk |
| `iR,md5,sha256` | Include hashes |
| `iRR` | Verbose (includes parsed XML for manifest, decoded strings, etc.) |
| `iRc` | Counts by type |


## 10. Conclusion

The resources subsystem is a small, self-contained, PE-only island in an
otherwise very format-plural radare2. The existing code is correct and
reasonably efficient — the issue is **reach**, not quality. We have all the
parser code for NE resources sitting unused, a stub for LE, no Mach-O
hooking, no Android hooking, no Mac resource fork plugin, and no
extraction/hashing story.

The proposal in §6 is entirely additive: introduce a thin plugin method
(`plugin->resources()` returning `RList<RBinResource *>`), swap the
`strncmp("pe",…)` dispatcher for an iteration over whoever implements it,
and then retrofit every existing format that already has the data. Three
small fixes would also correct the UTF-16 narrowing bug in PE named
resources, resurrect NE's dead parser, and give LE the handful of lines it
needs.

For malware analysts, reversers and localisers, a working `iRo` (extract)
and `iR,sha256` (hash) would alone make radare2 competitive with the
proprietary PE tooling people keep reaching for (ResourcesExtract, Restorator,
PE Explorer).


---

## References and further reading

* [Macintosh resource fork spec (Kaitai Struct)](https://formats.kaitai.io/resource_fork/)
* [Resource fork (Wikipedia)](https://en.wikipedia.org/wiki/Resource_fork)
* [rsrcdump — Jorio](https://github.com/jorio/rsrcdump)
* [resource_dasm — fuzziqersoftware](https://github.com/fuzziqersoftware/resource_dasm)
* [ForkView — kainjow](https://github.com/kainjow/ForkView)
* [GResource (GLib) reference](https://blogs.gnome.org/alexl/2012/01/26/resources-in-glib/)
* [gresource(1) manual](https://www.mankier.com/1/gresource)
* [GLib gresource.c source](https://github.com/GNOME/glib/blob/main/gio/gresource.c)
* [The resources.arsc file (Apktool)](https://apktool.org/wiki/advanced/resources-arsc/)
* [ARSC format (fileformat.com)](https://docs.fileformat.com/programming/arsc/)
* [android-arscblamer — Google](https://github.com/google/android-arscblamer)
* [Life of an Android Resource (Chromium)](https://chromium.googlesource.com/chromium/src/+/master/build/android/docs/life_of_a_resource.md)
* [Reverse engineering Assets.car — Timac](https://blog.timac.org/2018/1018-reverse-engineering-the-car-file-format/)
* [Apple asset catalog reference](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/index.html)
* [AppleSingle / AppleDouble formats (Wikipedia)](https://en.wikipedia.org/wiki/AppleSingle_and_AppleDouble_formats)
* [RFC 1740 — MIME Encapsulation of Macintosh Files](https://datatracker.ietf.org/doc/html/rfc1740)
* [AppleDouble (Library of Congress FDD)](https://www.loc.gov/preservation/digital/formats/fdd/fdd000625.shtml)
* [New Executable format (Wikipedia)](https://en.wikipedia.org/wiki/New_Executable)
* [NE format (OSDev Wiki)](https://wiki.osdev.org/NE)
* [NE format (fileformat.info)](https://www.fileformat.info/format/exe/corion-ne.htm)
* [Linear Executable (Wikipedia)](https://en.wikipedia.org/wiki/Linear_Executable)
* [LX format description (EDM2)](https://www.edm2.com/index.php/LX_-_Linear_eXecutable_Module_Format_Description)
* [LE format (OSDev Wiki)](https://wiki.osdev.org/LE)
* [XCOFF spec (IBM)](https://www.ibm.com/docs/ssw_aix_71/filesreference/XCOFF.html)
* [PE Format (Microsoft Learn)](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)
* [Application manifests (Microsoft Learn)](https://learn.microsoft.com/en-us/windows/win32/sbscs/application-manifests)
* [Comprehensive Guide to Windows Resources (gist)](https://gist.github.com/MangaD/9f6ca592948a8e7358998ed93c6ba039)
* [Finding the Needle: PE32 Rich Header study (DIMVA 2017)](https://users.ics.forth.gr/~zarras/files/DIMVA_2017_Finding.pdf)
* [Leveraging the PE Rich Header for Static Malware Detection (GIAC)](https://www.giac.org/paper/grem/6321/leveraging-pe-rich-header-static-alware-etection-linking/169729)
* [saferwall/pe — malware-analysis-oriented PE parser](https://github.com/saferwall/pe)
* [wrestool (icoutils) — Ubuntu manpage](https://manpages.ubuntu.com/manpages/bionic/man1/wrestool.1.html)
* [ResourcesExtract (NirSoft)](https://www.nirsoft.net/utils/resources_extract.html)
* [pefile — Go PE parser with resource extraction](https://github.com/folbricht/pefile)
