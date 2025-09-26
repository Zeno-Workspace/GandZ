## Developer Notes — ZENØ v2 (Refactored Single‑File)

These notes give contributors a concise map of the code, how to run/debug it, and where to extend features. The app is a single HTML file (`zeno-polyhedron-refactored.html`) using Three.js ES modules via an import map.

### Local dev
- Serve over HTTP (ES modules/import map won’t work from file://).
- Minimal server options:
	- Python: `python -m http.server 8000`
	- Node: `npx http-server -p 8000`
- Open: http://localhost:8000/zeno-polyhedron-refactored.html
- On WSL, place the repo under `~/work/GandZ` and run the server from there for a consistent path.

### Keyboard controls (dev)
- Space: Pause/Resume
- R: Reset camera and state
- P: Toggle performance monitor (shows FPS, draw calls, triangles)
- 1/2/3: System state (CALM / ALERT / DEFENSIVE)
- H: Toggle help text
- F: Fullscreen
- B: Toggle bloom (post‑processing)
- S: Toggle custom shaders (outer “Gates” layer when enabled)
- I: Cycle bloom quality presets (low/medium/high)
- Shift+I: Increase shader intensity (when custom shaders are enabled)

### High‑level architecture (all in one file)
- CONFIG: Centralized configuration (camera/controls, layers, animation, materials, lighting, effects, UI).
- Utils: Mobile/Apple detection, clamp/lerp/map, WebGL version utility.
- PerformanceMonitor: FPS/calls/triangles panel; rolling history and auto‑adjust hooks.
- SystemStateManager: CALM/ALERT/DEFENSIVE state, integrity/sync metrics, listeners, DOM binding.
- ResourceManager: Tracks geometries/materials/textures and disposes deterministically.
- LayerFactory: Builds the solid core and multiple wireframe shells; edges, points, subtle glow pass.
	- ensureWireframeShaderAttributes adds custom attributes (segmentDirection/Length/Offset/Seed) used by shaders.
- AnimationController: Rotation, breathing, opacity pulsing, vertex point pulsing; per‑layer phase behavior.
- PostProcessingManager: EffectComposer pipeline, selective bloom isolation + additive composite, quality presets, auto‑adjust.
- ShaderManager: “Energy flow” additive shader for wires; uniforms for colors/time/state/intensity.
- SceneManager: Renderer (WebGL1/2), scene, camera, fog, lights, OrbitControls; resize, render, dispose.
- ZenoApplication: Bootstraps everything, creates layers, wires UI/handlers, runs the animation loop, cleans up.

### Render/update flow
1) requestAnimationFrame → delta via THREE.Clock
2) If not paused: SystemState.update → AnimationController.update
3) Optional ShaderManager.update (when custom shaders enabled)
4) Controls.update → render via PostProcessingManager or direct renderer
5) Perf monitor update (if visible) and optional auto quality adjust

### Layers and how to add/modify them
- Layers are defined in `CONFIG.layers`. Each item:
	- type: 'solid' | 'wireframe'
	- name: string (used for legend/debug)
	- scale: number (relative size)
	- color: hex (e.g., 0xAA00FF)
	- complexity: icosahedron detail (2–3 works well)
- Core (solid) uses MeshPhysicalMaterial; shells (wireframe) are composed of:
	- Wireframe (LineSegments, additive, transparent)
	- Edges (for silhouette clarity)
	- Points (vertex markers; animated size)
	- Thin “glow” pass (extra LineSegments at ~1.003 scale)
- Each layer group stores references in `group.userData` (materials/meshes/baseScale) that AnimationController updates.

### Selective bloom (Phase 2 ready)
- Config: `CONFIG.visualEffects.bloom` (enabled, intensity, radius, threshold, layers)
- Flow:
	- Mark bloom candidates on THREE layer 1 via `prepareLayersForBloom()`.
	- Render bloom composer at scaled resolution.
	- Additively composite bloom over the base composer.
- Quality presets (toggle with “I”): low → medium → high; or call `postProcessing.setQualityPreset()`.
- Auto quality adjust can downshift on low FPS (`CONFIG.visualEffects.autoAdjust`).

### Custom shaders (Phase 3 scaffold implemented)
- Energy Flow shader lives in `ShaderManager` (additive, transparent, uses custom wireframe attributes + rim term).
- Enable via “S” or set `CONFIG.visualEffects.customShaders.enabled = true`.
- Intensity: Shift+I cycles 0.5 → 1.0 → 1.5 → 2.0 → 0.5.
- Colors: `ShaderManager.setColors(colorA, colorB, colorC)`; defaults target the “Gates” outer layer.

### Performance and quality tips
- Antialiasing is disabled under a width threshold (`CONFIG.performance.antialiasThreshold`).
- Pixel ratio is capped (`maxPixelRatio` desktop; `mobilePixelRatio` mobile) to contain cost.
- Wireframe line width is GPU/driver‑limited in WebGL; adopt Line2 (examples) if thicker lines are needed.
- Perf panel (“P”) resets `renderer.info` when shown to collect accurate next‑frame stats.

### Debugging & troubleshooting
- Blank page: ensure HTTP server and import map URLs load; check console for module errors.
- WebGL1 vs WebGL2: both supported; WebGL2 preferred for post‑processing stability/perf.
- Context loss: handlers pause + set ALERT; on restore, resumes. See `setupContextLossHandlers()`.
- Mobile: supported but desktop‑first; quality may auto‑downshift.

### Extending the system
- New effect pass: extend `PostProcessingManager.setupEffects()` and handle `resize`/`dispose`.
- True selective bloom (masking): swap non‑bloom materials to black, render layer 1, composite; current approach hides non‑bloom objects for isolation.
- Additional shader packs: add materials in `ShaderManager`, wire toggles and uniforms.
- Modularization: future step is to extract classes to ES modules; keep public APIs stable during extraction.

### Handy globals
- The app instance is exposed as `window.ZENO` for console debugging (e.g., `ZENO.togglePerformanceMonitor()`, `ZENO.postProcessing.setQualityPreset('high')`).

### House rules
- Keep `CONFIG` as the single source of truth for tweakable knobs.
- Track new geometries/materials/textures with `ResourceManager` so they’re disposed.
- Prefer additive, transparent materials for luminous wire effects; keep `depthWrite: false` to reduce artifacts.
