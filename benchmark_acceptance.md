# Benchmark rubric

Use the exact text in [`benchmark_prompt.txt`](./benchmark_prompt.txt) for every run. Report the raw agent output and any later refinement separately. Record the model, effort, service tier, OMP version, wall time, requests, tool calls, tokens, and cost when available.

## A. Prompt faithfulness

Check requirements stated in the prompt. A missing item here is a prompt failure.

- Playable browser game using Three.js in one HTML file or minimal files.
- A retro aerial tram follows a continuous 3D rail with straights, slopes, curves, and long bridges between at least Saltlight Terminus and Mango Tide.
- W accelerates, S brakes, and left/right adjusts the view or tram; mobile tap controls are attempted.
- Speed, passengers, road conditions, comfort, streak, and destination are represented. Driving harshly lowers comfort; smooth arrival earns a bonus. The stated broken-streak message appears when relevant.
- Station arrival opens doors and exchanges passengers, with townsfolk and subtitles.
- The world, tram, dusk sky, sea, and HUD include the visual features described in the prompt.
- Cloudworks/Oliver workshop provides the two named modifications, progress and workshop messaging, visible tram changes, and an exit sequence.
- Curved rail following, camera follow, acceleration inertia, braking, and corner roll are implemented; code is readable and commented.

## B. Quality rubric

These checks help compare craft and usability. They are evaluator preferences unless the prompt explicitly says otherwise. Report them separately from prompt failures.

- First viewport clearly shows a recognizable tram, town, and travel route; HUD does not dominate the scene.
- Workshop placement and access make narrative sense. Saltlight-only access is a preference, because the prompt does not identify Oliver's home island.
- The two upgrades have separate deliberate interactions. The prompt names both upgrades but does not prescribe click count.
- Mobile touch targets are comfortably sized, ideally at least 44 CSS pixels; visible keyboard focus and reduced-motion support are present.
- Console is free of errors; rendering and frame rate are reasonable. Consider instancing for numerous repeated meshes.
- Check both desktop and mobile screenshots and the main gameplay and workshop paths.

Do not silently convert a quality preference into a prompt requirement after seeing a run.
