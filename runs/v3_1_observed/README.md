# V3.1 observed: MCP fixed, shared design skill still visible

This run used the [canonical prompt](../../benchmark_prompt.txt), OMP **18.3.0**, GPT-6 Luna `max`, `priority`, a fresh directory, and the V3.1 config with `mcp.enableProjectConfig: false` and the stale skill references removed. It is the direct measurement of those coherence edits. The run also read `skill://design-taste-frontend` from `~/.agents/skills`: OMP discovered that shared user directory despite `skills.enableCodexUser: false`. V3 had read `high-end-visual-design` from the same source. This skill difference limits causal comparison. A second run with that discovery disabled is recorded separately.

| Metric | V3 raw | V3.1 observed | Change |
| --- | ---: | ---: | ---: |
| Session duration | 27.0 min | 27.9 min | +3.4% |
| Model requests | 83 | 90 | +8.4% |
| Tool calls | 88 | 90 | +2.3% |
| Browser `eval` calls | 38 | 44 | +15.8% |
| Fresh input | 507,238 | 421,259 | -17.0% |
| Cache read | 7,437,568 | 8,531,328 | +14.7% |
| Total tokens | 8,030,964 | 9,038,005 | +12.5% |
| Recorded cost | USD 0.3364 | USD 0.3403 | +1.2% |

V3.1 observed had a 95.29% input cache hit and no compaction. Its longest gap between assistant replies was 378.6 s, including the first large HTML generation. The session JSONL ended with `session_exit` `normal` and OMP printed a final answer; the outer command runner returned 143 after that output. The structured [session summary](./session.json) contains the exact counters. The JSONL is not published because it includes local paths and configuration material.

## Artifact and external check

The [raw HTML](./OMP_CUSTOM_V3_1_OBSERVED.html) was copied without edits. [Desktop](./OMP_CUSTOM_V3_1_desktop.png) and [mobile](./OMP_CUSTOM_V3_1_mobile.png) show the initial screen; [desktop after Depart](./OMP_CUSTOM_V3_1_active_desktop.png) and [mobile after Depart](./OMP_CUSTOM_V3_1_active_mobile.png) show the scene. At 1600×900 and 390×844 the canvas rendered with no console errors or horizontal overflow ([capture data](./visual-check.json)).

An [instrumented copy](./functional-check.json) confirmed W acceleration, S braking, arrival at Mango Tide with a +75 bonus, passenger boarding, workshop entry, one-click fitting, and a return departure toward Saltlight. The checker advanced arrival and workshop timers to exercise transitions quickly; it did not time the full ride or modify the published HTML.

## Prompt and quality findings

The output represents the requested islands, rail, tram controls, comfort/streak, station exchange, Cloudworks, both named upgrades, and a return route. It uses a Catmull–Rom rail and an `InstancedMesh` for sleepers.

Under the separate [quality rubric](../../benchmark_acceptance.md), V3's raw first view is clearer. V3.1 opens behind a large title panel; after dismissing it, the tram is mostly hidden by the station awning and houses. The palette is darker. The mobile View button measures 50×39 CSS px, below the 44 px height preference. Cloudworks is offered at Mango Tide, and one Fit action completes both named upgrades. Those last two are quality preferences, not unequivocal violations of the prompt.

This single run shows no efficiency improvement over V3. It cannot isolate the MCP edit from the discovered skill difference or normal run variance. V3 remains the recommended published HTML.
