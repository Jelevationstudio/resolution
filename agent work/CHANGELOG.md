# Changelog

Cross-session handoff log. Newest entry first. Append one entry per completed pass using the template in [`README.md`](README.md) § Changelog. Do not rewrite or delete prior entries.

---

<!-- New entries go directly below this line, newest first. -->

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
