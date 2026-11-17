---
name: blender
description: Operating manual for Blender 3D work — headless CLI + Python (bpy) scripting, scene/material/lighting/render craft, animation via code, and GLB/glTF export pipelines for web and game targets. Use whenever building or modifying 3D assets with Blender, writing bpy scripts, exporting GLB/FBX, rendering images or animations, fixing Blender context errors, or optimizing meshes/textures for a target platform.
---

# Blender — script-first 3D craft

The rule that shapes everything: **assets are built by Python scripts, not by hand** — a `.blend` file nobody can reproduce is a liability; a script that rebuilds it is an asset. Every deliverable ships with its build script checked in.

## Headless pipeline
- Run: `blender --background <file.blend> --python build.py -- <args>`; everything after `--` reaches the script via `sys.argv`.
- Pin the Blender version (`blender --version`) — bpy API breaks between majors (color management, node names, operator args). Guard version-sensitive calls.
- Scripts are idempotent: on rerun they either rebuild fresh (`bpy.ops.wm.read_factory_settings(use_empty=True)` style purge, or explicit delete-by-name) or skip-if-exists — never blindly append duplicate objects, materials, or collections.

## bpy essentials (and their traps)
- Data API over operators where possible: `bpy.data.objects.new()`, assignment to `collection.objects.link()`, transforms via `.location/.rotation_euler/.scale` — `bpy.ops` requires a working context (mode, selection, area) that headless runs don't have. The classic crash is `bpy.ops.object.*` with wrong context: either set context explicitly (`context.override`) or use the data API.
- Modes matter: mesh edit operations run in edit mode via `bpy.ops` with object mode toggling — or use `bmesh` for direct mesh surgery (more reliable headless).
- Apply transforms (rotation/scale) before export: unapplied scale breaks physics, skinning, instancing downstream.
- Naming discipline: collections by role (`sm`, `props`, `bg`), objects by purpose — the scene tree is the next artist's map.
- Purge orphans before saving (`bpy.ops.outliner.orphans_purge` semantics via data API) — orphan meshes/materials/images bloat files and exports.

## Scene craft
- Units: metric, scene scale 1.0 — **1 Blender unit = 1 meter**. Web/game targets assume it.
- Materials: Principled BSDF node; keep shader graphs minimal; absolute vs relative image paths — use relative and pack external textures when the file travels.
- Lighting: three-point or an HDRI environment; **color management is not optional** — set `view_transform` to AgX (Blender 4.x) or Filmic for.render-ready look; "blown-out neon" usually means the default Standard transform leaked through.
- Render: Cycles for physical quality (samples + denoise tradeoff), EEVEE for speed/realtime look; always set explicit resolution, camera, and output path — never render "whatever the last scene had".

## Animation by code
- Keyframes: `obj.keyframe_insert(data_path="location", frame=N)`; shape motion with fcurve interpolation/mediums in the action curves.
- Frame range and FPS set explicitly per deliverable; looping animations must be frame-exact (first == last frame state) — test the loop by scrubbing end-to-start.
- For character/prop motion for games: export ACTIONS individually (not the NLA soup) so the runtime maps clips by name.

## Export pipelines
- Web/game default: glTF 2.0 `.glb` — self-contained, Y-up (Blender converts Z-up automatically on export), supports skins/animations/DRACO.
- Set `export_draco_mesh_compression` + KTX2/basis texture compression downstream (target engine does) when size matters; report before/after sizes.
- Triangulate for engines that demand it; check normals/shading (smooth-by-angle) before shipping; skeleton scale 1.0, no transform animations on root if the engine forbids.
- Validation before delivery: non-manifold edges = export bugs; verify with a re-import (import the GLB into an empty scene and diff stats) — the export round-trip is the test.

## Optimization by budget
- State polygon/texture budgets per target first (e.g., web hero: <100k tris, <10MB GLB; mobile game prop: <20k tris) — then decimate/instancing/atlas to hit them. LODs via decimate modifiers per level.
- Instancing over duplication (`collection instances` or shared mesh data) — 200 copied chairs as unique meshes is a beginner crime scene in file size and RAM.

## Verification & honesty
- Render frame previews at the deliverable's real sizes and READ the images back (this session can view PNGs).
- Validate the GLB with a re-import; report tri counts, file size, material count.
- Sculpt-level detail, complex UV unwrap fixes, or precise retopology that exceeds script reliability: flag "needs manual" with exact specs — never hand back "probably fine" geometry.
