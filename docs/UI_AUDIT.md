# F13LD.grain — UI audit (stage 1)

Branch `ui-refresh`, 2026-11-25. Read-only: no code changed. Screenshots in `docs/ui-audit/`.
No `CLAUDE.md` in this repo; the handoff brief applies as written.

Screenshots (headless Chrome, swiftshader, DPR 1):

| File | What |
|---|---|
| `desktop-1440.png` | 1440×900, spinodoid default |
| `desktop-view-menu.png` | shared view menu open |
| `desktop-help.png` | help drawer |
| `desktop-hyperuniform-full.png`, `desktop-hyperuniform-panel-bottom.png` | hyperuniform panel (state after probe edits: center 0.45, κ 6.5) |
| `desktop-rd.png` | reaction-diffusion panel, simulation running |
| `desktop-homog.png` | after Run homogenization: glyph box fills half the panel |
| `phone-390.png`, `phone-390-panel-scrolled.png`, `phone-390-help.png` | 390×844 |
| `tablet-700.png` | 700×900 |

Page errors: 0 at 390 / 700 / 1024 / 1440 (desktop run with every probe: 0).
Horizontal scroll: none at 390 / 700 / 1024 / 1440 (page and `#panel` both 0 px).
Harness note: the first WebGL page in a fresh swiftshader browser loses its context ~150 ms after load and then throws on every input (see F12). Open one throwaway page first. Not a user-facing bug by itself, but F12 is.

---

## 1. Current state

- **Version:** header `v0.11.0` (line 116). Export schema `meta.version: '0.9.4'` (line 1330). Page `<title>` is still "Spinodoid Field Explorer" (line 6).
- **Size:** `index.html`, 1885 lines, 157 KB. Single file, no build.
- **Blocks:**

| Lines | Block | Notes |
|---|---|---|
| 7–87 | `<style>` | tool CSS, hex colours, header CSS |
| 89–312 | markup | header, `#cw` viewer, `#panel` (one long scrolling column) |
| 313–1413 | `<script>` (main) | F13LD-SHADE (314–361), F13LD-VIEW (362–431), worker source (433–474), field builders, `Raymarcher`, stiffness glyph, homogenization, RD presets, handlers, export |
| 1423–1437 | `style#f13h-theme` | help theme, accent `#22d3ee` |
| 1440–1530 | `style#f13h-style` | help engine CSS (shared) |
| 1532–1839 | `script#f13h-script` | help engine v1.1 + per-tool CONFIG (1536–1706) |
| 1842–1884 | `<script>` (queue) | `F13LD_snapshot`, loads `f13ld-queue.js` from mshomper.github.io, `alert()` fallback |

- **Drop-ins:** help (`f13h-*`, v1.1), queue client (remote script, `F13LDQueue.mountButton`, `tool:'f13ld.grain'`), view menu (`F13LD-VIEW v1`, `f13ViewInit`), shading (`F13LD-SHADE v1`). No liquid-ui yet.
- **Renderer:** WebGL2 fragment ray-march over a baked `R8` 3D texture (`uField`). The field (spinodoid / GRF / hyperuniform / RD) is baked in a Blob worker (`GRAIN_WORKER_SRC`) at N³ (64/96/128/256 by quality, 48 for RD), main-thread fallback `_bakeFieldSync`. Iso level, half-width, topology, half-invert, clip, tile are uniforms. One `requestAnimationFrame` loop that never stops (line 948); auto-rotate on until first drag. Second WebGL2 context for the stiffness glyph (`#ellcv`) with its own endless loop (line 958).
- **Export:** `buildExportJSON()` (1315–1339), single source for Export / Mesh / Lab / Queue.

```
{ meta:{ version:'0.9.4', tool:'grain', timestamp },
  field:{ type, rd_system, rd_F, rd_k, rd_Du, rd_tile, rd_steps, n_waves, kappa, frequency, rng_seed,
          dir_mode, principal_direction|null, ortho_weights|null,
          hu_n, hu_aspect, hu_width, hu_cross, hu_sharp, hu_blend, hu_ell, grf_sigma,
          rd_Dv  (omitted for Gray-Scott) },
  geometry:{ center, half_width, smoothing, topology, half_invert },
  homogenization:{ grid, E_solid_GPa, poisson, volume_fraction, Ex_GPa, Ey_GPa, Ez_GPa, Gxy_GPa, nu_eff, anisotropy, method:'MIL-HS' }  (only after a run) }
```

- **`meta.tool` = `'grain'`.** Lab and Mesh route on it. Never rename it. The queue button's `tool:'f13ld.grain'` is a separate tag; leave it too.

## 2. Findings

Severity: H = blocks the standard, M = visible problem, L = tidy-up.

| # | Issue | Where | Sev | Rule |
|---|---|---|---|---|
| F1 | One long scrolling panel, 31 `.sc-group` rows, no sections. Viewer gets 352 of 900 px at 1440 (`#panel` max-height 52vh, `#container` max-width 1100). | 10, 21, 154–311 | H | Lesson 6; one focus per screen |
| F2 | All controls native: 4 selects, 26 ranges, 2 number fields, 1 checkbox, 4 pill button groups. | 158–310 | H | Lesson 5; controls → widgets |
| F3 | Range thumbs carry a CSS glow (`box-shadow:0 0 6px`). | 28–29 | M | Lesson 8 |
| F4 | Font sizes in use: 8, 9, 10, 11, 12, 13, 14, 15, 17, 18 px. Hints are 10 px inline styles; view menu 10 px mono. | 13–52, 167–308, 378–385 | H | Type scale 11/13/15/20/28 |
| F5 | 48 visible targets under 44 px (mode pills `padding:3px 10px`, clip pills, quality pills, selects, number fields, view button, help `i`). | 16, 32, 34–35, 86, 379 | H | 44 px targets |
| F6 | Hex colours, no OKLCH tokens. Four competing accent hues: cyan `#22d3ee` (sliders, help), violet `#7c6ff7` (active pills, Run), green `#18f370` (section labels), amber `#f59e0b` (warnings, "working"). | 9–86, 1423–1437 | H | OKLCH tokens; one tool accent |
| F7 | Low-contrast text: `#555` hints/labels on `#111118` ≈ 2.5:1; tool name `#3D3A30` and version `#2a3a2a` on `#06080f` ≈ 1.6:1 and 1.3:1. | 13, 20, 50, 111, 167… | M | Contrast ≥ 4.5 / 4.5 / 3 |
| F8 | Symbol glyphs as icons: `⬡ Mesh`, `▦ Lab`, `⊕ Queue`, `▾`, `◐ view`, `◐ flip side`, `▶ Simulate`, `🎲 Roll`, `⚠`, `×`, `?`, `Tutorial ↗`. | 114–115, 176, 195, 212–214, 267, 308, 390, 1371, 1857 | H | Drawn SVG only |
| F9 | Manual quality switch Low / Med / High / Ultra over the viewer. It sets both ray-march steps and the bake resolution N. | 147–152, 857–858, 917, 1069 | H | Lesson 2; fewer choices |
| F10 | Render loop never sleeps: rAF every frame forever; auto-rotate on by default draws continuously (43 draws / 2.5 s idle in swiftshader). Canvas is resized inside `render()`; DPR cap 1.5; no interactive/rest tiers. | 947–948 | H | Lesson 2; render-quality §2, §4 |
| F11 | Stiffness glyph loop runs at display rate forever once a homogenization has run, also off-screen (150 draws / 2.5 s idle). | 958, 960 | M | render-quality §4 |
| F12 | Failed or lost program makes every input recompile: `needRebuild` is true whenever `!this.prog&&!this._compiling`, and `_mkShader` throws on a null info log (`log.match`). Seen as 35 page errors in one session after a context loss. | 932, 945 | M | Robustness; lesson 1 |
| F13 | Field sliders (κ, N waves, frequency, seed, θ, φ, weights, all HU, σ) re-bake the 3D texture: 300 ms `regenerate` debounce + 150 ms bake debounce + bake time before anything changes. Slow drag (400 ms per tick) posted one bake per tick (8/8). | 946, 1268 | H | Respond within 400 ms |
| F14 | `smoothing` slider does nothing visible: not in the rebake key, not in the shader, not in `voxelizeField`; only used by the unused `SpinodoidGridGenerator`. Still exported, and the help card says it smooths. | 274, 782–789, 946, 963, 1027, 1332, 1676 | M | Fewer choices; tidy output |
| F15 | Dead code: `SpinodoidGridGenerator` (never called), `generateSectionGrid` defined twice (never called), `invertSign` uniform not in the shader. | 771–800, 842–854, 902–916, 943 | L | Housekeeping |
| F16 | Help `i` for the clip card never renders: its anchor `row:'#clipPillX'` has no `.sc-group` or label. 31 anchored cards, 30 `i` on the page. | 1544 | M | Lesson 7 |
| F17 | View button (`top:64px`) overlaps the field/shader indicator pills on every width. | 14, 1409 | M | Layout |
| F18 | Stiffness glyph box is square at half the panel width: ~520×400 px at 1440, pushes results off-screen. | 301–306 | M | One focus; chunking |
| F19 | RD order is wrong for the workflow: `sim steps` sits below Simulate / Roll. | 213–216 | L | Workflow order |
| F20 | Hint lines placed with negative margins (`margin-top:-16px`) as inline styles; empty `hr.divider` pairs when GRF/HU blocks are hidden. | 155, 167–185, 169 | L | Layout |
| F21 | `alert()` queue fallback and a custom `#labNote` toast instead of the shared header message. | 76–79, 1363–1373, 1872 | M | Shared header message |
| F22 | `huPerfHint` refers to "Section 2D", which no longer exists. Page title is still "Spinodoid Field Explorer". | 6, 176 | L | Wording |
| F23 | RNG seed lives in Directional bias, which is hidden for RD, but RD uses the seed and Roll writes it. | 218–253, 1036, 1259–1266 | M | Workflow; forgiving input |

## 3. Proposed section map

Four sections, one at a time (rail on desktop, tab bar on phone). Section ids for `.sec[data-section]`. Icons by name; `surface`, `volume`, `analyze` exist in the kit's sections widget, `direction` is new (drawn SVG, add to the kit).

| # | Section (id, icon) | Groups and controls (≤ 7 visible per group) |
|---|---|---|
| 01 | **Field** (`field`, `surface`) | **Type:** field type dropdown, RNG seed (moved here, F23). **GRF:** σ. **HU kernels:** aspect ratio, kernel width, N kernels, lock feature scale. **HU shape:** cross-section, edge sharpness, union blend, transverse aspect. **RD system:** system, preset. **RD parameters:** D_u, D_v, F / a, k / b, tile. **RD run:** sim steps, Simulate (progress), Roll. Only the active type's groups show. |
| 02 | **Direction** (`direction`, new icon) | mode (Single / Orthotropic / Isotropic), θ, φ (Single) or weight X/Y/Z (Ortho), κ (not in Iso), N waves, frequency (spinodoid, GRF). Max 6 at once. Disabled for RD (see decisions). |
| 03 | **Geometry** (`geometry`, `volume`) | topology (Sheet / Half-solid / Solid), flip side (Half-solid), center, half-width (not Half-solid), smoothing (if kept). |
| 04 | **Analyze** (`analyze`, `analyze`) | grid, E_s, ν, Run homogenization (progress); results: metric cards, Zener bar, stiffness glyph (sized to ~240 px). Results badge on the rail. |

Viewer overlay: clip radio X / Y / Z (drag handle stays), view menu (drawn icon, moved clear of the status pills), status line. Quality switch removed.
Header: help, Recipe menu (Open recipe…, Export JSON, Queue, Choose queue…, Mesh, Lab). The panel's Export button and the separate Mesh / Lab / Queue pills go into the Recipe menu.

## 4. Control → widget map

Cross-check by grep of the source: `<select` 4, `type="range"` 26, `type="number"` 2, `type="checkbox"` 1, static `<button` 20 (+1 inside the `#labNote` string). DOM at runtime adds 5 view-menu checkboxes and 36 buttons (help trigger, 30 `i`, help close, Queue main + caret, view, view reset). Playwright DOM counts match.

**Total mapped: 65 controls** = 4 selects + 26 ranges + 2 numbers + 6 checkboxes + 13 buttons in 4 segmented groups + 14 action buttons. Plus 30 help `i` buttons, native and unchanged.

### Selects (4)

| Id | Widget | Notes |
|---|---|---|
| `fieldType` | dropdown | relabel to short names: Spinodoid / Gaussian / Hyperuniform / Reaction-diffusion |
| `rdSystem` | dropdown | options get "(desktop only)" and `disabled` when the worker fails (824–835): needs disabled options in the kit dropdown |
| `rdPreset` | dropdown | options rebuilt by `repopulateRDPresetDropdown` (1176): needs a groups refresh in the kit or remount on `childList` |
| `homoGrid` | dropdown, compact | as TPMS `w-grid` |

### Ranges (26) — all `slider` (rows `.sc-group.sl[data-w-slider]`)

| Id | Format / options |
|---|---|
| `grfSigma` | 2 dp |
| `huAspect` | 1 dp |
| `huWidth` | 3 dp; hint "eff 0.0xx" when lock is on (`#huWidthEff`) |
| `huN` | integer |
| `huCross` | 2 dp; detents 1, 2 |
| `huSharp` | 2 dp |
| `huBlend` | shows `1/(1−0.875v)` (1.00–8.00) — `format`/`parse` |
| `huEll` | shows round / `a:1` / `1:a` (`4^v`) — `format`/`parse`; detent 0 |
| `rdDu`, `rdDv`, `rdF`, `rdK` | min/max/step/label rewritten by `swapRDParams` (1148): needs `setRange` + label refresh in the kit slider, or remount |
| `rdTile` | integer 1–4; detents 1..4 |
| `rdSteps` | integer; step 500 |
| `dirTheta`, `dirPhi` | `0°`; detents 0/90/180 (θ), 0/90/180/270/360 (φ) |
| `wX`, `wY`, `wZ` | 2 dp |
| `kappa` | 1 dp; hint line stays as row hint |
| `nWaves` | integer |
| `frequency` | 2 dp |
| `rngSeed` | integer (Roll writes it via `.value`: covered by `mirrorValue`) |
| `isoLevel` | 2 dp; detent 0 |
| `thickness` | 2 dp |
| `smoothing` | 2 dp — or remove (decision) |

### Number fields (2) — stay native

| Id | Reason |
|---|---|
| `homoEs`, `homoNu` | typed material constants, as TPMS `Es`/`nu`. Restyle to tokens, 44 px height. |

### Checkboxes (6)

| Id | Widget |
|---|---|
| `huLockScale` | toggle `sm` (wire needs a checkbox variant; current CONFIG.switches expects `role=switch` buttons) |
| view menu ×5 (Shadows, Occlusion, Warm / cool light, Shade cut faces, Lime edges) | toggles via `viewMenu:true` |

### Segmented groups (4 groups, 13 buttons)

| Buttons | Widget | Notes |
|---|---|---|
| `btnDirSingle`, `btnDirOrtho`, `btnDirIso` | segmented | sync on `class` |
| `btnSheet`, `btnHalf`, `btnSolid` | segmented | `btnSheet` hidden for HU (1054): needs per-option hide in the kit segmented, or remount with 2 options. Active state is inline `style`, not a class: sync on `style` |
| `clipPillX`, `clipPillY`, `clipPillZ` | radio `sm`, `allowNone` (CONFIG.clip `get:()=>clipState.axis`, `set:onClipPill`) | as TPMS |
| `btnQLow`, `btnQMed`, `btnQHigh`, `btnQUltra` | none: remove | replaced by adaptive render (decision on bake N) |

### Action buttons (14)

| Id / selector | Widget |
|---|---|
| `flipBtn` | toggle `sm` "flip side" (state is `halfInvert`; add `aria-pressed` for sync) |
| `rdRunBtn` | progressButton (worker progress `d.step/tot` feeds `setProgress`) |
| `rdRollBtn` | native secondary pill, drawn `refresh` icon |
| `runHomoBtn` | progressButton (`CONFIG.run`) |
| `exportBtn` | Recipe menu → Export JSON (`CONFIG.recipe.exportBtn`) |
| `.fh-mesh-btn` Mesh | Recipe menu → Mesh |
| `.fh-mesh-btn` Lab | Recipe menu → Lab |
| `.f13ldq-main` Queue | Recipe menu → Queue |
| `.f13ldq-caret` | Recipe menu → Choose queue… |
| `.f13h-trigger` help | native, drawn icon (help engine owns it) |
| `.f13v>button` view | native, drawn icon (shared drop-in) |
| `.f13v-r` view reset | native (shared drop-in) |
| `.f13h-close` | native, drawn `close` icon |
| `#labNote button` | removed: messages go to `F13UI.message` |

Not mapped: `.f13h-i` ×30 (help engine, native, keep), `#f13ldq-fallback` (only when the queue script fails; replace its `alert` with `F13UI.message`).

## 5. Render cost

- **Shader build:** `_buildFieldShader(params)` (917) emits GLSL ES 3.00 with the step count as a literal (`128/192/384/512`). Field data is a texture; geometry values are uniforms. Stiffness glyph: `buildStiffFrag` bakes compliance numbers as literals (956).
- **Recompile call sites:**
  - `setParams` → `_submitShader` when there is no previous params, when `!prog && !compiling` (F12), or when `quality` changes (945).
  - `rebuildStiffnessShader` once per homogenization run (959, 972). Fine.
- **Per tick:** no slider recompiles. Measured 20-tick drags: `isoLevel` 0, `thickness` 0, `kappa` 0 compiles. Quality change: 2 compiles + 1 link. **Lesson 1 holds**, except after a lost/failed program (F12).
- **Re-bake instead:** field sliders re-bake the 3D texture in the worker after 450 ms of debounce (F13). That is the real latency on drag, not compiles.
- **Quality tiers:** Low / Med / High / Ultra = steps 128 / 192 / 384 / 512 and bake 64³ / 96³ / 128³ / 256³; RD bakes 48³. Ultra is disabled when the worker fails. Frame time at 1070×352, swiftshader: ~25 ms in every tier (cost is shading: 48-step soft shadow + 5-tap AO + normals, not march length). Shadows already drop while orbiting/dragging (`uF13Interact`).
- **Loop:** continuous rAF (948); draws when `_dirty` or auto-rotate. Glyph: continuous at display rate (958).
- **Changes from `render-quality.md`:**
  1. Steps and hit threshold as uniforms (`uSteps` with a fixed loop bound + break). Remove the quality literal; keep a structure key so only real structure changes compile.
  2. Interactive tier: ~160 steps into an offscreen FB at adaptive scale 0.4–0.85, `blitFramebuffer` LINEAR. Context `antialias:false`. Keep `preserveDrawingBuffer:true` (queue `F13LD_snapshot` reads `#glCanvas`), or render a rest frame on snapshot.
  3. Rest tier after 150 ms idle: 384 steps, full res (DPR ≤ 2), in strips, one blit.
  4. Fixed canvas size; resize only from a ResizeObserver, not inside `render()`.
  5. On-demand loop (`requestRender` / `kick`; document `input`/`change` kick); sleep when idle. Auto-rotate keeps it awake only while on.
  6. Glyph: draw on change, ≤ 30 fps spin, pause while the main view moves, stop off-screen, respect Reduce Motion.
  7. Bake resolution decoupled from the removed quality switch (decision). Option: a 48³ preview bake while a field slider is held, full bake on release, to get under 400 ms.
  8. Fix F12: compile only when the structure key changes; guard null info logs; stop retrying on context loss and handle `webglcontextrestored`.

## 6. Recipe import

- **Format:** as in §1. `meta.tool:'grain'` (template `LEGACY` already maps `grain` → F13LD.grain). `TOOL.id='grain'`. `looksLike`: `r.field && r.geometry && ['spinodoid','gaussian','hyperuniform','reactiondiffusion'].includes(r.field.type)`.
- **Version keys:** `meta.version` `'0.9.4'` now. Pre-0.9.4: no `hu_cross/hu_sharp/hu_blend/hu_ell` → defaults 2 / 1 / 1 / 1. Pre-0.9.3 may carry `rd_Dv` for Gray-Scott → ignore for GS. `parseInt('0.9.4')` is 0, so `TOOL.major` must be 0 (or compare major.minor) for the newer-version warning to work.
- **Setters that exist:** `onFieldTypeChange()`, `onRDSystemChange()` (re-ranges RD sliders, repopulates presets, then applies the first preset, which overwrites center / half-width / topology / tile / steps), `onRDPresetChange()`, `onDirModeChange(mode)`, `onTopologyToggle(mode)`, `onHuLockChange()`, `onParamChange(input, displayId)`, `onRDParamChange(input, displayId)`, `onRDStepsChange(input)`, `regenerate()`. No setter for `halfInvert` (only the toggle `onFlipHalf`): set the global and `#flipGlyph.flipped`.
- **`plan` must:** map keys to controls; clamp to each slider's range (RD ranges depend on `rd_system`); invert transforms:
  - θ = acos(z), φ = atan2(y, x) from `principal_direction` (degrees, round to 1°). φ is undefined at θ = 0; export is identical either way.
  - `hu_blend` → v = (1 − 1/b)/0.875, round to 0.02. `hu_ell` → v = log₄(e), round to 0.05.
  - `hu_width` is the *effective* width when lock is on (`w·∛(30/n)`), off the slider grid. Lock state is not exported (decision).
- **`apply` order:** stub `regenerate` and `raymarcher.setParams` → `fieldType` → `rd_system` (+ `swapRDParams`) → RD values (`rd_Du`, `rd_Dv` unless GS, `rd_F`, `rd_k`, `rd_tile`, `rd_steps`; preset → `custom`) → `dir_mode` → θ/φ or weights → κ, N waves, frequency, seed → HU values (lock off) → σ → topology → `half_invert` → center, half-width, smoothing → restore stubs, one `regenerate`. RD recipes also need one `_bakeField` (RD only simulates on Simulate) — decision.
- RD values are exported even for non-RD fields (from hidden sliders); apply them too so the round trip stays exact.
- **Exclude:** the `homogenization` block. Clear `lastHomogResult`, hide `#homoResults`, reset the Export label on import. Round-trip test ignores `meta.timestamp` and `homogenization`.
- `CONFIG.recipe`: `{host:'w-recipe', exportBtn:'exportBtn', mesh:openInMesh, lab:openInLab}`. `openInLab`'s warnings move to `F13UI.message`.

## 7. Help system

- 36 cards, 31 with anchors, **30 `i` on the page** (clip card's anchor does not resolve, F16). Count must stay 30 (or 31 if the clip `i` is fixed — say which in stage 2).
- Engine: `ROW_SEL='.sc-group'`; `row` anchors find the row's `<label>` (or a wrapping label); `titleBefore:'#fieldType'` appends to the previous sibling (`.sec-lbl`). Labels with an id get the `i` as a sibling: `rdDuLbl`, `rdFLbl`, `rdKLbl` (the app rewrites their `innerHTML`).
- Ids that must survive: `clipPillX`, `fieldType` (+ a title element just before it), `grfSigma`, `huAspect`, `huWidth`, `huN`, `huCross`, `huSharp`, `huBlend`, `huEll`, `rdSystem`, `rdPreset`, `rdDu`, `rdDv`, `rdF`, `rdK`, `rdTile`, `rdSteps`, `btnDirSingle`, `dirTheta`, `dirPhi`, `wX`, `kappa`, `nWaves`, `frequency`, `rngSeed`, `btnSheet`, `isoLevel`, `thickness`, `smoothing`, `homoGrid`, plus `rdDuLbl`, `rdFLbl`, `rdKLbl`.
- Classes/ids the engine and drop-ins use: `.sc-group`, `.f13ld-header`, `.fh-right`, `.fh-mesh-btn` (queue anchor), `.f13h-*`, `f13h-card-<id>`, `#glCanvas` (snapshot), `#cw` (view menu host).
- Add the "show" button per card (`CONFIG.helpTargets`), mapping card id → control selector, so it switches section, scrolls and flashes.
- Help text to update: `smoothing` (F14), `export` card (Recipe menu), `quality` card to add (as TPMS "render quality · automatic").

## 8. Accent colour — to confirm with Matt

Current dominant colours (OKLCH L C h):

| Hex | Use | OKLCH |
|---|---|---|
| `#22d3ee` cyan | sliders, help accent, indicators, X axis (26 uses) | 0.797 0.134 211.5 |
| `#7c6ff7` violet | active pills, Run, Export after homog | 0.623 0.196 283.4 |
| `#f59e0b` amber | warnings, "working" state, Z-ish | 0.769 0.165 70.1 |
| `#18f370` green | section labels | 0.843 0.232 149.3 |
| `#3D3A30` / `#EDECE8` | tool name / tool icon tile (warm greige "grain") | 0.348 0.017 93 / 0.943 0.005 95 |

**Recommended: violet `oklch(0.80 0.15 290)`** (kit palette violet). Grain already uses violet for every active/selected state; cyan is shared with noise, beam and bundle, so violet keeps grain distinct. Clip axis colours (X cyan, Y magenta, Z amber) stay as data colours.
Alternative: keep aqua `oklch(0.84 0.14 200)` for continuity with today's sliders and help theme.
**To confirm with Matt.**

## 9. Risks

1. **Kit gaps.** Slider has no `setRange`/label refresh (RD sliders re-ranged by `swapRDParams`); dropdown has no groups refresh or disabled options (`rdPreset` rebuilt, `rdSystem` "(desktop only)"); segmented cannot hide one option (Sheet hidden for HU). Fix in `liquid-ui` (branch, tests, rebuild) or remount widgets on change.
2. **Removing the quality switch changes what users see:** it also picks the bake resolution (64³–256³). Ultra 256³ is 16.7 M voxels and runs on the main thread when the worker fails.
3. **Field sliders stay slow** (≥ 450 ms to first change) however smooth the widget is, unless a preview bake is added.
4. **Round trip:** `hu_width` with lock on is off-grid; RD state needs a simulation (seconds) after import; RD preset and quality are not exported.
5. **Help anchors:** wiring moves each row's `<label>` into the widget; RD label rewrites (`innerHTML`) and the `i` sibling placement must survive, and the count must stay 30.
6. **`preserveDrawingBuffer`** is needed by the queue snapshot; offscreen/strip rendering must keep the snapshot correct.
7. **Swiftshader harness** loses the first page's context; verifier must warm up first or it will report false errors.

## Decisions for Matt

- Accent: violet `oklch(0.80 0.15 290)` (recommended) or keep aqua?
- Low / Med / High / Ultra goes away with adaptive rendering. Bake resolution: fixed 96³, or 96³ while editing and 128³ at rest, or keep a "detail" choice in Geometry?
- `smoothing` does nothing today but is exported. Remove it, or make it work?
- Move RNG seed into Field so reaction-diffusion can see it (Roll already changes it)?
- Direction section for reaction-diffusion: shown disabled (rail stays the same) or hidden?
- Auto-rotate: keep on at start (keeps drawing), or start still?
- Add a `hu_lock_scale` key to the export so hyperuniform recipes with lock on round-trip exactly? (Mesh and Lab must ignore unknown keys; to check.)
- After importing a reaction-diffusion recipe: run the simulation automatically, or wait for Simulate?
- Fix the page title to "F13LD.grain" and the stale "Section 2D" hint?
