# AeroDeck Configurator — Reliability and Robustness Findings

**Context:** `agent work/improvments/`

This document captures concrete gaps between the current implementation and the goal of a trustworthy field configurator. The tool is meant to prevent an operator from walking away with a plan that "looks valid and collects poor data." The findings below address the five areas specified.

## 1. "IN SPEC" is achievable with marginal or poor overlap values

**Problem:** There is no floor or quality signal on front or side overlap. Plans with low overlap can still report as fully in spec.

**Current behavior:**
- The `inSpec` result in `compute()` is defined only as:
  ```js
  const frontOverlapFail = state.frontOverlap > overlapCeilingRaw;
  const blurFail = state.speedMs > blurMaxSpeed;
  const inSpec = !frontOverlapFail && !blurFail;
  ```
- Only an *upper* ceiling on front overlap (from speed × along-track footprint × capture rate) is checked. There is no minimum acceptable overlap.
- `sideOverlap` is stored, displayed (`el.sideOverlapOut`), and has its own slider, but it never enters the return value of `compute()`, never affects `inSpec`, binding state, or any derived calculation.
- Slider ranges are static (`front: 50–95%`, `side: 50–90%`).
- Architecture §3 describes side overlap as "a pure coverage/robustness control" that is "independent of speed and blur." This is correct for the physics model but means low values produce no visible consequence.

**Effect:** 50%/50% (or any value under the front ceiling) can display a clean green "IN SPEC" with no binding highlights or other warnings.

## 2. Terrain flag (Dynamic AGL) is purely decorative

**Problem:** The checkbox changes nothing about the numbers or what the pinned spec means.

**Current behavior:**
- `state.terrainDynamic` is set in `readInputs()`.
- The only consumer is the display string in `repaint()`:
  ```js
  el.terrainNote.textContent = state.terrainDynamic
    ? 'Terrain: dynamic AGL'
    : 'Terrain: static AGL (flat ground assumed)';
  ```
- `terrainDynamic` is never read inside `compute()`. It has zero effect on GSD, footprint, ceilings, blur, or `inSpec`.
- The pinned GSD or altitude continues to be treated as an exact value either way.

**Supporting statements from Architecture.md:**
- §3: "Terrain mode — Flag — Static vs. dynamic AGL declaration. Annotates outputs; does no terrain math."
- §8: "No DEM / terrain drape. Terrain is a declaration of intent, not a computation."

**Effect:** Selecting dynamic AGL gives the operator a different line of small text and nothing else. No GSD range, no uncertainty, no change to the verdict or binding logic.

## 3. Multiple soft assumptions are invisible and non-adjustable

The model contains several simplifying assumptions that are either hardcoded constants or implicit and have no operator controls or prominent effect on outputs.

**Key examples:**
- Blur budget: `const MAX_BLUR_PX = 1;` (line 250). Used for both `blurCeiling()` and the "Blur margin" readout. Not exposed in the UI.
- Capture rate: Documented as "max sustained trigger rate." Values are fixed literals on the sensor objects. No distinction between theoretical rate and real sustained rate (card speed, file type, buffer).
- Orientation: `footprint()` always assigns `heightPx` to along-track. No landscape/portrait choice; this directly affects the front-overlap ceiling.
- Terrain: "flat ground assumed" is only mentioned in the static-case note. No relief/terrain variation parameter exists.
- Flight dynamics: All formulas assume constant speed on straight lines. Acceleration, turns, and turnarounds are not modeled.
- Trigger behavior: The model assumes the camera can be triggered at the exact computed interval with no additional real-world constraints.

These are consistent with the narrow "closed-form physics only" scope, but because a strong "IN SPEC" verdict is shown, their impact is not visible to the operator.

## 4. Front overlap target can still be set above its physical ceiling

**Problem:** The front-overlap slider permits (and retains) values above the computed capture-rate ceiling. The violation is highlighted but not prevented.

**Current behavior:**
- The HTML slider has fixed attributes: `min="50" max="95" step="1"`.
- In `repaint()`:
  - The ceiling is shown as a live readout.
  - `setBindingHighlight` applies the `binding` class to the row and ceiling wrap.
  - The binding note explains the constraint.
- `state.frontOverlap` keeps the value the user set. Nothing clamps the input or blocks an unachievable target.
- `inSpec` will be false (verdict shows OUT OF SPEC), but the numeric value the operator may still plan to use remains invalid.

**Effect:** The UI can make the problem obvious after the fact, but does not stop the configuration of an impossible front-overlap target.

## 5. Only a binary physics verdict exists

**Problem:** The large verdict banner is driven solely by the two hardware physics limits. No secondary robustness or quality indicator is present.

**Current behavior:**
- `verdictState` and the pin text are set exclusively from the boolean returned by `compute()`.
- The only two conditions that matter are `frontOverlapFail` and `blurFail`.
- Values that are computed and shown but do **not** affect the verdict include:
  - Chosen front overlap (as long as it is below the ceiling)
  - Side overlap (any value)
  - Blur margin (px and m/s headroom)
  - Terrain mode
- The derived section and binding note provide additional detail, but the prominent top band remains strictly binary.

**Architecture framing (for context):**
- §4: The pinned end "draws a line the live values must respect."
- The verdict is intentionally limited to physical executability against that line.

**Effect:** "IN SPEC" currently means only "the camera can physically take the pictures without obvious motion blur or dropped frames." It communicates nothing about expected reconstruction quality, data robustness, or sensitivity to unmodeled terrain variation.

## Summary Table

| Area                          | Affects "IN SPEC"? | Visible effect?              | Operator control? | Notes |
|-------------------------------|--------------------|------------------------------|-------------------|-------|
| Front overlap (upper ceiling) | Yes                | Ceiling + binding highlight  | Partial (shown, not blocked) | Target can remain invalid |
| Front/Side overlap (minimum)  | No                 | None                         | No                | 50% treated like 80% if under ceiling |
| Side overlap                  | No                 | Raw % only                   | No                | Never participates in logic |
| Dynamic AGL flag              | No                 | Text annotation only         | No                | No numeric impact |
| Blur budget (1 px)            | Implicit           | Derived readout only         | No                | Hardcoded |
| Capture rate "sustained"      | Implicit           | Comments only                | No                | No real-world rate distinction |
| Orientation / flat ground     | Implicit           | Minimal                      | No                | Hardcoded assumptions |
| Verdict model                 | Binary physics only| Large banner                 | —                 | No combined quality/robustness signal |

## Implications

These gaps are consequences of the deliberate narrow scope rather than implementation bugs. Because the tool is positioned as the reliable resolver of "what alt and overlap should I use?", the current "IN SPEC" state can still allow plans that are physically executable but weak for reconstruction or sensitive to unstated real-world conditions.

This document is intended as the seed for improvement work in the `improvments` folder.
