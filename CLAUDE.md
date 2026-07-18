# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page 3D visualization of a ~3.33 × 2.89 m home office ("חדר עבודה"), presenting four furniture-layout options (A / B / C / D) with an interactive orbit camera, preset views, on-model dimension labels, a measuring tool, and PNG snapshot export. The UI is in Hebrew and the page is right-to-left (`<html dir="rtl">`).

Everything lives in **`index.html`** — markup, CSS, and all JavaScript are in that one file. There is no build system, no package manager, no tests, and no dependencies to install. Three.js **r128** is loaded from a CDN.

## Running & deploying

- **Preview locally:** open `index.html` directly in a browser, or serve the folder (`python3 -m http.server`) and visit it. No build step.
- **Deploy:** the site is served via GitHub Pages from `main` at the domain in `CNAME` (`building-vis.dvash3.com`). Pushing to `main` publishes. There is no CI.

## Architecture (the parts that span the file)

The JS is one IIFE (`(function(){ ... })()` starting ~line 88). Key structural facts:

- **Coordinate system.** Room constants `RW`/`RD`/`RH` (width/depth/height). Floor-plan Y and 3D Z are inverted via `const Z = y => RD - y` — plan-y 0 (bottom) maps to the far wall, plan-y RD to the near/door wall. Most furniture placement math goes through `Z(...)`, so read it before moving anything.
- **Scene rebuild.** `build(opt)` is the central dispatcher: it clears the `world` group (`clearWorld()` disposes geometries), then composes the room from `buildShell` + `buildDoors` + a cabinet + a per-option layout. Options B and C share `buildB(world, seatAxis)` differing only by seating axis (`'x'` vs `'z'`); A uses `buildA`; D uses `buildD` (cabinet moves to the far wall). Switching options in the UI re-runs `build`.
- **Geometry helpers.** `box(w,h,d,mat,x,y,z,parent)` and `plane(w,h,mat)` build everything; furniture (chairs, desks, etc.) is assembled procedurally from these into `THREE.Group`s. Materials live in the shared `M` map; wood/wall textures are generated on a `<canvas>` at runtime (no image assets for materials).
- **Camera.** A hand-rolled spherical orbit camera (`radius/theta/phi` + goal values that ease toward targets each frame in `tick()`), **not** `OrbitControls`. `viewsFor(opt)` returns named preset views with literal `eye`/`tgt` coordinates that were dialed in manually through the HUD — treat those numbers as tuned data, not something to "clean up." A separate top-down mode (`gTop`) uses the orthographic camera.
- **Dimensions.** `buildDims(opt, group)` builds labeled dimension groups (`room`/`cabinet`/`desk`/`seating`); `updateDims()` makes them context-aware — overview/top views show all, a detail view shows only its part.
- **Reference images.** `window.REFDATA` (defined in a separate `<script id="refdata">`, ~line 87) is a large map of base64-encoded JPEG reference photos keyed by `"<opt>|<viewName>"` (e.g. `"A|מבט־על"`). `showRef(opt, viewName)` overlays the matching reference for side-by-side comparison. This blob is what makes the file ~1.2 MB.

## Working notes

- `index.html` is ~1.2 MB almost entirely because of the embedded base64 `REFDATA`. Avoid dumping the whole file; the actual code is the short lines (`awk 'length($0) < 300' index.html` skips the base64 blobs). Editing near or inside `REFDATA` risks corrupting the encoded images.
- View names, legend text, and button labels are Hebrew string literals used as object keys (e.g. `"מבט כללי"`, `"שולחן עבודה"`). Keep them byte-identical when referencing them in code.
