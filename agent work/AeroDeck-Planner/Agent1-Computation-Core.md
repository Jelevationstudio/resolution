# Agent 1: Computation Core

**Depends on:** nothing (first phase)
**Blocks:** Agent 2

---

## Goal
Implement the closed-form computation core and the sensor profile data as a `const`, with correct values and locked variable roles — no UI.

## Deliverables
1. `prod/aerodeck-planner.html` created, containing a `<script>` with:
   - `SENSORS` — a `const` object literal of sensor profiles (e.g. X10 wide, Mavic 3E), each resolving **focal length**, **pixel pitch**, **sensor resolution** (image width/height in px), and **max capture rate** (Hz; use `3` for every sensor for now — each gets its own real value later, so it must be a per-sensor field, not a shared constant). Data, not logic.
   - Pure computation functions implementing architecture §4:
     - `gsd(altitude, sensor)` = `pixel pitch × altitude / focal length` (the bijective lock).
     - `altitudeFromGsd(gsd, sensor)` = the inverse (the other end of the same axis).
     - `footprint(gsd, sensor)` → ground footprint (along-track and cross-track) from GSD and sensor resolution.
     - `maxSpeedForFrontOverlap(targetFrontOverlap, alongTrackFootprint, captureRate)` → the speed at which the required trigger interval hits the camera's min interval (`1 / captureRate`). Above this speed, the target front overlap is physically unachievable. Returns the **speed wall**.
     - `maxFrontOverlapAtSpeed(speed, alongTrackFootprint, captureRate)` → the **derived overlap ceiling**: the highest front overlap achievable at the current speed/altitude. The exact inverse view of `maxSpeedForFrontOverlap` — both describe one constraint surface across {speed, along-track footprint, front overlap}; pinning any two yields the third's limit. This is the value shown as a readout next to the (free input) front-overlap slider.
     - Front overlap itself is **never computed** — it is the operator's free input. We compute only its *ceiling*. This is a soft clamp, not a rigid lock: the operator may set any overlap at or below the ceiling.
     - `sideOverlap` is set by line spacing and is independent of speed — no capture-rate interaction.
     - `blurCeiling(...)` ≈ `speed × shutter / GSD` → the speed *maximum* before blur breaks (a clamp, not a value). This is speed's *other* ceiling, independent of the capture-rate wall above.
2. No `<body>` UI required yet (a minimal or empty body is fine). Functions must be callable/inspectable.

## Reference Files
- [`../../AeroDeck-Flight-Planner-Architecture.md`](../../AeroDeck-Flight-Planner-Architecture.md) §3 (variable model), §4 (computation core).

## Before coding (CLAUDE.md §1)
State explicitly, in your handoff, the **units** for every input and output (m, mm, µm, cm/px, m/s, s, %) and any conversion constants. Unit consistency is the whole game here; document it in code comments. If a formula in §4 is ambiguous on units, state your assumption — do not pick silently.

## Key traps
- **Altitude and GSD are one axis.** Provide both `gsd()` and `altitudeFromGsd()` as exact inverses. Do not build anything that treats them as two independent inputs.
- **Blur ceiling is a clamp, not a derivation.** It returns a max speed; it must not set speed.
- **Front overlap is a free input with a derived ceiling — not a hard relation.** AeroDeck is the configurator, not the planner — the operator's front overlap slider is the number they dial into the flight controller, and they spend it freely (lower for faster plans, higher for robustness). We do **not** reproduce the planner's trigger-interval math and we never compute the overlap *value*. We compute only its **ceiling** (`maxFrontOverlapAtSpeed`) and the symmetric **speed wall** (`maxSpeedForFrontOverlap`). Soft clamp, adjustable — not the rigid bijective lock of altitude↔GSD.
- **Front overlap clamp couples to speed; side overlap does not.** The capture-rate clamp depends on speed; side overlap is pure coverage and has no speed interaction.
- Secondary factors (light, terrain) do **not** appear in this phase as numbers. Light's only role here, if any, is supplying the shutter value the blur ceiling consumes.

## Acceptance Criteria
- [ ] `SENSORS` is a `const` object literal; at least two real profiles with focal length, pixel pitch, resolution, and a per-sensor `captureRate` field (= `3` for now).
- [ ] `gsd()` and `altitudeFromGsd()` are exact inverses (round-trip a value, get it back).
- [ ] `footprint`, `maxSpeedForFrontOverlap`, `maxFrontOverlapAtSpeed`, `blurCeiling` return correct values for hand-checked inputs.
- [ ] `maxSpeedForFrontOverlap` and `maxFrontOverlapAtSpeed` are exact inverses on the same constraint surface (round-trip a value, get it back).
- [ ] `maxSpeedForFrontOverlap` decreases as target front overlap increases; `maxFrontOverlapAtSpeed` decreases as speed increases; raising altitude raises both ceilings. Side overlap has no speed dependency.
- [ ] Units documented in comments; conversions explicit and consistent.
- [ ] No HTML UI, no libraries, no single-use abstractions (CODE-DISCIPLINE.md).
- [ ] CHANGELOG.md entry appended.
