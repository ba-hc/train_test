# V3.1 clean: OMP 18.3.0, one global skill, empty global MCP

This is the completed run from a fresh directory with the exact [canonical prompt](../../benchmark_prompt.txt), GPT-6 Luna `max`, OpenAI `priority`, and OMP **18.3.0**. The active profile had `mcp.enableProjectConfig: false`, no global MCP servers, `skills.enableAgentsUser: false`, and only Impeccable in OMP's global skill directory. The agent read Impeccable and delegated one read-only visual review to `designer`. It did not read a skill from `~/.agents/skills`.

| Metric | V3 raw | V3.1 clean | Change |
| --- | ---: | ---: | ---: |
| Session duration | 27.0 min | **43.9 min** | +62.5% |
| Model requests | 83 | **113** | +36.1% |
| Tool calls | 88 | **121** | +37.5% |
| Browser `eval` | 38 | **33** | -13.2% |
| Fresh input | 507,238 | **874,477** | +72.4% |
| Cache read | 7,437,568 | **10,003,328** | +34.5% |
| Total tokens | 8,030,964 | **11,044,328** | +37.5% |
| Recorded cost | USD 0.3364 | **USD 0.5415** | +61.0% |

Input cache hit was 91.96%, versus 93.62% for V3. One remote compaction occurred at 224,482 tokens, reducing context to 17,400. The longest interval between assistant responses was 393.9 seconds. The run exited normally with code 0. These are values from the session JSONL, summarized in [session.json](./session.json); the full JSONL is not published because it contains local paths and configuration material.

## Artifact and checks

The [HTML](./OMP_CUSTOM_V3_1_CLEAN.html) is exactly the file OMP left at session end, with no later refinement. [Desktop](./OMP_CUSTOM_V3_1_CLEAN_desktop.png) and [mobile](./OMP_CUSTOM_V3_1_CLEAN_mobile.png) are final captures. Chromium rendered a canvas at 1600×900 and 390×844 without horizontal overflow or console errors ([capture data](./visual-check-clean.json)). The single-file HTML loads Three.js from jsDelivr, so it needs network access when opened.

An external [instrumented check](./functional-check-clean.json) confirmed station doors, W acceleration, S braking, arrival at Mango Tide with a +75 shell bonus, Cloudworks access, upgrade completion, and return direction to Saltlight. The check advanced arrival and workshop progress in a temporary copy to exercise transitions quickly; it did not time the complete ride or edit the published HTML. OMP's own final response reported a full route and mobile control checks.

## Evaluation

The game covers the main [prompt requirements](../../benchmark_acceptance.md): two named islands, curving rail, tram and mobile controls, comfort/streak, passenger exchange, workshop, and return trip. The final first viewport is more legible than its first draft after camera edits, yet V3 refined still presents the tram, station, town, lighthouse, and route more clearly at a glance. The V3.1 workshop sits at Mango Tide and one Begin action completes both named upgrades. Those are quality rubric preferences, not unequivocal textual prompt failures. On mobile, Power/Brake are 57 px tall, but Open doors is 41 px, under the 44 px quality target.

The smaller browser-call count did not reduce total work. A long initial generation, later visual/camera repair cycles, read-only designer review, and remote compaction contributed to the larger run. This is one sample; it does not prove any individual setting caused the regression. The package as measured is **worse than V3 on efficiency**, and V3 remains the recommended public HTML.
