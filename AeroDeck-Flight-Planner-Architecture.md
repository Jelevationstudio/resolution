# AeroDeck Flight Planner — Architecture

## 1. Purpose & Scope

A single-file, offline field tool that re-couples the flight-planning variables existing planners present as independent fields. It does not control the camera, does not execute missions, and does not perform flight-time or DEM/terrain math. Its reason for being: vendor guidance answers "what altitude and overlap?" with an unconditioned range that is correct but useless because no constraints have been spent. The tool is the function that collapses that range to a single point once the operator declares their constraints—making the physical coupling between resolution, altitude, speed, and front/side overlap visible in real time so an operator cannot build a plan that looks valid and collects poor data.

## 2. Platform & Delivery

- **Target:** Chrome on Android, opened from local storage via `file://`.
- **Form factor:** Single self-contained `.html` file. CSS inlined in `<style>`, JS inlined in `<script>`, sensor profiles as a `const` object literal.
- **Rationale:** Chrome on Android sandboxes `file://` and blocks reliable access to sibling files (external CSS/JS/JSON). Inlining everything sidesteps all cross-origin and file-access restrictions, requires zero `fetch` calls, no build step, and no hosting.
- **Footprint:** Tens of kilobytes. No libraries, no dependencies.

## 3. Variable Model

The system has **three degrees of freedom**, not six co-equal inputs: the altitude/GSD axis, speed, and overlap (now split into front and side). Treating every value as simultaneously editable over-determines the system and has no single solution. The critical correction: **GSD and altitude are not independent variables—they are the same axis viewed from opposite ends**, rigidly locked by the sensor, so they can never both be live knobs. The model breaks circularity by declaring roles:

| Variable | Role | Behavior |
|---|---|---|
| **Altitude / GSD axis** | One axis, pinned at one end | Bijective lock: GSD = pixel pitch × altitude / focal length. A toggle picks which end is held (resolution spec, or hard altitude); the held end is a field, the other end is a derived live readout. They are never both editable. |
| **Speed** | Always-free knob | Live slider beneath a ceiling. Altitude does **not** derive speed—it sets speed's blur ceiling (clamp), not its value. Speed stays free under that clamp. |
| **Front overlap** | Live knob, coupled to speed | Live slider. Along-track; coupled to speed via the trigger interval, so pushing speed up eats front overlap. Shares speed's failure mode. |
| **Side overlap** | Live knob, independent | Live slider. Cross-track; set by line spacing. Independent of speed and blur—a pure coverage/robustness control. |
| **Light** | Flag / clamp | Categorical box. Resolves to a shutter-floor implication, never a number. |
| **Terrain mode** | Flag | Static vs. dynamic AGL declaration. Annotates outputs; does no terrain math. |

The independent variables are always the sliders; the readout values are always dependent. The operator cannot edit a derived value directly — they discover it by moving the knobs. The pinned-end toggle determines which of altitude/GSD is a slider and which is a readout.

## 4. Computation Core

All closed-form arithmetic, microseconds per tick. No learned model, no new science.

- **GSD** = (pixel pitch × altitude) / focal length. Pixel pitch and focal length come from the selected sensor profile. This is a bijective lock—pin either GSD or altitude and the other is exactly determined.
- **Footprint** derived from GSD and sensor resolution.
- **Front overlap actual** derived from footprint, capture rate, and speed (trigger interval). This is where speed eats along-track overlap.
- **Side overlap actual** derived from footprint and flight-line spacing—independent of speed.
- **Blur ceiling** ≈ (speed × shutter) / GSD. Altitude sets speed's *maximum* before blur breaks, not speed's value. Since GSD rises with altitude, flying higher raises the speed ceiling—a clamp, not a derivation. Blur scales with angular rate, so it bites harder at low altitude for the same speed.
- **Spec check** = the pinned end of the altitude/GSD axis drawn as a threshold the live readout crosses or violates.

The pinned end does not solve the system — it draws a line the live values must respect. As the operator drags speed and overlap, the readout shows the moment the plan falls out of spec.

## 5. Coupling Logic (the actual product)

The value is not the math; it is forcing the coupling onto one surface where moving any setting visibly moves the consequences of the others. The central distinction the tool teaches is **rigid derivation vs. clamp**:

- **Altitude ↔ GSD is a bijective lock.** Pinning one exactly determines the other through the sensor. They share one axis; the operator picks which end to hold.
- **Altitude ↔ speed is a ceiling, not a derivation.** Altitude raises the blur ceiling that caps speed; it does not set speed. Speed stays free underneath.
- **Speed ↔ front overlap is a coupling.** Pushing speed up eats along-track overlap through the trigger interval—so speed has two simultaneous consequences (blur ceiling and front-overlap collapse) that light up together.
- **Side overlap is standalone.** A coverage/robustness knob, independent of speed and blur.
- **Light** sets the shutter floor that the blur ceiling depends on.

The teaching moment — *fast and low can't both be free, and fast costs front overlap too* — emerges from this adjacency, not from a solve or a warning lookup. The operator running 40/30 at high speed watches front overlap fall out of spec as a direct result of the speed they chose, which is exactly the silent failure a vendor range cannot warn against.

## 6. UI / UX Layout

Mobile-first from the start, not retrofitted. Scroll is disabled; the entire interface lives in one viewport.

- **Top band — verdict.** Pinned spec + live in-spec / out-of-spec state, large and color-coded. The first thing the eye lands on.
- **Pinned-end toggle.** Selects which end of the altitude/GSD axis is held. This swaps which control is a live slider and which is a derived readout, so the layout must support a slider↔readout swap in that slot.
- **Middle — the live sliders.** The derived end of the altitude/GSD axis is shown as a readout; the held end plus speed, front overlap, and side overlap are sliders. Large touch targets. Front overlap sits adjacent to speed (shared failure mode); side overlap stands alone.
- **Bottom — derived readout.** The consequences that move (the derived altitude or GSD, blur margin, front/side overlap actuals), with the **binding constraint highlighted**.
- **Edge controls — flags.** Sensor profile dropdown, light category box, terrain toggle.

### UX constraints
- `dvh` units to handle the mobile address bar; fixed layout so nothing scrolls during input.
- `clamp()`-based type sizing for glanceable readout typography.
- High contrast, large hit areas. Readability decided at design time, not adjusted to fit.
- Defined styling to be supplied separately and applied within these structural rules.

## 7. Reactivity Model

- Sliders drive an `oninput` handler that recomputes and repaints the readout on every frame.
- No calculate button — the system is never over-determined because roles are fixed (sliders independent, readout dependent) and the pinned-end toggle guarantees altitude/GSD are never both live.
- The pinned-end toggle re-binds which control is a slider vs. a readout, then repaints.
- Flags (light, terrain) update state and trigger the same repaint.

## 8. Explicit Non-Goals

- No flight-time calculation. Flight time appears only as a relationship reference, not a computed output.
- No exposure control. The camera owns ISO/shutter; the tool only warns when conditions push it into a corner.
- No DEM / terrain drape. Terrain is a declaration of intent, not a computation.
- No mission execution, waypoint export, or aircraft communication.
- No network, no stored ephemeris, no live data.

## 9. File Structure

```
aerodeck-planner.html        ← entire application, self-contained
```

Optional, only if it grows beyond comfortable single-file maintenance:

```
/aerodeck-planner/
  index.html                 ← still the deliverable; inlined for field use
  /src/                      ← un-inlined working copies (dev only, not shipped)
```

The shipped artifact is always one inlined HTML file.
