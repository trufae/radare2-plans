# JNI support roadmap

This is the implementation roadmap for JNI-aware analysis in radare2. It is
based on the behavior documented in `doc/jni.md` and on tests with the ARM64
libraries in `../jni`.

Every actionable item has a stable identifier. Do not renumber identifiers when
items move, complete, or become obsolete. Mark obsolete items as superseded and
refer to the replacement identifier.

Priority meanings:

* **P0**: required for JNI analysis to produce materially useful results on the
  current samples;
* **P1**: correctness, usability, and broader static-analysis coverage;
* **P2**: advanced, runtime-dependent, or less common cases.

## Recommended implementation order

The first useful vertical slice is `JNI-001` -> `JNI-002` -> `JNI-003`. It can
recover 379 dynamically registered methods from the RenderScript and Conscrypt
samples without requiring perfect `JNI_OnLoad` emulation.

The second slice is `JNI-005` -> `JNI-006` -> `JNI-007` -> `JNI-008`. It turns
anonymous JNI `callreg` operations into named and typed calls in disassembly and
`pdc`.

The third slice is `JNI-009` -> `JNI-010`, followed by DEX correlation in
`JNI-014` and `JNI-015`.

## P0: recover Java-callable native APIs

- [x] **JNI-001 (P0): Implement a strict JVM/JNI method-descriptor parser**

  Completed 2026-08-21. `r_bin_java_member_parse()` returns one structured
  class/field/method result with Java and JNI type names, object class names,
  array dimensions, JVM slots, visibility, and normalized attributes. The Java
  class loader and `rabin2 -D java` now share it; malformed descriptors are
  covered by unit tests.

  Deliver a reusable parser that returns structured argument and return types
  for descriptors such as `(ZILjava/lang/String;)Z`. It must support all
  primitive types, objects, nested arrays, multidimensional arrays, and `void`
  only as a return type. It must reject truncated objects, invalid escapes,
  missing parentheses, trailing garbage, and invalid `void` arguments.

  Keep the parser independent of ELF and DEX loaders so exported JNI names,
  `JNINativeMethod` recovery, DEX, and `rabin2 -D java` can share it. Fix the
  malformed current output for `nativeGreater(J[J[JJ)V` as part of adopting the
  parser.

  Acceptance criteria:

  * structured results preserve object class names and array dimensions;
  * callers can request Java display names or radare2 C/JNI type names;
  * malformed input has unit tests and never returns a partial valid signature;
  * `rabin2 -D java` produces valid output for the descriptors observed in all
    four samples.

  Dependencies: none.

- [x] **JNI-002 (P0): Discover constant `JNINativeMethod` arrays**

  Completed 2026-08-21. The `anal.jni` pre-analysis plugin runs from `aa` for
  binaries tagged `R_BIN_LANG_JNI`, validates pointer-aligned native method
  records with the JNI-001 parser, and stores tables and parsed signatures in
  the `anal/jni` SDB namespace. Tests cover malformed pointer triples, the
  two-record threshold, 32/64-bit pointers, and target endianness.

  Implement this native-side analysis in a dedicated
  `libr/anal/p/anal_jni.c` plugin. Do not put it in `anal_java.c`, which is for
  Java class and bytecode analysis. Run the JNI plugin from `aa` only when the
  current bin object reports `R_BIN_LANG_JNI`.

  Scan relocated data and read-only data for runs of pointer-width triples where
  the first pointer references a plausible method name, the second references a
  descriptor accepted by `JNI-001`, and the third references executable code.
  Require at least two consecutive records by default to reduce false positives.

  The scanner must use target endianness and pointer width, honor mapped
  addresses and relocations, stop safely at invalid/unmapped data, and avoid
  scanning every byte when section alignment provides a tighter candidate set.

  Acceptance criteria:

  * find 93 records at `0x20008` in `librsjni.so`;
  * find 286 records at `0x215010` in `libconscrypt_jni.so`;
  * report no table in `libimage_processing_util_jni.so`;
  * expose table address, record count, name, descriptor, and function address
    in an internal representation consumable by `JNI-003` and `JNI-004`;
  * include negative fixtures containing string/function pointer triples that
    are not valid JNI tables.

  Dependencies: `JNI-001`.

- [ ] **JNI-003 (P0): Materialize recovered native methods in analysis**

  For every record recovered by `JNI-002`, create or update the function at
  `fnPtr`, add data-to-code xrefs, retain the Java method name and descriptor,
  and apply descriptor-derived argument and return types after the implicit
  `JNIEnv *` and receiver arguments.

  Do not overwrite a better DWARF, user, or exact symbol prototype. The
  descriptor cannot distinguish `jobject` from `jclass`, so preserve that
  uncertainty until `JNI-015` supplies Java declaration metadata.

  Acceptance criteria:

  * `afl`, `pdf`, and `pdc` expose all 379 recovered sample methods;
  * overloads remain distinct even when their Java method names match;
  * descriptor-derived arrays and object references use JNI-compatible types;
  * recovered metadata survives project save/load;
  * rerunning analysis is idempotent and does not duplicate flags or xrefs.

  Dependencies: `JNI-001`, `JNI-002`.

- [ ] **JNI-004 (P0): Recognize `RegisterNatives` and associate class names**

  Recognize calls through `JNINativeInterface.RegisterNatives`, recover the
  constant `JNIEnv *`, `jclass`, table pointer, and count when possible, and
  connect the call site with a table found by `JNI-002`. Trace the `jclass` back
  to a nearby `FindClass` result and its class-name string.

  Direct ARM64 patterns must work before adding interprocedural cases. A table
  found without a recoverable class must still be materialized by `JNI-003` with
  an anonymous/unknown class rather than discarded.

  Acceptance criteria:

  * associate the RenderScript table with
    `android/support/v8/renderscript/RenderScript`;
  * validate that the immediate count `0x5d` matches the discovered table;
  * emit a diagnostic, without aborting analysis, when a call-site count and
    scanned table length disagree;
  * add call-site/table/class xrefs or equivalent durable metadata.

  Dependencies: `JNI-002`, `JNI-007`. Interprocedural Conscrypt support depends
  on `JNI-016`.

## P0: resolve JNI dispatch-table calls

- [ ] **JNI-005 (P0): Add typed definitions for high-value JNI table slots**

  Replace `void *` members with function-pointer types for the slots required by
  the samples and registration analysis. The initial set must include JavaVM
  `GetEnv` plus JNIEnv `FindClass`, `GetMethodID`, `GetStaticMethodID`,
  `RegisterNatives`, `NewStringUTF`, `GetStringUTFChars`,
  `ReleaseStringUTFChars`, `GetArrayLength`, `GetDirectBufferAddress`, and the
  `Call*Method` families.

  Preserve exact member order. Include `JNIEnv *` as the first argument of each
  JNIEnv operation and `JavaVM *` as the first argument of each JavaVM
  operation. Use variadic prototypes only for the actual variadic JNI slots;
  keep the `V` and `A` variants distinct.

  Acceptance criteria:

  * type queries return a function type, arguments, and return type for every
    initial slot;
  * `RegisterNatives` has the `JNINativeMethod *` and `jint` count arguments;
  * ARM64 offsets remain `0x30`, `0x6b8`, and `0x730` for the sample operations;
  * existing generic type tests continue to pass.

  Dependencies: none. Full table coverage is tracked by `JNI-018`.

- [ ] **JNI-006 (P0): Model `JNIEnv` and `JavaVM` interface indirection**

  Replace the opaque `void *` aliases with types that accurately describe the
  first interface-table dereference used by compiled JNI C and C++ code. The
  model must remain compatible with function signatures written as `JNIEnv *`
  and `JavaVM *` and must not add an extra or missing pointer level.

  Use a small compiled C/C++ fixture to validate both header modes. Ensure that
  the type system can follow `[env]` to `JNINativeInterface` and `[vm]` to
  `JNIInvokeInterface` on ARM64.

  Acceptance criteria:

  * `afvt` displays the expected public JNI argument spelling;
  * type propagation reaches the table member after the initial load;
  * C and C++ fixture code resolve to the same slot names;
  * no regressions occur in `JNI_OnLoad` and exported `Java_*` prototypes.

  Dependencies: `JNI-005`.

- [ ] **JNI-007 (P0): Automatically annotate JNI slot loads**

  Track a JNI interface pointer through register copies, the initial
  dereference, simple spills/reloads, and constant displacement loads. Resolve
  the displacement against the appropriate interface and attach the same
  structure-offset hint currently produced manually by `aht`.

  Receiver type must disambiguate identical offsets: `0x30` is JavaVM `GetEnv`
  and JNIEnv `FindClass`. Do not label a coincidental load from an untyped
  object merely because its displacement matches a JNI slot.

  Acceptance criteria:

  * annotate JavaVM `GetEnv` and JNIEnv `FindClass` in `librsjni.so`;
  * annotate `RegisterNatives` at `0x6b8`;
  * annotate all three `GetDirectBufferAddress` loads at `0x730` in
    `nativeShiftPixel`;
  * preserve annotations in `pdf` and `pdc`;
  * cover direct, copied, and spilled `JNIEnv *` values in tests.

  Dependencies: `JNI-005`, `JNI-006`.

- [ ] **JNI-008 (P0): Propagate JNI member signatures into indirect calls**

  Carry the function-pointer type from the slot load in `JNI-007` into the
  following indirect call. The call must retain the JNI operation name,
  argument types, return type, and call-convention behavior instead of appearing
  only as an anonymous `callreg`.

  Make the result available to argument analysis, return-value propagation,
  `pdf`, and `pdc`. Avoid inventing a concrete target address: the semantic
  callee is a table member, not a local function.

  Acceptance criteria:

  * `pdc` identifies `GetDirectBufferAddress` and its buffer argument in
    `nativeShiftPixel`;
  * `pdc` identifies `FindClass` and `RegisterNatives` in `librsjni.so`;
  * call argument comments no longer show unrelated stale values;
  * variadic, `V`, and `A` call families retain distinct prototypes;
  * analysis behaves safely when the member signature is missing.

  Dependencies: `JNI-005`, `JNI-007`.

## P0: exported `Java_*` methods

- [ ] **JNI-009 (P0): Decode exported JNI symbol names**

  Parse `Java_package_Class_method` and the optional `__arguments` overload
  suffix. Implement `_1`, `_2`, `_3`, and `_0xxxx` escapes and reject malformed
  encodings without truncating the original symbol. Preserve both the raw ELF
  symbol and structured class/method/argument metadata.

  Remember that the overload suffix contains only argument types. It does not
  encode the return type or whether the Java declaration is static.

  Acceptance criteria:

  * decode `Java_io_realm_internal_TableQuery_nativeGreater__J_3J_3JJ` into the
    correct class, method, and four Java arguments;
  * round-trip escaped underscores, arrays, objects, and Unicode escapes;
  * do not send `Java_*` names through the C++ demangler;
  * make structured results reusable by DEX correlation in `JNI-014`.

  Dependencies: `JNI-001` for parsing the decoded overload descriptor.

- [ ] **JNI-010 (P0): Merge JNI knowledge with recovered function prototypes**

  Replace the unconditional generic
  `void *jni_native(JNIEnv *env, jobject thiz)` behavior with a non-destructive
  merge. Always type the implicit `JNIEnv *`; use an unknown receiver when
  staticness is unavailable; append decoded overload arguments when available;
  and retain arguments already recovered by `afva` when no descriptor exists.

  Never claim a `void *` return merely because the symbol begins with `Java_`.
  Prefer exact DWARF, user, dynamically registered, or DEX-derived prototypes
  over inferred JNI metadata.

  Acceptance criteria:

  * the Realm `nativeGreater` function displays its four Java arguments after
    the two implicit JNI arguments;
  * `nativeShiftPixel` no longer loses the arguments already observed in
    registers and stack slots;
  * user-applied `afs` signatures are not overwritten by later `aaa` runs;
  * unknown return types remain explicitly unknown until better evidence exists.

  Dependencies: `JNI-009`; exact return/static information depends on
  `JNI-014` and `JNI-015`.

## P0: regression fixtures and integration gate

- [ ] **JNI-011 (P0): Add small end-to-end JNI test binaries**

  Add sources to `radare2-testbins` for a static exported JNI library and a
  dynamically registered JNI library. Build at least ARM64 and x86 variants.
  Keep generated binaries out of the radare2 repository.

  The static fixture must contain an overloaded `Java_*` method, a method
  without an overload suffix, register and stack arguments, and at least two
  distinct JNIEnv calls. The dynamic fixture must use `JNI_OnLoad`, `GetEnv`,
  `FindClass`, and a constant `JNINativeMethod` table.

  Acceptance criteria:

  * tests assert recovered class, method, descriptor, function address, and
    prototype;
  * tests assert JNI slot names in `pdf` and `pdc`;
  * tests cover 32-bit and 64-bit offsets;
  * every P0 implementation item adds or extends a focused test rather than
    depending on the large `../jni` samples.

  Dependencies: fixture creation can start immediately; expected outputs evolve
  with `JNI-001` through `JNI-010`.

## P1: correctness and usability

- [ ] **JNI-012 (P1): Make all JNI layouts pointer-size correct**

  Generate layouts from one ordered interface definition using the target
  pointer width, or split generic, 32-bit, and 64-bit type databases. Remove the
  current inconsistency where stored SDB offsets are 64-bit while `tsv`
  recomputes a different 32-bit layout.

  Acceptance criteria:

  * `FindClass` is at `0x18`, `RegisterNatives` at `0x35c`, and
    `GetDirectBufferAddress` at `0x398` on 32-bit targets;
  * the corresponding ARM64 offsets remain `0x30`, `0x6b8`, and `0x730`;
  * `ahts`, `tsv`, type size queries, and analysis agree;
  * x86, ARM32, MIPS32, and ARM64 tests assert representative offsets.

  Dependencies: coordinate with `JNI-005` and `JNI-006` so layouts have one
  source of truth.

- [ ] **JNI-013 (P1): Fix `JNINativeMethod` structured printing**

  Fix `r_type_format`/`tp` handling so `const char *` structure fields are read
  as pointer-width string pointers rather than inline strings. The current
  `JNINativeMethod` format becomes `zzp` and reads real tables incorrectly.

  Acceptance criteria:

  * `tp JNINativeMethod @ 0x20008` prints `nLoadSO`, its descriptor, and
    `0x2510` on the ARM64 sample;
  * one record consumes exactly 24 bytes on 64-bit and 12 bytes on 32-bit;
  * arrays of records can be printed without manual `pxq`/`ps` commands;
  * add a generic regression test because this bug affects non-JNI structures
    containing string pointers too.

  Dependencies: none.

- [ ] **JNI-014 (P1): Correlate ELF JNI methods with DEX/class declarations**

  Define a way to associate native ELF analysis with one or more loaded DEX,
  class, JAR, or APK objects. Match class name, method name, and descriptor while
  handling overloads and multiple ABI libraries.

  Acceptance criteria:

  * recover the full descriptor for a static `Java_*` symbol without an
    overload suffix;
  * detect ambiguous declarations instead of selecting one silently;
  * retain bidirectional xrefs between the Java declaration and native function;
  * work after project save/load.

  Dependencies: `JNI-001`, `JNI-009`.

- [ ] **JNI-015 (P1): Apply Java staticness and exact return information**

  Use `JNI-014` metadata to type the second native argument as `jclass` for
  static methods or `jobject` for instance methods and to apply the exact Java
  return type. Reconcile Java metadata with dynamic-table descriptors and report
  mismatches.

  Acceptance criteria:

  * static and instance fixtures display different receiver types;
  * exported methods without overload suffixes receive complete prototypes;
  * conflicting ELF/DWARF/DEX/table evidence follows a documented precedence
    order and preserves the discarded evidence for inspection.

  Dependencies: `JNI-003`, `JNI-010`, `JNI-014`.

- [ ] **JNI-016 (P1): Follow registration through helper functions**

  Extend `JNI-004` across simple wrapper calls, tail calls, register moves, and
  constant argument forwarding from `JNI_OnLoad`. Bound interprocedural analysis
  to avoid turning JNI recovery into unrestricted whole-program emulation.

  Acceptance criteria:

  * associate Conscrypt's table at `0x215010`, count `0x11e`, with
    `org/conscrypt/NativeCrypto` through its registration helper;
  * handle a wrapper shared by multiple class/table pairs;
  * stop safely on recursion, indirect wrappers, or non-constant arguments;
  * preserve anonymous tables found by `JNI-002` when class recovery fails.

  Dependencies: `JNI-004`, existing constant propagation/call analysis.

- [ ] **JNI-017 (P1): Expose recovered JNI metadata in commands and JSON**

  Provide a stable query surface for recovered classes, methods, descriptors,
  binding kind, table address, registration call site, native address, and
  confidence/source. Prefer extending an appropriate existing info/analysis
  command over creating a one-off command namespace.

  Acceptance criteria:

  * human and JSON output contain the same fields;
  * users can filter static exports, dynamic registrations, unresolved tables,
    and JNI call sites;
  * flags remain concise while raw class/method/descriptors remain accessible;
  * project serialization preserves the metadata.

  Dependencies: `JNI-003`, `JNI-004`, `JNI-009`.

- [ ] **JNI-018 (P1): Type every supported JNI interface slot**

  Expand `JNI-005` from the high-value subset to all JNIEnv and JavaVM slots
  supported by Android's JNI ABI. Generate definitions from a reviewed canonical
  list to avoid hand-maintained offset and prototype drift.

  Acceptance criteria:

  * every member has a function-pointer type except reserved slots;
  * member order and prototypes are checked against Android JNI headers;
  * variadic, `va_list`, and `jvalue *` variants are distinct;
  * representative return types and offsets are tested across both widths.

  Dependencies: `JNI-005`, `JNI-012`.

- [ ] **JNI-019 (P1): Bound table scanning and document confidence**

  Measure `JNI-002` on large C++ Android libraries and prevent excessive startup
  cost or false positives. Separate high-confidence tables confirmed by a
  `RegisterNatives` call from heuristic table candidates.

  Acceptance criteria:

  * scanning can be enabled/disabled independently if it is not cheap enough for
    default analysis;
  * JSON exposes confidence and validation reasons;
  * duplicate tables and aliases are coalesced without losing call sites;
  * performance tests cover a library at least as large as `librealm-jni.so`.

  Dependencies: `JNI-002`, `JNI-004`, `JNI-017`.

- [ ] **JNI-020 (P1): Keep user documentation synchronized with implementation**

  Update `doc/jni.md` as each vertical slice lands. Replace current-limitation
  text with actual command examples and keep a short troubleshooting section for
  stripped files, missing DEX metadata, and unresolved dynamic registration.

  Acceptance criteria:

  * every new command or configuration option has a tested example;
  * sample-specific addresses are clearly labeled;
  * limitations distinguish unsupported behavior from analysis that merely
    lacks sufficient evidence.

  Dependencies: ongoing.

## P2: advanced dynamic cases

- [ ] **JNI-021 (P2): Recover runtime-constructed native tables**

  Detect tables assembled in writable memory by stores, copies, constructors,
  or decoding routines before `RegisterNatives`. Reuse bounded ESIL/value-set
  analysis where possible rather than adding a JNI-only emulator.

  Acceptance criteria:

  * recover a small fixture that constructs three records at runtime;
  * attach confidence and the construction trace to each recovered value;
  * fail closed when a pointer or descriptor cannot be proven.

  Dependencies: `JNI-001`, `JNI-004`, `JNI-016`.

- [ ] **JNI-022 (P2): Support runtime capture of `RegisterNatives`**

  Define a debugger-assisted workflow that records actual class, method table,
  count, and function pointers when static recovery is insufficient. Keep this
  separate from default static analysis and make imported results reusable in a
  later non-debug session.

  Acceptance criteria:

  * capture a registration whose table is allocated at runtime;
  * import captured records into the same metadata model as `JNI-003`;
  * account for ASLR when mapping runtime pointers back to file addresses.

  Dependencies: `JNI-003`, `JNI-017`.

- [ ] **JNI-023 (P2): Model `UnregisterNatives` and repeated registration**

  Recognize `UnregisterNatives`, multiple registrations for one class, and
  replacement/alias tables. Represent lifecycle events without deleting the
  statically recovered history.

  Acceptance criteria:

  * list registration and unregistration call sites in order;
  * preserve all candidate bindings with their source and confidence;
  * avoid duplicate function creation when several records point to one native
    implementation.

  Dependencies: `JNI-004`, `JNI-017`.

- [ ] **JNI-024 (P2): Expand real-architecture coverage**

  Validate slot resolution, table scanning, and prototypes on ARM32, x86,
  x86-64, and MIPS samples in addition to ARM64. Include big-endian coverage if
  a maintained Android/JNI fixture is available.

  Acceptance criteria:

  * the same logical fixture produces equivalent recovered JNI metadata across
    architectures;
  * pointer widths, endianness, calling conventions, and stack arguments are
    asserted independently;
  * architecture-specific failures do not silently fall back to ARM64 offsets.

  Dependencies: `JNI-011`, `JNI-012`.

## Completed foundation

These items came from the review of commit `7a1c3636ac` ("Initial native
support for JNI") and are retained with identifiers for historical reference.

- [x] **JNI-D001:** Add the ordered `JNINativeInterface` structure and ARM64
  offsets from 0 through 1856.
- [x] **JNI-D002:** Add `JNINativeMethod` with `name`, `signature`, and `fnPtr`.
- [x] **JNI-D003:** Add the `jvalue` union members `z,b,c,s,i,j,f,d,l`.
- [x] **JNI-D004:** Add `JavaVM` and `JNIInvokeInterface` definitions.
- [x] **JNI-D005:** Add `JNI_OnLoad` and `JNI_OnUnload` function prototypes.
- [x] **JNI-D006:** Auto-apply a generic JNI type to `Java_*` symbols through
  type guessing and function-signature lookup.
- [x] **JNI-D007:** Expose built-in type loading through `tol types-jni`.
- [x] **JNI-D008:** Add initial tests for type loading, ARM64 offsets, `jvalue`,
  lifecycle prototypes, generic `Java_*` typing, and `tol`.

## Verified baseline and known caveats

* `libimage_processing_util_jni.so` exports 4 `Java_*` methods and uses JNIEnv
  slot `0x730` three times in `nativeShiftPixel`.
* `librealm-jni.so` exports 428 `Java_*` methods and 2 JNI lifecycle methods.
* `librsjni.so` registers 93 methods from `0x20008`.
* `libconscrypt_jni.so` registers 286 methods from `0x215010`.
* The stored SDB offsets are currently ARM64-specific even though JNI types are
  loaded on 32-bit Android targets.
* `afs` can show the generic type name `jni_native`, while `pdf` substitutes the
  actual function name.
* The earlier gperf lookup correction involving `r_str_replace_char` is valid
  and remains in place.
