# radare2-extras Open Issues Review

*Generated 2026-03-18 — 38 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 4 | 10% |
| 🗑️ obsolete | 10 | 26% |
| 🔧 still_open | 24 | 63% |
| **Total** | **38** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🟢 5 | 3 | 1 | 4 |
| 🔵 4 | 1 | 6 | 7 |
| 🟡 3 | 0 | 2 | 2 |

## ✅ Likely Resolved (4)

### Confidence 🟢 5 (3)

<details>
<summary>Click to expand 3 issues</summary>

#### [#391](https://github.com/radareorg/radare2-extras/issues/391) — r2k: suppor linux kernel build with Clang
*16d old · 0 comments*

The single commit in the current repo (7a6e373) is titled 'fix(r2k): allow Linux module to compile using clang' and directly addresses this issue. The code in r2k/linux/arch/x86/x86_definitions.h now uses preprocessor macros (R2K_FORCE_INPUT, R2K_FORCE_ORDER) to handle the __FORCE_ORDER difference between GCC and Clang, replacing the bare __FORCE_ORDER in inline asm that Clang rejected.

---

#### [#304](https://github.com/radareorg/radare2-extras/issues/304) — Unicorn initialization parameters change for ARM
*4y old · 1 comments*

The unicorn debug plugin (libr/debug/p/debug_unicorn.c) now correctly uses 'uc_open(UC_ARCH_ARM64, UC_MODE_ARM, &uh)' for 64-bit and 'uc_open(UC_ARCH_ARM, UC_MODE_ARM, &uh)' for 32-bit, exactly matching the fix suggested in the issue. The old broken code 'uc_open(UC_ARCH_ARM, bits==64? UC_MODE_64: UC_MODE_32, &uh)' is no longer present.

---

#### [#138](https://github.com/radareorg/radare2-extras/issues/138) — Stack buffer overflow of parsing swf
*8y old · 4 comments · `swf`*

The dangerous 'memset(&header, 0, SWF_HDR_MIN_SIZE + rect_size_bytes)' call in r_bin_swf_get_header has been replaced. The current code (libr/bin/format/swf/swf.c line 49) uses 'r_buf_read_at(arch->buf, 0, (ut8*)&header, R_MIN(8, sizeof(header)))' which properly bounds the read to the struct size. The stack buffer overflow is fixed.

---

</details>

### Confidence 🔵 4 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#387](https://github.com/radareorg/radare2-extras/issues/387) — ppcdisasm plugin not in sync with rarch
*6mo old · 0 comments*

The ppcdisasm plugin (libr/asm/p/arch_ppc_disasm.c) has been ported to the new RArch plugin API. It uses RArchSession, RAnalOp, RArchDecodeMask, and R_ARCH_OP_MASK_DISASM — all consistent with the current rarch interface. The referenced radare2 issue #21782 complained about the plugin not being in sync; it now appears to be.

---

</details>

## 🗑️ Obsolete (10)

### Confidence 🟢 5 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#200](https://github.com/radareorg/radare2-extras/issues/200) — r2pm -i bpf fails: struct r_bin_plugin_t' has no member named 'load'
*6y old · 4 comments*

The BPF plugin code no longer exists in radare2-extras. No bpf-related files were found anywhere in the repository. BPF support was likely moved into radare2 core, as radare2 now has built-in BPF disassembly support. The plugin that was failing to build has been removed entirely.

---

</details>

### Confidence 🔵 4 (6)

<details>
<summary>Click to expand 6 issues</summary>

#### [#18](https://github.com/radareorg/radare2-extras/issues/18) — Add Google's ART disassembler asm plugins
*10y old · 6 comments*

The discussion concluded that all architectures targeted by ART are already supported by capstone (which r2 uses). Adding ART's C++ disassembler code would be redundant. The discussion effectively concluded this was unnecessary since capstone handles the same architectures. The ART disassembler would add no new capability.

---

#### [#201](https://github.com/radareorg/radare2-extras/issues/201) — Split repository into multiple
*6y old · 6 comments*

This was a proposal to split radare2-extras into multiple repos (extra-architectures, extra-formats, extra-misc). The discussion concluded with maintainers deciding against splitting, noting it would complicate r2pm and releases. The repo remains monolithic. This is a rejected organizational proposal, effectively obsolete.

---

#### [#234](https://github.com/radareorg/radare2-extras/issues/234) — Binary build all the plugins for r2pm-go in CI
*6y old · 1 comments*

This issue requested CI binary builds for r2pm-go. The referenced dependency (radareorg/r2pm#7) was for the old r2pm-go infrastructure. The repository has no CI configuration for this, and the r2pm package management has evolved significantly since 2020. The old r2pm-go approach appears abandoned in favor of the current r2pm system.

---

#### [#133](https://github.com/radareorg/radare2-extras/issues/133) — PyREBox Integration
*8y old · 3 comments*

PyREBox (Cisco-Talos/pyrebox) has been archived and is no longer maintained. The project was based on QEMU and Python 2. Integration would no longer be practical. The hypervisor-level debugging space has evolved with other tools like r2ghidra and Wenzel's microvmi project.

---

#### [#14](https://github.com/radareorg/radare2-extras/issues/14) — Loading gzipped yara rules is broken
*11y old · 1 comments · `Yara`*

The yara directory no longer exists in radare2-extras. Yara support has been moved/reorganized. The original gzip decompression bug for loading yara rules is moot since the yara plugin code is no longer present in this repository.

---

#### [#11](https://github.com/radareorg/radare2-extras/issues/11) — Add script to convert magic rule into yara one
*11y old · 0 comments · `Yara`*

The yara directory no longer exists in radare2-extras. Any yara-related feature requests are moot since yara support has been removed from this repository.

---

#### [#10](https://github.com/radareorg/radare2-extras/issues/10) — Creating Yara rule on the fly from the radare2 session
*11y old · 1 comments · `Yara`*

The yara directory no longer exists in radare2-extras. External tools like r2kit have implemented similar functionality independently. Yara support has been reorganized outside of this repository.

---

#### [#7](https://github.com/radareorg/radare2-extras/issues/7) — Add command/option to show yara results in JSON format
*11y old · 0 comments · `Yara`*

The yara directory no longer exists in radare2-extras. All yara-related issues are moot as the code has been removed from this repository.

---

</details>

### Confidence 🟡 3 (2)

<details>
<summary>Click to expand 2 issues</summary>

#### [#139](https://github.com/radareorg/radare2-extras/issues/139) — Makefile fatal issue when installing r2snow
*8y old · 3 comments*

The r2snowman directory still exists but contains a very minimal Makefile. The snowman decompiler project (the upstream dependency) has been largely abandoned and superseded by other decompilers. The git clone issue reported (cloning into existing directory) is a minor Makefile bug in what is now an obsolete tool.

---

#### [#13](https://github.com/radareorg/radare2-extras/issues/13) — Add all radare2-extras stuff to the coverity build
*11y old · 1 comments · `enhancement`*

This 2015 request to add radare2-extras to Coverity static analysis builds is outdated. The build system has changed significantly, many plugins have been removed or moved, and Coverity integration for the extras repo was never set up. Modern CI/CD practices have largely superseded this specific request.

---

</details>

## 🔧 Still Open (24)

### Confidence 🟢 5 (5)

<details>
<summary>Click to expand 5 issues</summary>

#### [#326](https://github.com/radareorg/radare2-extras/issues/326) — Microblaze compilation error
*2y old · 1 comments*

The microblaze anal plugin (microblaze/anal/anal_microblaze_gnu.c) still uses R_ANAL_OP_MASK_DISASM on line 1144 (should be R_ARCH_OP_MASK_DISASM) and .set_reg_profile with a return type mismatch. The plugin has NOT been ported to the new RArch API like blackfin was. The exact errors reported in the issue would still occur.

---

#### [#253](https://github.com/radareorg/radare2-extras/issues/253) — Integer overflow when disassembling swp files
*5y old · 0 comments · `swf`*

The vulnerable code in libr/asm/arch/swf/swfdis.c still contains the exact integer overflow: 'ut8 max = strsize*(len/2);' on line 78, where strsize=20 and len is a ut16 read from user input, but max is a ut8 that overflows beyond 0xFF. The subsequent malloc(max) allocates too little memory, leading to a heap overflow via strcat/strncpy.

---

#### [#170](https://github.com/radareorg/radare2-extras/issues/170) — Move xc800 plugin here from wolfvan repo + r2pm
*7y old · 0 comments*

No xc800-related files exist anywhere in the radare2-extras repository. The plugin was never moved from the external wolfvan/radare2_xc800 repo as requested.

---

#### [#155](https://github.com/radareorg/radare2-extras/issues/155) — Support sleuthkit with r2
*7y old · 0 comments*

No sleuthkit-related code exists in the repository. This feature request to integrate The Sleuth Kit for forensic filesystem analysis was never implemented.

---

#### [#118](https://github.com/radareorg/radare2-extras/issues/118) — Implement plugins for naked-asm
*8y old · 5 comments · `easy`*

No naken_asm related code exists in the repository. The proposed integration with mikeakohn/naken_asm was never implemented despite discussion between the maintainers.

---

#### [#104](https://github.com/radareorg/radare2-extras/issues/104) — Use qAPI for emulation
*9y old · 1 comments*

No QEMU QAPI integration code exists in the repository. This feature request to use QEMU's QAPI for emulation was never implemented.

---

#### [#50](https://github.com/radareorg/radare2-extras/issues/50) — feature: 6800 plugin
*9y old · 0 comments*

No Motorola 6800 (distinct from 68000/m68k) plugin exists in the repository. This feature request was never implemented.

---

#### [#5](https://github.com/radareorg/radare2-extras/issues/5) — Implement debugger for LibVMI
*11y old · 2 comments · `debug`*

No LibVMI integration code exists in the repository. This feature request from 2015 to implement KVM/XEN/QEMU debugging via libvmi.com was never implemented, despite interest from multiple parties including the microvmi/kvm-vmi developer.

---

</details>

### Confidence 🔵 4 (7)

<details>
<summary>Click to expand 7 issues</summary>

#### [#262](https://github.com/radareorg/radare2-extras/issues/262) — Load structures and types from IDB files in r2idb.py
*5y old · 0 comments*

The r2ida/ida2r2/ida2r2.py script is still present but does not appear to have been updated to load structures and types from IDB files. The issue references upstream python-idb issues/PRs. This is a feature request that has not been implemented.

---

#### [#219](https://github.com/radareorg/radare2-extras/issues/219) — ida2r2 error
*6y old · 0 comments · `bug`*

The r2ida/ida2r2/ida2r2.py script still calls 'api.idautils.Names()' on line 140, which is the exact line that causes the AttributeError reported in the issue. The python-idb library's idautils wrapper may not implement Names(). No fix has been applied.

---

#### [#157](https://github.com/radareorg/radare2-extras/issues/157) — r2k-xnu
*7y old · 0 comments*

The r2k/darwin directory exists with a basic Makefile and main.c, but it is a minimal skeleton. The issue requested full r2k support on macOS (XNU) with /dev/kmem access. The darwin implementation is incomplete and the various referenced external projects (kextstat_aslr, Kmem) remain unmaintained. This feature request is largely unfulfilled.

---

#### [#148](https://github.com/radareorg/radare2-extras/issues/148) — Better crashlog parsers
*8y old · 1 comments*

The crashlog directory contains only r2crash.js and a twitter.log file. No Windows, Android, or iOS crash log parsers have been added as requested in this issue.

---

#### [#120](https://github.com/radareorg/radare2-extras/issues/120) — Read/Write DRX on r2k-x86
*8y old · 0 comments*

The r2k/linux directory contains the kernel module code, and the r2k/linux/arch/x86 directory has x86-specific definitions. However, there is no evidence of debug register (DRX/DR0-DR7) read/write support being implemented in the r2k x86 code.

---

#### [#106](https://github.com/radareorg/radare2-extras/issues/106) — Add scripts to detect if packed/obfuscated
*9y old · 0 comments*

No packing/obfuscation detection scripts were found in the repository. The issue requested combining iI, p=, iS entropy, yara, and ssdeep data to detect packed/obfuscated binaries. This feature was never implemented.

---

#### [#16](https://github.com/radareorg/radare2-extras/issues/16) — libc detector for static binaries
*10y old · 0 comments*

No libc detection scripts or zignature-based libc identification tools were found in the repository. Radare2 core has since gained FLIRT signature support and zignatures, but no dedicated 'libc detector for static binaries' tool exists in extras.

---

#### [#4](https://github.com/radareorg/radare2-extras/issues/4) — Adding MyCPU and a demo file format examples
*11y old · 0 comments · `enhancement`*

No MyCPU example plugin or demo file format example was found in the repository. The examples directory exists but does not contain the requested tutorial/demo plugins for teaching how to implement new architectures and formats in radare2-extras.

---

</details>

### Confidence 🟡 3 (7)

<details>
<summary>Click to expand 7 issues</summary>

#### [#267](https://github.com/radareorg/radare2-extras/issues/267) — spc700
*5y old · 3 comments*

The spc700 plugin exists in libr/asm/p/arch_spc700.c and has been ported to the RArch API with a decode function. However, the issue requested much more: reg profile, calling conventions, analysis, emulation, parsing, and assembler support. The plugin is still disassembly-only. The issue tracks improving spc700 to full support, which has not been achieved.

---

#### [#191](https://github.com/radareorg/radare2-extras/issues/191) — disassembly error
*7y old · 2 comments · `evm`*

The EVM plugin still exists (libr/asm/p/arch_evm.c) and appears to still use the older RAsm plugin API rather than RArch. The issue reports that delegatecall into another contract shows wrong disassembly, which is a logic bug in how the EVM debugger handles cross-contract calls. No evidence of a fix for this specific behavior.

---

#### [#160](https://github.com/radareorg/radare2-extras/issues/160) — r2tox install error don't know how to fix it
*7y old · 0 comments*

The r2tox directory still exists with its client.c and Makefile. The build errors reported (missing pthread linkage, incompatible pointer types) are likely still present since the code appears unchanged. The r2tox plugin depends on building c-toxcore which requires specific linking flags.

---

#### [#80](https://github.com/radareorg/radare2-extras/issues/80) — m68k negative branch produces invalid instruction
*9y old · 0 comments*

The m68k disassembler plugin exists at libr/asm/p/arch_m68k_net.c with the underlying disassembler in libr/asm/arch/m68k/m68k_disasm/. Without being able to test the specific instruction (bpl.w with negative offset), it is unclear if this specific bug was fixed. The code is still present and appears to have been maintained but the specific bug fix cannot be confirmed from code inspection alone.

---

#### [#58](https://github.com/radareorg/radare2-extras/issues/58) — Feature: 6809 plugin
*9y old · 0 comments*

The MC6809 plugin exists at libr/asm/p/arch_mc6809.c and has been ported to the RArch API. The original checklist had: Disassembly (done), Assembly (not done), Analysis (done per checklist), ESIL (not done), r2pm definition (done per checklist). Assembly and ESIL remain unimplemented, so this feature is partially complete but the issue remains open for those items.

---

#### [#52](https://github.com/radareorg/radare2-extras/issues/52) — feature: unicorn non x86 versions
*9y old · 1 comments*

The unicorn debug plugin (libr/debug/p/debug_unicorn.c) does contain ARM and ARM64 initialization code now (UC_ARCH_ARM, UC_ARCH_ARM64), so ARM support was added. However, the issue also mentions MIPS and other architectures. Without full code review of all architecture support, partial progress is evident but the full request may not be complete.

---

#### [#40](https://github.com/radareorg/radare2-extras/issues/40) — Add r2pm for ramoji2
*9y old · 0 comments · `easy`*

The ramoji2 directory exists with index.js and package.json. Whether an r2pm package definition was created for it is unclear from the repo alone, as r2pm definitions may live in a separate r2pm packages database. The plugin code exists but the r2pm integration status is uncertain.

---

#### [#31](https://github.com/radareorg/radare2-extras/issues/31) — unicorn can't write to stack
*10y old · 3 comments*

The unicorn debug plugin still relies on explicit memory synchronization between r2 and unicorn (via dpa). The fundamental limitation described — that unicorn cannot share memory with r2 — is an architectural constraint of the unicorn engine. While newer unicorn versions may have improved mmap APIs, the r2 plugin may not have been updated to take advantage of them.

---

#### [#29](https://github.com/radareorg/radare2-extras/issues/29) — unicorn doesn't map loaded libraries
*10y old · 0 comments · `debug`*

The unicorn debug plugin handles memory mapping when initializing emulation, but mapping dynamically loaded libraries is a fundamental challenge with unicorn-based emulation. No evidence of a comprehensive solution for automatically mapping shared libraries.

---

</details>

### Confidence 🟠 2 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#22](https://github.com/radareorg/radare2-extras/issues/22) — Implement disSssembler and analyzer plugin for lua
*10y old · 4 comments*

A lua53 directory exists in the repository with anal, asm, and bin subdirectories, suggesting some Lua 5.3 bytecode support was implemented. However, the original issue from 2015 did not specify a Lua version. The lua53 plugin may partially address this issue, but completeness (analysis, ESIL) is unknown without deeper inspection.

---

</details>
