# sdb Open Issues Review

*Generated 2026-03-18 — 25 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 4 | 16% |
| 🗑️ obsolete | 2 | 8% |
| 🔧 still_open | 19 | 76% |
| **Total** | **25** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🟢 5 | 1 | 0 | 1 |
| 🔵 4 | 2 | 2 | 4 |
| 🟡 3 | 1 | 0 | 1 |

## ✅ Likely Resolved (4)

### Confidence 🟢 5 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#104](https://github.com/radareorg/sdb/issues/104) — feature request sdb_merge (Sdb *dst, sdb*src)
*9y old · 0 comments*

sdb_merge() is declared in sdb.h and implemented in sdb.c. There is also a test file test/merge.c. The function merges all records from src into dst, exactly as requested.

---

</details>

### Confidence 🔵 4 (2)

<details>
<summary>Click to expand 2 issues</summary>

#### [#123](https://github.com/radareorg/sdb/issues/123) — Hashtable depends on sdb
*9y old · 0 comments*

The ht_pp.h, ht_pu.h, ht_su.h, ht_up.h, ht_uu.h files provide independent hashtable implementations with configurable function pointers (DupKey, DupValue, CalcSize, HashFunction, ListComparator). The hashtable is now generic and does not require string-only keys/values. The SdbHt wrapper has been removed. This addresses the request for an independent, agnostic hashtable.

---

#### [#152](https://github.com/radareorg/sdb/issues/152) — Kill SdbHt or Ht
*7y old · 4 comments*

The SdbHt type no longer appears in the current codebase (only in a fuzz test output file). The ht_pp.h header defines HtPP (and HtPU, HtSU, HtUP, HtUU) as the generic hashtable implementations. The old SdbHt wrapper has been removed in favor of the templated Ht* types, which is what was requested.

---

</details>

### Confidence 🟡 3 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#114](https://github.com/radareorg/sdb/issues/114) — sdb_array_insert chop strings sometimes
*9y old · 1 comments*

The issue states it was fixed in commit b71e11fcd773422ecb105935e5c826f97bc7364b but lacked tests. The sdb_array_insert function in array.c has been substantially reworked since 2016. However, the original author noted tests were needed and none were added specifically for this bug.

---

</details>

## 🗑️ Obsolete (2)

### Confidence 🔵 4 (2)

<details>
<summary>Click to expand 2 issues</summary>

#### [#63](https://github.com/radareorg/sdb/issues/63) — push/pop methods should be _head and _tail like in GLib?
*11y old · 0 comments*

This is a 2014 API naming discussion with no body text. After 12 years with no action or further discussion, and given that the SDB API has stabilized, this naming bikeshed is obsolete.

---

#### [#94](https://github.com/radareorg/sdb/issues/94) — Evaluate the use of warp's hash string algorithm (in D)
*10y old · 0 comments*

This was a 2016 investigation ticket about evaluating an alternative hash function from Facebook's warp project. The issue author noted 'right now we have no reproducible collisions in the current string hashing algorithm used.' After 10 years with no action and no demonstrated need, this evaluation task is obsolete.

---

</details>

## 🔧 Still Open (19)

### Confidence 🟢 5 (4)

<details>
<summary>Click to expand 4 issues</summary>

#### [#4](https://github.com/radareorg/sdb/issues/4) — Add checksum to verify cdb contents
*12y old · 0 comments*

No checksum verification has been added to the CDB file format. The cdb.c, disk.c, and related files contain no CRC, hash, or checksum logic for verifying file integrity. This is closely related to issue #24 (footer) and neither has been implemented.

---

#### [#24](https://github.com/radareorg/sdb/issues/24) — Add footer in sdb files (cdb)
*11y old · 0 comments*

Searched for footer, checksum, crc16 implementations in the CDB format code and found none. The disk format remains the standard CDB format without any footer containing magic number, version, timestamp, or checksum.

---

#### [#87](https://github.com/radareorg/sdb/issues/87) — Do not use 'ln' in case of mingw/cygwin
*10y old · 0 comments*

The Makefiles still extensively use 'ln -fs' commands (over 20 occurrences in the main Makefile and sub-Makefiles). No conditional logic exists to use 'cp' or alternatives on mingw/cygwin platforms. This remains unfixed.

---

#### [#184](https://github.com/radareorg/sdb/issues/184) — SDB Logo?
*7y old · 2 comments*

This is a request for a logo/artwork for the SDB project's entry in the database encyclopedia (dbdb.io). There is no logo file in the repository. This is a non-technical community/design request that remains open after 7+ years.

---

</details>

### Confidence 🔵 4 (7)

<details>
<summary>Click to expand 7 issues</summary>

#### [#18](https://github.com/radareorg/sdb/issues/18) — Cannot open ../../ namespaces or define sandbox rules
*12y old · 0 comments*

The namespace system in ns.c does not appear to have path traversal restrictions or sandbox rules. The issue about preventing directory traversal in namespace paths and defining sandbox rules remains unaddressed.

---

#### [#40](https://github.com/radareorg/sdb/issues/40) — Add support for queries like 'pe.*'
*11y old · 0 comments*

Searched the query parser and found no wildcard/glob pattern matching for key queries. The sdb query language does not support 'pe.*' style wildcard expansion to return all matching key=value pairs.

---

#### [#91](https://github.com/radareorg/sdb/issues/91) — Support cross directory compilations
*10y old · 0 comments*

The Makefiles do not use VPATH or support out-of-tree builds. The only srcdir references are in the nodejs binding's auto-generated Makefile. The main build system still requires in-tree compilation. Meson support exists as an alternative but the Makefile issue remains.

---

#### [#161](https://github.com/radareorg/sdb/issues/161) — Implement sliced memory and use it to store iterators in ls.c
*7y old · 0 comments*

No sliced/slab memory allocator has been added to ls.c or anywhere in sdb. The ls.c still uses standard malloc-based linked list nodes. This optimization feature request remains unimplemented.

---

#### [#195](https://github.com/radareorg/sdb/issues/195) — Spaces should be trimmed in sdb query
*6y old · 0 comments*

Searched query.c for any space-trimming logic around key=value parsing and found none. The issue reports that 'foo = bar' (with spaces around =) fails to set the key, while 'foo= bar' works. No trimming has been added to the query parser.

---

#### [#199](https://github.com/radareorg/sdb/issues/199) — Provide public sdb_ref sdb_unref
*6y old · 1 comments*

The ns.c source has a comment 'r->refs++; // sdb_ref / sdb_unref //' but no public API functions sdb_ref/sdb_unref are declared in sdb.h. The refcounting is internal only, and the requested public API was never implemented. The feature request to expose reference counting for better memory management in radare2 remains unaddressed.

---

#### [#301](https://github.com/radareorg/sdb/issues/301) — Makefile: Vala code generated tries to include sdb.h at wrong path
*4mo old · 2 comments*

The vala bindings still exist in bindings/vala/ and the Makefiles are unchanged. The issue is about the Makefile build path for vala-generated C files including sdb.h at the wrong path. The maintainer could not reproduce but acknowledged it might be related to build order (bindings expecting sdb to be installed first). Meson builds are unaffected. The Makefile for vala bindings has not been modified to fix include paths.

---

</details>

### Confidence 🟡 3 (8)

<details>
<summary>Click to expand 8 issues</summary>

#### [#11](https://github.com/radareorg/sdb/issues/11) — sdb_file() should be deprecated or fixed. it's buggy in some situations
*12y old · 0 comments*

sdb_file() is still declared in sdb.h and presumably still in use. The function has not been deprecated or removed. Without specific test cases for the reported bugs, it's unclear if the buggy situations have been fixed, but the function persists without deprecation notices.

---

#### [#22](https://github.com/radareorg/sdb/issues/22) — Implement sdb slices support in query
*11y old · 2 comments*

sdb_aslice exists in util.c and is declared in sdb.h. However, the issue specifically asks for integration with query.c (the query language parser) so that '[2:3]foo' syntax works. A comment from 2014 confirms sdb_aslice exists but is 'Not yet integrated with query.c'. No query.c integration was found.

---

#### [#30](https://github.com/radareorg/sdb/issues/30) — Add human readable sdb_encode/decode
*11y old · 0 comments*

base64.c exists providing sdb_encode/sdb_decode functions, but these are standard base64 encoding. The issue asks for a human-readable encoding (possibly with compression). The current implementation appears to be standard base64 without the human-readable improvements described.

---

#### [#88](https://github.com/radareorg/sdb/issues/88) — Dump/Load nested SDBs in JSON format
*10y old · 1 comments*

There is a json/ subdirectory in src/ with json parsing/formatting utilities, and main.c has some JSON import/export functionality. However, the specific feature of dumping/loading nested SDBs (with automatic nesting of JSON objects into sub-namespaces) has not been fully implemented as described in the issue.

---

#### [#122](https://github.com/radareorg/sdb/issues/122) — cdb_init called too many times
*9y old · 1 comments*

cdb_init is still called in 3 locations: disk.c:167, sdb.c:103, and sdb.c:448. The issue from 2017 reported excessive cdb_init calls during piped operations. Without deeper analysis of the call paths it's unclear if the redundant calls have been eliminated, but the multiple call sites still exist.

---

#### [#126](https://github.com/radareorg/sdb/issues/126) — Implement Journal bulk mode
*9y old · 1 comments*

journal.c exists in the source tree, but the issue describes a performance optimization for bulk sdb_set() operations by using a linear buffer instead of individual hashtable lookups. The PoC patch shown was for radare2's dwarf loading, not sdb itself. No journal bulk mode API has been added to sdb.

---

#### [#140](https://github.com/radareorg/sdb/issues/140) — sdb fails to sync from mem to file
*8y old · 1 comments*

sdb_sync() exists in sdb.c and sdb_merge() exists as well. The original bug was about syncing an in-memory-only sdb to a new file path. Without a specific test case it's hard to confirm if the underlying CDB sync logic has been fixed. No test was ever added as requested by the maintainer.

---

#### [#173](https://github.com/radareorg/sdb/issues/173) — Implement generic ut64 k=v api
*7y old · 0 comments*

The dict.c file exists in the source tree providing some dictionary functionality, and ht_uu (hashtable ut64->ut64) exists in ht_uu.c. However, the issue specifically asks for a generic ut64 key=value API that is different from the dict implementation. The issue author (pancake) noted 'We have dict for this, but it shouldnt be the way to go.' The feature as envisioned has not been fully implemented.

---

</details>
