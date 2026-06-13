# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file vanilla web application (`index.html`) that renders an interactive 3D particle system with webcam-based hand gesture control. No build tools, no framework, no package manager — the entire app (HTML, CSS, JS, GLSL shaders) lives in one file. UI is in Chinese (zh-CN).

## Running

Open `index.html` directly in a browser, or serve locally (required for webcam hand-tracking due to browser camera permission policies):

```
npx serve .
```

or

```
python -m http.server
```

No build step. No installation required.

## Architecture

Everything is in `index.html` (~755 lines). The `<script>` block is organized into labeled sections:

- **CONFIG** — Global constants: `PARTICLE_COUNT_DEFAULT`, `LERP_SPEED`, `EXPLOSION_STRENGTH`, `TURBULENCE_STRENGTH`, `SWIPE_FORCE`, `SWIPE_DECAY`, `CONTRACT_STRENGTH`, `SPIRAL_STRENGTH`
- **THREE.JS SETUP** — Scene, camera (z=15), renderer, particle system initialization
- **createParticles** — Builds `BufferGeometry` with custom `ShaderMaterial` (inline GLSL); per-particle `size` and `aHue` attributes; uniforms: `uColor`, `uTime`, `uPixelRatio`, `uRainbow`
- **TEMPLATES** — Five shape generators: heart (parametric surface), flower (rose curve, k=5), saturn (sphere+ring), buddha (head/body/halo/legs composite), firework (5 radial bursts)
- **ANIMATION** — `requestAnimationFrame` loop with three gesture effect layers:
  - Open hand: explosion + spiral twist + shockwave ripple
  - Closed fist: contraction + vortex swirl + turbulence
  - Directional swipe: palm velocity tracking → push force (left/right/up/down)
- **MEDIAPIPE HANDS** — Webcam hand tracking; computes openness from fingertip-to-palm distance; tracks palm position history for swipe velocity detection
- **UI** — Sidebar: template buttons, color picker, 8 color presets, rainbow mode toggle, particle count slider. Gesture HUD: openness bar + direction indicator (compass dot)
- **INIT** — Entry point: `initThree()` → `initUI()` → `initHands()` → `animate()`

## External Dependencies (CDN)

- Three.js v0.160.0 (unpkg)
- MediaPipe Hands v0.4 (unpkg)
- MediaPipe Camera Utils (unpkg)

No local dependencies or lockfiles.

## Key Interactions

- **Open hand**: Particles explode outward radially, spiral-twist around Y axis, and emit shockwave ripples. Strength scales with openness percentage.
- **Closed fist**: Particles contract toward center with pulsing, vortex-swirl around Y, and chaotic turbulence.
- **Directional swipe**: Palm wrist position tracked over recent frames; velocity computed and smoothed into `gestureDir`; applies directional push force to all particles. Decays over time when hand stops.
- **Template switching**: Sidebar buttons trigger `generateTemplate()` which regenerates target positions.
- **Color**: Picker, 8 preset swatches, and rainbow mode (per-particle HSV via shader `uRainbow` uniform).
- **Particle count slider**: Recreates the entire particle system (2000-20000).

## Modifying the Project

- **Add a new template**: Add a `case` in the TEMPLATES `switch` block computing `x, y, z` per particle, then add a `<button class="tpl-btn" data-tpl="name">` in HTML.
- **Tune physics**: Adjust CONFIG constants. `LERP_SPEED` (0-1) controls animation smoothing; `EXPLOSION_STRENGTH`/`CONTRACT_STRENGTH` scale gesture effects; `SWIPE_FORCE`/`SWIPE_DECAY` control directional responsiveness.
- **Shaders**: Inline in `createParticles()`. Vertex shader handles point size, alpha, and passes `aHue` to fragment. Fragment shader does HSV-to-RGB conversion for rainbow mode, sharp-edged circular particles with bright center core, additive blending.
