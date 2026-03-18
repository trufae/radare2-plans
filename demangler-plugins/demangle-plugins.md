# Demangler Plugin Support (Issue #23061)

## Summary

Issue #23061 requests dedicated demangler plugin support. Currently, demanglers are
hardcoded in a switch statement in `libr/bin/demangle.c:185-194`, with a partial escape
hatch via `RBinPlugin.demangle` (only used by dlang). The existing mechanism piggybacks
on `RBinPlugin` — a massive struct designed for binary format parsing — making it awkward
for external demangler-only plugins. A proper `RDemanglePlugin` type is needed.

## Current Architecture Problems

### 1. Hardcoded dispatch (`libr/bin/demangle.c:185-194`)

```c
switch (type) {
case R_BIN_LANG_JAVA:  demangled = r_bin_demangle_java (str); break;
case R_BIN_LANG_RUST:  demangled = r_bin_demangle_rust (bf, str, vaddr); break;
// ... 7 more hardcoded cases
case R_BIN_LANG_DLANG: demangled = r_bin_demangle_plugin (bin, "dlang", str); break;
}
```

Only dlang goes through the plugin path. All others are direct function calls with no
extension point.

### 2. Plugin mechanism abuses RBinPlugin (`libr/bin/demangle.c:58-69`)

`r_bin_demangle_plugin()` iterates `bin->plugins` (which are `RBinPlugin` — binary format
plugins) looking for a `.demangle` callback. An external plugin would need to export a full
`RBinPlugin` struct (687-byte struct with 30+ callbacks) just to provide a demangler.

### 3. Inconsistent signatures

Built-in demanglers have inconsistent function signatures — some take `RBinFile*`, some
don't, some take `vaddr`, some take extra flags. The plugin `.demangle` callback is
`char* (*demangle)(const char *str)` — the simplest form, missing context that some
demanglers need.

## Recommended Implementation: New `RDemanglePlugin` Type

Following the pattern established by `RMutaPlugin` (the most recent plugin type added).

### Step 1: Create RDemanglePlugin struct and r_demangle.h header

Define a new lightweight `RDemanglePlugin` struct with focused callbacks. This is the core
API contract for demangler plugins.

```c
// libr/include/r_demangle.h
typedef struct r_demangle_plugin_t {
    RPluginMeta meta;                // name="cxx", desc="C++ Itanium ABI demangler"
    const char *lang;                // language identifier matching RBinLanguage names
    bool (*check)(const char *str);  // return true if this plugin can handle the symbol
    char* (*demangle)(const char *str); // do the demangling
} RDemanglePlugin;

typedef struct r_demangle_t {
    RList /*RDemanglePlugin*/ *plugins;
} RDemangle;
```

Keep it minimal — the `check` callback allows auto-detection, `demangle` does the work.
No session/key/state complexity needed (unlike muta).

Reference: `libr/include/r_bin.h:671-677` (current `.demangle_type` and `.demangle` in RBinPlugin)

### Step 2: Add R_LIB_TYPE_DEMANGLE to plugin type system

Register the new plugin type in the RLib infrastructure so external `.so` plugins can be
loaded.

Changes required:

1. `libr/include/r_lib.h:117` — Add `R_LIB_TYPE_DEMANGLE,` before `R_LIB_TYPE_LAST`
2. `libr/util/lib.c` — Add `"demangle"` to the `r_lib_types[]` string array
3. `libr/core/libs.c:33` — Add `CB(demangle, demangle)` callback macro
4. `libr/core/libs.c:103` — Add `DF(DEMANGLE, "demangle plugins", demangle);` handler registration
5. `libr/config.h` — Add `R_DEMANGLE_STATIC_PLUGINS` macro listing built-in demanglers

Reference: `libr/include/r_lib.h:99-119`

### Step 3: Refactor r_bin_demangle() to iterate RDemanglePlugin list

Replace the hardcoded switch statement with a plugin iteration loop. This is the key change
that makes the system extensible.

In `libr/bin/demangle.c:184-194`, replace the switch with:

```c
RDemanglePlugin *dp;
RListIter *it;
r_list_foreach (bin->demangle->plugins, it, dp) {
    if (dp->check && dp->check (str)) {
        demangled = dp->demangle (str);
        if (demangled) {
            break;
        }
    }
}
// fallback: try by language name match if type was explicitly specified
if (!demangled && def) {
    r_list_foreach (bin->demangle->plugins, it, dp) {
        if (dp->lang && !strcmp (dp->lang, def)) {
            demangled = dp->demangle (str);
            if (demangled) {
                break;
            }
        }
    }
}
```

This preserves backward compatibility — existing demanglers become built-in plugins with
`check` functions matching the current prefix detection logic in `blang.c`.

Reference: `libr/bin/demangle.c:184-194`

### Step 4: Convert existing demanglers to RDemanglePlugin instances

Wrap each existing demangler (cxx, java, rust, objc, swift, msvc, pascal) as an
`RDemanglePlugin` struct. They stay in `libr/bin/mangling/` but now export plugin structs.

Example for C++:

```c
// libr/bin/mangling/cxx.c
static bool cxx_check(const char *str) {
    return r_str_startswith (str, "_Z") || r_str_startswith (str, "__Z");
}

static char *cxx_demangle(const char *str) {
    return r_bin_demangle_cxx (NULL, str, 0);
}

RDemanglePlugin r_demangle_plugin_cxx = {
    .meta = {
        .name = "cxx",
        .desc = "C++ Itanium ABI demangler",
        .license = "GPL-2.0",
    },
    .lang = "c++",
    .check = cxx_check,
    .demangle = cxx_demangle,
};
```

Repeat for all 7 built-in languages. The `check` functions should be extracted from the
existing detection logic in `blang.c:14-83`.

Reference: `libr/bin/mangling/cxx.c:1-10`

### Step 5: Update iD command and r_bin_demangle_list() for new plugin system

The `iD` command listing (`libr/bin/demangle.c:28-56`) has a hardcoded `langs[]` array.
Replace it with iteration over the plugin list so new plugins automatically appear.

```c
R_API void r_bin_demangle_list(RBin *bin) {
    RDemanglePlugin *plugin;
    RListIter *it;
    r_list_foreach (bin->demangle->plugins, it, plugin) {
        bin->cb_printf ("%s\n", plugin->meta.name);
    }
}
```

Reference: `libr/bin/demangle.c:28-56`

### Step 6: Add r2skel template for external demangler plugins

The issue author specifically requested an example in r2skel. An external demangler plugin
`.c` template would look like:

```c
#include <r_demangle.h>
#include <r_lib.h>

static bool my_check(const char *str) {
    return r_str_startswith (str, "_My");
}

static char *my_demangle(const char *str) {
    // custom demangling logic
    return strdup (str + 3);
}

RDemanglePlugin r_demangle_plugin_example = {
    .meta = {
        .name = "example",
        .desc = "Example demangler",
    },
    .lang = "example",
    .check = my_check,
    .demangle = my_demangle,
};

#ifndef R2_PLUGIN_INCORE
R_API RLibStruct radare_plugin = {
    .type = R_LIB_TYPE_DEMANGLE,
    .data = &r_demangle_plugin_example,
    .version = R2_VERSION,
};
#endif
```

Reference: `libr/core/libs.c:87-104`

## Key Design Decisions

| Decision | Recommendation | Rationale |
|----------|---------------|-----------|
| New library (`libr/demangle/`) vs keep in `libr/bin/` | Keep in `libr/bin/` initially | Demanglers are tightly coupled to bin analysis; avoid library proliferation. Add `RDemangle*` to `RBin` struct. |
| Plugin signature | `char* (*demangle)(const char *str)` | Simple, stateless. Demanglers that need `RBinFile` context (like swift's `trylib`) can access it through plugin `user` data or a future extended callback. |
| `check` vs language enum dispatch | Both — `check` for auto-detect, `lang` field for explicit `iD lang sym` | Preserves both use cases: auto-detection during symbol import and explicit user-driven demangling. |
| Remove `RBinPlugin.demangle` | Deprecate, keep for 1 release cycle | Existing external plugins using the old mechanism shouldn't break immediately. |

## Files That Need Changes

| File | Change |
|------|--------|
| `libr/include/r_demangle.h` (new) | `RDemanglePlugin`, `RDemangle` struct definitions |
| `libr/include/r_lib.h` | Add `R_LIB_TYPE_DEMANGLE` enum value |
| `libr/include/r_bin.h` | Add `RDemangle *demangle` to `RBin` struct |
| `libr/util/lib.c` | Add `"demangle"` to type string array |
| `libr/core/libs.c` | Add CB/DF registration for demangle plugins |
| `libr/config.h` | Add `R_DEMANGLE_STATIC_PLUGINS` macro |
| `libr/bin/demangle.c` | Refactor dispatcher, update list function |
| `libr/bin/mangling/cxx.c` | Export `RDemanglePlugin` struct |
| `libr/bin/mangling/java.c` | Export `RDemanglePlugin` struct |
| `libr/bin/mangling/rust.c` | Export `RDemanglePlugin` struct |
| `libr/bin/mangling/objc.c` | Export `RDemanglePlugin` struct |
| `libr/bin/mangling/swift-sd.c` | Export `RDemanglePlugin` struct |
| `libr/bin/mangling/msvc.c` | Export `RDemanglePlugin` struct |
| `libr/bin/mangling/pascal.c` | Export `RDemanglePlugin` struct |
| `libr/bin/bin.c` | Initialize `RDemangle`, register static plugins |
| `libr/core/cmd_info.inc.c` | Update `iD` command to use new plugin API |

## Currently Supported Demanglers

| Language | File | Signature |
|----------|------|-----------|
| C++ | `libr/bin/mangling/cxx.c` | `r_bin_demangle_cxx(RBinFile*, const char*, ut64)` |
| Java | `libr/bin/mangling/java.c` | `r_bin_demangle_java(const char*)` |
| Rust | `libr/bin/mangling/rust.c` | `r_bin_demangle_rust(RBinFile*, const char*, ut64)` |
| Objective-C | `libr/bin/mangling/objc.c` | `r_bin_demangle_objc(RBinFile*, const char*)` |
| Swift | `libr/bin/mangling/swift-sd.c` | `r_bin_demangle_swift(const char*, bool, bool)` |
| MSVC | `libr/bin/mangling/msvc.c` | `r_bin_demangle_msvc(const char*)` |
| FreePascal | `libr/bin/mangling/pascal.c` | `r_bin_demangle_freepascal(const char*)` |
| D lang | via `r_bin_demangle_plugin()` | `RBinPlugin.demangle(const char*)` |
