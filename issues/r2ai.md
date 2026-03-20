# r2ai Open Issues Review

*Generated 2026-03-18 — 11 issues reviewed*

## Summary

| Status | Count | % |
|--------|------:|--:|
| ✅ likely_resolved | 5 | 45% |
| 🗑️ obsolete | 0 | 0% |
| 🔧 still_open | 6 | 54% |
| **Total** | **11** | |

### Closeable confidence breakdown

| Confidence | Resolved | Obsolete | Total |
|:----------:|--------:|---------:|------:|
| 🟢 5 | 1 | 0 | 1 |
| 🔵 4 | 4 | 0 | 4 |

## ✅ Likely Resolved (5)

### Confidence 🟢 5 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#223](https://github.com/radareorg/r2ai/issues/223) — What is the config format for /.config/r2ai/rc?
*3mo old · 2 comments*

The maintainer explained the rc file format (lines starting with -e or r2ai -e). The user confirmed understanding with 'ah got it now!'. This is a support question that was fully answered.

---

</details>

### Confidence 🔵 4 (4)

<details>
<summary>Click to expand 4 issues</summary>

#### [#130](https://github.com/radareorg/r2ai/issues/130) — Implement deterministic mode for r2ai.c and r2ai.py
*1y old · 0 comments*

The r2ai.c source now has r2ai.temperature config variable set to 0.01 by default, described as 'Sampling temperature for LLM output (0 = deterministic)'. The temperature is propagated through to the API calls in gemini.c and openai.c. This implements the deterministic mode concept referenced from decai settings. The Python version is in the Attic (deprecated), so implementing it for r2ai.py is moot.

---

#### [#134](https://github.com/radareorg/r2ai/issues/134) — FR: Use foreign Ollama instance?
*12mo old · 11 comments*

The C rewrite supports custom base URLs via r2ai.baseurl config option. In llm.c, the ollama provider is defined with supports_custom_baseurl=true, and r2ai_get_provider_url() checks for r2ai.baseurl to construct the proper URL. Users can now point to a remote Ollama instance by setting this config variable. The Python version issue is moot as the Python code is in the Attic.

---

#### [#151](https://github.com/radareorg/r2ai/issues/151) — decai repl
*11mo old · 0 comments*

The r2ai.c source shows 'r2ai -r' is documented as 'enter the chat repl'. A REPL mode exists in the C rewrite. The decai TypeScript plugin is separate from this repo, but the r2ai C plugin now has its own REPL functionality which was the stated goal.

---

#### [#177](https://github.com/radareorg/r2ai/issues/177) — rename "-e api" to "-e provider" ?
*10mo old · 6 comments*

The C rewrite uses both concepts: r2ai.api config variable for backward compatibility, and -p flag for provider selection. The R2AIProvider struct in the source code uses 'provider' terminology. The maintainer suggested keeping both -e api and -p/-m flags. The discussion concluded with the community agreeing to keep both approaches, making this effectively resolved by consensus even if -e api was not formally renamed.

---

</details>

## 🔧 Still Open (6)

### Confidence 🔵 4 (3)

<details>
<summary>Click to expand 3 issues</summary>

#### [#106](https://github.com/radareorg/r2ai/issues/106) — Support realtime voice support
*1y old · 1 comments*

Voice support code exists only in the Attic/py directory (voice.py), which is archived/deprecated code. The C rewrite in src/ has no voice or whisper integration. The maintainer commented in October 2025 about using VoiceInk externally. No native voice support has been implemented in the current codebase.

---

#### [#225](https://github.com/radareorg/r2ai/issues/225) — Support for Claude on Bedrock
*3mo old · 2 comments*

Bedrock support exists only in the Attic/py directory as archived code (backend/bedrock.py). The C rewrite in src/ has no AWS Bedrock integration. The maintainer explicitly stated 'Bedrock is awful. The only way i would support this would be thru popen(aws-cli)' -- indicating no plans for native support.

---

#### [#226](https://github.com/radareorg/r2ai/issues/226) — Add support for extra headers
*3mo old · 5 comments*

No custom HTTP header support was found in the C source code. The http.c file constructs requests without configurable extra headers. The discussion between the user and maintainer explored approaches (file-based header config) but no PR was submitted. The user said they would try modifying the code but no follow-up occurred.

---

</details>

### Confidence 🟡 3 (2)

<details>
<summary>Click to expand 2 issues</summary>

#### [#84](https://github.com/radareorg/r2ai/issues/84) — Prompt engineering
*1y old · 2 comments*

The issue tracks applying Chain-of-Thought and other prompt engineering techniques. The project now has prompt files in doc/role/ with various role-based prompts. The maintainer commented in October 2025 about applying techniques from a blog post. The r2ai.c source contains some prompt engineering with system prompts. However, the systematic application of techniques described in the issue (CoT methodology, step-by-step reasoning) is an ongoing effort rather than a resolved issue.

---

#### [#172](https://github.com/radareorg/r2ai/issues/172) — r2ai C resets context between each direct request
*11mo old · 1 comments*

The README.md in src/ explicitly documents this behavior: 'Note that the context is reset between each direct query.' It also lists as a TODO: 'keep context between direct commands unless explicitly reset'. This confirms the behavior is known and documented but not yet fixed. The feature request for maintaining context between -d queries remains unimplemented.

---

</details>

### Confidence 🟠 2 (1)

<details>
<summary>Click to expand 1 issues</summary>

#### [#207](https://github.com/radareorg/r2ai/issues/207) — r2ai windows version doesn't connect to AI api
*5mo old · 10 comments*

Multiple issues were reported: initial connection failures on Windows, then (null) responses even when Ollama returned valid data, then v1.2.0 not working at all. The maintainer released v1.2.2 with fixes and asked for confirmation, but the reporter never confirmed it works. The fix for shell injection in curl (commit 3db0744) may be related. Without user confirmation, the status remains uncertain.

---

</details>
