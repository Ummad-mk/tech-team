---
name: web-3d
description: Three.js / React-three-fiber essentials for shipping 3D on the web — GLB loading pipelines, texture compression, draw-call and memory budgets, render loop discipline, and the leak/jank bugs that kill web 3D. Use when adding 3D to a web page (hero scenes, product viewers, backgrounds), debugging WebGL memory leaks or frame drops, or reviewing someone's three.js/R3F integration.
---

# Web 3D — three.js / R3F field rules

The two ways web 3D dies: **memory leaks** (scene not disposed → tab heap climbs until crash) and **jank on real hardware** (dev-machine 120fps lies about the mid-range phone).

## Loading pipeline
- GLB via GLTFLoader with **DRACO + meshopt decoders** wired; textures as KTX2/basis (loader config) — uncompressed PNG textures are the #1 avoidable crash of VRAM on mobile.
- Preload before mount when it's the page's hero: percentage-based loading state; show the page instantly with a gradient/placeholder, never a blank canvas spin.
- Verify assets upstream in Blender first (see the `blender` skill): triangle counts, DRACO export, texture sizes — the web side cannot fix a 40MB GLB, only punish it.

## Performance budgets & the loop
- State budgets first: hero scene ~<100 draw calls, <2M triangles total, VRAM <200MB on mobile targets. Measure with `renderer.info` (draws, triangles) — not vibes.
- Render on demand (`invalidate()` in R3F / render only when state/animation changed) for mostly-static scenes — a requestAnimationFrame loop burning GPU on an idle page is waste; controls damping + invalidate keeps it smooth.
- Per-frame allocations are forbidden: reusable vectors/matrices per loop (`.copy()` from scratch instances), no closures/arrays in the render loop.
- Pixel ratio clamp: `Math.min(devicePixelRatio, 2)`; antialiasing (MSAA) vs postprocessing tradeoff made explicitly, resolution scale down on weak GPUs.
- Instancing for repeats (`InstancedMesh`), merging static geometry, deleting unused nodes** (`traverse` → `dispose()` geometry/materials/textures on unmount and after swaps) — leaks come from EVERY swap, not just unmount.
- Pause offscreen: IntersectionObserver or R3F frameloop="demand" gated by visibility — a 3D hero scrolled past must cost zero GPU.

## Scene craft
- Environment lighting via `RoomEnvironment`/PMREM — far cheaper and cleaner than a pile of rect lights; shadows: one, budgeted, and only if they earn narrative value.
- Tone mapping ACESFilmic + correct color space output on the renderer — or everything looks washed/dark for "mysterious reasons".
- Camera discipline: explicit FOV + near/far tuned to content (never 0.1/1000 defaults when a product viewer needs 0.5/20 — depth precision causes z-fighting).

## UX rules
- `prefers-reduced-motion` respected everywhere: autorotate, idle drift, entrance choreography suppressed to a static frame or opacity-only.
- Mobile controls: touch orbit (one finger) / pinch zoom (two); desktop orbit + zoom + pan; inertia MUST have damping — raw controls feel buggy instantly.
- Interaction affordances: cursor changes, hover highlighting on interactive meshes, and a visible loading state; canvas is not exempt from keyboard/AT considerations — critical content stays in DOM, canvas is presentation.
- Test on a REAL mid-tier phone (period), not just a desktop with small window — throttled CPU + integrated GPU is your actual audience floor.

## Debug conversational triggers (senior catalog)
- Steadily climbing memory = a dispose path you skipped (swap, route change, HMR).
- Z-fighting = scale/precision or coincident planes — epsilon offset, not luck.
- Shimmer/strobing = depth buffer precision: tune near/far or logarithmic depth.
- "Fast on my machine" = unmeasured; profile with frame timings (`advanced timer` / Spector) under simulated load.
