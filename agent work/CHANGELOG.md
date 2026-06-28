# Changelog

Cross-session handoff log. Newest entry first. Append one entry per completed pass using the template in [`README.md`](README.md) § Changelog. Do not rewrite or delete prior entries.

---

<!-- New entries go directly below this line, newest first. -->

## AeroDeck Planner — Visual Design + Polish (2026-06-28)

| Field | Value |
|-------|--------|
| **Agent / pass** | AeroDeck Planner / Agent 3 |
| **Agent brief** | `Agent Work/AeroDeck-Planner/Agent3-Visual-Design.md` |
| **Status** | `shipped` |
| **Depends on** | Agent 2 functional handoff in `prod/aerodeck-planner.html`, Architecture §6 |
| **Unblocks** | — (final phase) |

### Goal (one sentence)
Apply full mobile-first visual treatment to the working planner — CSS only, zero computation or reactivity changes.

### Changed
- **App / engine:** `prod/aerodeck-planner.html` — inlined `<style>` block, viewport meta, `class` hooks on existing body elements (`app`, `slider-block`, `speed-row`, `side-overlap-row`). **16485 bytes** (`wc -c prod/aerodeck-planner.html`). Script block byte-identical to Agent 2 handoff — verified by diff.

### Behavior / contract delta
- **Layout:** one viewport (`100dvh`, `overflow:hidden`); body CSS grid stacks verdict → pin toggle → sliders → derived → flags.
- **Verdict band:** large `clamp()` typography; green/red color-coded via `.in-spec` / `.out-of-spec` on `#verdict-state` and `:has()` band background.
- **Coupling visual:** speed + front overlap share blue left accent; side overlap standalone with neutral accent.
- **Binding highlight:** `.binding` amber inset border/background on speed row, front-overlap row, ceiling wrap, blur-margin row (unchanged JS targets).
- **Touch/readability:** 44px min hit areas on radios, selects, checkbox; large range thumbs; high-contrast dark instrument theme. No external assets.

### Verify
- Commands run: `diff` Agent 2 vs Agent 3 `<script>` blocks; `wc -c prod/aerodeck-planner.html`; all handoff `id`s present
- Result: `pass` (16485 bytes; script diff exit 0)

### Deferred
- Real per-sensor capture rates — **placeholder 3 Hz** — **sensor data pass**

### Pitfalls / do not redo
- Do not edit `<script>` in polish passes — styling is CSS + class hooks only.
- Do not wrap slider rows in new DOM nodes — JS uses `.parentElement` for binding highlight.
- Negative overlap ceiling display clamp unchanged; verdict uses raw value.

### Next agent should
- — (AeroDeck Planner phase complete). Ship `prod/aerodeck-planner.html` when ready.

## AeroDeck Planner — Structure + Reactivity (2026-06-28)

| Field | Value |
|-------|--------|
| **Agent / pass** | AeroDeck Planner / Agent 2 |
| **Agent brief** | `Agent Work/AeroDeck-Planner/Agent2-Structure-Reactivity.md` |
| **Status** | `shipped` |
| **Depends on** | Agent 1 computation core in `prod/aerodeck-planner.html`, Architecture §5–§7 |
| **Unblocks** | Agent 3 (Visual Design + Polish) |

### Goal (one sentence)
Wrap the computation core in functional HTML — sliders, readouts, pinned-end toggle, flags — with live `oninput` reactivity and no styling.

### Changed
- **App / engine:** `prod/aerodeck-planner.html` — full `<body>` structure, UI state/repaint layer, `LIGHT` shutter-floor const; Agent 1 functions unchanged. No `<style>` block yet (Agent 3). **11544 bytes** (`wc -c prod/aerodeck-planner.html`).

### Behavior / contract delta
- **Reactivity:** every slider, select, checkbox, and pin-end radio calls `repaint()` on input/change. No calculate button.
- **Altitude/GSD axis:** one held slider (`#axis-held-slider`) + one derived readout (`#axis-derived-out`). Pin-end radios (`name="pinEnd"`, values `gsd` | `altitude`) call `onPinToggle()`, which preserves the physical point on the axis, rebinds slider min/max/step via `bindAxisSlider()`, then repaints. Never both editable.
- **Free inputs (operator dials):** speed (`#speed`, 1–25 m/s), front overlap (`#front-overlap`, 50–95%), side overlap (`#side-overlap`, 50–90%). Overlap *values* are never recomputed by the tool.
- **Derived readouts:** opposite end of alt/GSD axis; blur margin px + m/s headroom; front-overlap ceiling (`maxFrontOverlapAtSpeed`) beside front-overlap slider in `#front-overlap-ceiling-wrap`. Ceiling **display** clamped to ≥ 0; **verdict** uses raw ceiling (may be negative).
- **Verdict:** `#verdict-state` text `IN SPEC` / `OUT OF SPEC`; class `in-spec` or `out-of-spec`. `#verdict-pin` shows pinned end value (declared constraint, not pass/fail). Out-of-spec when `frontOverlap > overlapCeilingRaw` (capture rate) or `speedMs > blurMaxSpeed` (blur) only — pinned alt/GSD cannot violate itself (bijective lock); derived end is exact, not gated. Side overlap does not affect verdict. Intentional per §4: pinned end draws the line; free knobs (speed, overlap) are what cross it.
- **Binding highlight:** class `binding` toggled on `#front-overlap-row`, `#front-overlap-ceiling-wrap`, speed row parent, blur-margin row when that constraint is active. `#binding-note` carries human-readable binding text.
- **Flags:** `#sensor` dropdown (from `SENSORS`), `#light` dropdown (from `LIGHT`), `#terrain-dynamic` checkbox → `#terrain-note` annotation only (no terrain math).
- **`LIGHT` shutter floors (seconds):** overcast `1/500`, bright midday `1/2000`, low angle `1/400`. Passed to `blurCeiling(..., MAX_BLUR_PX)` with `MAX_BLUR_PX = 1`.
- **Defaults:** Mavic 3E, pin GSD @ 2.0 cm/px, speed 8 m/s, front 80%, side 70%, overcast, static AGL.

### DOM map (for Agent 3 styling)
| Region | IDs |
|--------|-----|
| Verdict band | `#verdict`, `#verdict-state`, `#verdict-pin` |
| Pin toggle | `#pin-toggle` |
| Sliders | `#sliders`, `#axis-held`, `#axis-held-label`, `#axis-held-slider`, `#axis-held-out`, `#axis-derived`, `#axis-derived-label`, `#axis-derived-out`, `#speed`, `#speed-out`, `#front-overlap-row`, `#front-overlap`, `#front-overlap-out`, `#front-overlap-ceiling-wrap`, `#front-overlap-ceiling`, `#side-overlap`, `#side-overlap-out` |
| Derived | `#derived`, `#blur-margin`, `#blur-speed-headroom`, `#binding-note` |
| Flags | `#flags`, `#sensor`, `#light`, `#terrain-dynamic`, `#terrain-note` |

### Agent 2 acceptance (all met)
- [x] All sliders move and repaint live; no calculate button.
- [x] Pin-end toggle swaps slider↔readout and repaints.
- [x] Front/side overlap are free inputs; overlap values never recomputed.
- [x] `maxFrontOverlapAtSpeed` shown beside front overlap; drops with speed, rises with altitude.
- [x] Target > ceiling → out-of-spec (capture); lowering target or raising altitude clears it.
- [x] Blur ceiling crossing → out-of-spec (blur); side overlap independent of speed.
- [x] Verdict + binding constraint highlighted via `.binding`.
- [x] Sensor, light, terrain flags repaint.
- [x] Derived values not user-editable.
- [x] No libraries; no duplicated Agent 1 math.

### Verify
- Commands run: Node round-trip + monotonicity (Mavic 3E @ 100 m: ceiling drops with speed, rises with altitude; blur fail at 20 m/s; raising alt clears capture fail); `wc -c prod/aerodeck-planner.html`
- Result: `pass` (11544 bytes)

### Deferred
- Visual treatment (`<style>`, dvh, clamp typography, color-coded verdict) — **Agent 3** — **Agent 3**
- Real per-sensor capture rates — **placeholder 3 Hz** — **sensor data pass**

### Pitfalls / do not redo
- Do not compute front overlap as an output; ceiling readout only.
- Do not duplicate GSD/overlap/blur math in UI — call `gsd`, `altitudeFromGsd`, `footprint`, `maxFrontOverlapAtSpeed`, `blurCeiling`.
- Negative `maxFrontOverlapAtSpeed` is intentional; display clamps at 0, verdict uses raw value.
- Altitude slider bounds when pin=altitude are `altitudeFromGsd(0.5|10, sensor)` — same axis as GSD 0.5–10 cm/px; rebind on sensor change.
- No max-altitude regulation gate on derived altitude — out of scope unless product adds a separate ceiling input later.
- Agent 3: style only. Do not edit `compute()`, `repaint()`, `onPinToggle()`, or Agent 1 functions.

### Next agent should
1. Read `Agent Work/AeroDeck-Planner/Agent3-Visual-Design.md` and Architecture §6.
2. Add inlined `<style>` in `prod/aerodeck-planner.html` — style existing structure; hook `.in-spec`, `.out-of-spec`, `.binding`. Place front overlap adjacent to speed; side overlap standalone.
3. Record post-style `wc -c` in Agent 3 CHANGELOG entry. Do not touch JS logic or reactivity wiring.

## AeroDeck Planner — Computation Core (2026-06-28)

| Field | Value |
|-------|--------|
| **Agent / pass** | AeroDeck Planner / Agent 1 |
| **Agent brief** | `Agent Work/AeroDeck-Planner/Agent1-Computation-Core.md` |
| **Status** | `shipped` |
| **Depends on** | Architecture §3–§4, `Agent Work/CODE-DISCIPLINE.md` |
| **Unblocks** | Agent 2 (Structure + Reactivity) |

### Goal (one sentence)
Implement the closed-form computation core and sensor profiles in `prod/aerodeck-planner.html` with no UI.

### Changed
- **App / engine:** `prod/aerodeck-planner.html` — new file with `SENSORS` (X10 wide, Mavic 3E) and pure functions: `gsd`, `altitudeFromGsd`, `footprint`, `maxFrontOverlapAtSpeed`, `maxSpeedForFrontOverlap`, `blurCeiling`.

### Behavior / contract delta
- Unit contract documented in script header: altitude (m), GSD (cm/px), focal length (mm), pixel pitch (µm), footprint (m), speed (m/s), shutter (s), capture rate (Hz), front overlap (fraction 0–1), blur budget (px).
- GSD lock: `gsd = pixelPitchUm × altitudeM / (focalLengthMm × 10)`; exact inverse in `altitudeFromGsd`.
- Footprint: along-track = GSD(m/px) × heightPx; cross-track = GSD(m/px) × widthPx.
- Front-overlap constraint surface: `maxFrontOverlapAtSpeed = 1 − speed / (alongTrack × captureRate)`; `maxSpeedForFrontOverlap` is the inverse. Front overlap value is never computed — ceiling only.
- Blur ceiling: `maxSpeed = maxBlurPx × GSD(m/px) / shutterS`.
- Per-sensor `captureRateHz: 3` placeholder on both profiles.

### Verify
- Commands run: Node VM round-trip and monotonicity checks on Mavic 3E @ 100 m AGL
- Result: `pass`

### Deferred
- Real per-sensor capture rates — **placeholder 3 Hz on all profiles** — **sensor data pass or Agent 2+**
- Light/terrain flags — **Agent 2 scope** — **Agent 2**
- UI, styling, reactivity — **Agent 2 / Agent 3** — **Agent 2**

### Pitfalls / do not redo
- Do not compute front overlap as an output; only `maxFrontOverlapAtSpeed` (ceiling) and `maxSpeedForFrontOverlap` (speed wall).
- Do not treat altitude and GSD as independent inputs.
- `blurCeiling` returns a max speed clamp; it does not set speed.
- Side overlap has no capture-rate math in this core (independent knob).

### Next agent should
1. Open `Agent Work/AeroDeck-Planner/Agent2-Structure-Reactivity.md` and build HTML sliders, pinned-end toggle, flags, and `oninput` wiring on top of `prod/aerodeck-planner.html`.
2. Call existing functions only; do not rewrite overlap or GSD math.
3. Pass explicit `maxBlurPx` (typically 1) and shutter from the light flag into `blurCeiling`.
