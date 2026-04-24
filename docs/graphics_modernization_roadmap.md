# Graphics Modernization Roadmap

## What this project is

OpenTTD is a long-running, cross-platform transport simulation game with a classic isometric pixel-art rendering model, inherited from Transport Tycoon Deluxe and extended through base graphics sets and NewGRF modding.

From a rendering architecture perspective in this repository:

- The game initializes colour gradients from base-set recolour sprites at startup (`SetupColoursAndInitialWindow`).
- The default widget system is built around palette-derived shading and bevel drawing.
- The OpenGL backend uploads the paletted screen buffer and renders it mostly with nearest-neighbour filtering.
- Base graphics and extra sprites are loaded as data packs (e.g., OpenGFX or original assets), not tightly baked into engine code.

This means modernization should focus on **renderer capabilities + UI skinning + asset pipeline compatibility**, while keeping gameplay and NewGRF compatibility intact.

---

## Current bottlenecks to a modern look

### 1) Palette- and shade-driven UI look
The GUI style is heavily dependent on colour gradient tables and bevel-style widget painting. This gives the classic 1990s raised-panel look and limits modern flat/soft UI styles.

### 2) Nearest-filtered final composition
The OpenGL backend uses nearest filtering for key textures, preserving crisp pixels but also producing visible aliasing at modern resolutions and when scaling.

### 3) Legacy-first sprite assumptions
A lot of visual quality is constrained by base-set asset resolution and palette workflows; richer modern visuals require either higher-res sprite sets or hybrid rendering paths.

### 4) Limited visual post-processing path
There is no generalized post-process pipeline for effects like gamma-correct compositing, subtle bloom/ambient shading, colour grading, or temporal smoothing.

---

## Modernization goals (without breaking OpenTTD identity)

1. Keep gameplay readability and the isometric style.
2. Preserve NewGRF and existing base-set compatibility by default.
3. Make modern visuals opt-in, then progressively default where safe.
4. Allow users to choose between classic, crisp-pixel, and modern-smooth profiles.

---

## Phased implementation plan

## Phase 0: Baseline and metrics (1-2 weeks)

- Add benchmark captures for representative scenes (dense city, rail hub, zoomed-out map, heavy GUI).
- Record FPS, frame-time variance, and VRAM usage on low/mid/high hardware.
- Capture visual goldens for regression testing.

Deliverable: reproducible visual-performance baseline report.

## Phase 1: Renderer quality toggles (2-4 weeks)

- Add a render quality profile setting:
  - `Classic` (today’s behavior)
  - `Crisp HD` (pixel-preserving but improved scaling paths)
  - `Modern Smooth` (linear or hybrid filters where appropriate)
- Split filtering strategy by layer:
  - World view
  - GUI/text
  - Minimap/overlays
- Keep text/icons sharp even when world smoothing is enabled.

Deliverable: immediate visual upgrade for modern displays with reversible settings.

## Phase 2: UI skin modernization (4-8 weeks)

- Introduce theme tokens (surface, border, accent, text) instead of hard-coded shade coupling.
- Implement a `Modern` skin while retaining `Classic` skin.
- Update core widgets:
  - Buttons, panels, scrollbars, toolbars, dropdowns
  - Spacing and density tuning for high DPI
- Keep low-contrast and accessibility-safe palette variants.

Deliverable: significantly updated UI appearance without touching gameplay logic.

## Phase 3: Asset modernization track (parallel, ongoing)

- Define an official “HD-compatible base-set profile” for sprite packs.
- Provide tooling/docs for dual-target assets:
  - legacy 8bpp/palette path
  - enhanced 32bpp/alpha path
- Support fallback rules so any missing enhanced asset gracefully drops to legacy sprite.

Deliverable: improved terrain/buildings/vehicles over time without forcing an all-at-once conversion.

## Phase 4: Lighting and compositing enhancements (optional advanced)

- Add optional shader-based pass for:
  - subtle ambient occlusion approximation
  - soft selection glow/highlight polish
  - day/night tinting (if enabled by scenario/settings)
- Ensure deterministic rendering behavior where network determinism matters (visual-only pipeline must not affect simulation state).

Deliverable: modern depth/contrast improvements while maintaining readability.

---

## Engineering guardrails

- **Compatibility first:** Modern features must degrade gracefully to classic paths.
- **Mod safety:** NewGRF sprite resolution/palette variants must remain predictable.
- **Performance budgets:** Any modern mode needs bounded GPU/CPU cost and fallback on low-end systems.
- **Test strategy:** Use screenshot regression tests + perf checks for each quality profile.

---

## Suggested concrete work items in this repo

1. Add `render_quality_profile` setting and plumbing in settings UI.
2. Implement per-layer filter selection in OpenGL video path.
3. Add theme abstraction layer for widget colours and bevel model.
4. Create first-pass `Modern` theme token table and update high-impact widgets.
5. Add developer docs for enhanced asset packs and fallback behavior.

---

## Why this approach is “significantly better” but realistic

- Users get immediate visual gains from filtering, UI skin updates, and DPI polish.
- Classic look remains available for purists and compatibility.
- Asset improvements can ship incrementally, avoiding a risky full-asset rewrite.
- The architecture supports future visual work without destabilizing simulation code.

