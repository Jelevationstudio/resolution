# Agent 1: Terrain Contributor

**Depends on:** Scoring-Engine / Agent 1 (framework + `Scoring-Spec.md` + contributor interface must exist)
**Blocks:** nothing (single pass for this folder)

---

## Goal
Add a categorical terrain-type selector and register a terrain robustness contributor whose penalty is the selected terrain weight under **static** AGL and ~0 under **dynamic** AGL — no DEM, no terrain math.

## Deliverables
1. **`TERRAIN_TYPES` const** — an object literal modeled on `LIGHT`, with **4 entries from flat to mountainous**, each `{ label, weight }` and a one-line rationale comment. **Escalation is non-linear, not evenly spaced** (values tunable, aligned to the Scoring-Engine penalty scale):
   - Flat / desert → ~0 (negligible relief)
   - Gentle / rolling → small (minor, noticeable issues)
   - Hilly / variable → moderate
   - Mountainous / steep → **heavily / dominantly weighted** — static AGL here can mean hundreds of feet of uncompensated vertical change; the jump from hilly to mountainous is large, not incremental, and may drive the estimate into a clearly poor band on its own.
2. **New DOM control** — a terrain-type `<select>` in the `#flags` area, populated from `TERRAIN_TYPES` exactly as the light `<select>` is populated. New `id` (e.g. `#terrain-type`). **Keep** the existing `#terrain-dynamic` checkbox and `#terrain-note`.
3. **State + wiring** — `state.terrainType` (default the flattest = no penalty until the operator declares relief), read in `readInputs()`, `onchange` → `repaint()` via the existing reactive path.
4. **Contributor in `compute()`** — terrain penalty = `state.terrainDynamic ? 0 : TERRAIN_TYPES[state.terrainType].weight`. Emit a `{ id:'terrain', label, penalty, reason }` contribution into the array passed to `scorePlan()`. Reason names the active combination (mode + terrain type).
5. **Weights placement** — keep per-category relief weights in `TERRAIN_TYPES` (categorical data, like light's shutter floors); if `Scoring-Spec.md` requires central registration, add a single `terrain` scale entry in `SCORE_WEIGHTS` that references them. Follow the spec's convention; document either way.

## Reference Files
- [`../Scoring-Engine/Scoring-Spec.md`](../Scoring-Engine/Scoring-Spec.md) — contributor interface, decomposition format, weight convention.
- [`Agent0-Overview.md`](Agent0-Overview.md) — the settled selector + interaction model.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §2.
- `prod/aerodeck-planner.html` — target. Model the new select on the existing `LIGHT`/`#light` selector and the `#terrain-dynamic` wiring.

## Key traps
- **Categorical only — no terrain math.** The selector resolves to a documented weight, exactly like the light selector resolves to a shutter floor. No DEM, no relief parameter, no GSD-range computation.
- **Weighting is proportional to relief severity and escalates non-linearly.** Mountainous must carry far more than a linear step above hilly — static AGL over mountains (100s of ft of vertical change) is a near-disqualifying robustness risk. Do not space the four weights evenly.
- **Interaction model:** static AGL → apply terrain weight; dynamic AGL → suppress to ~0. Keep it decompositional and explicit.
- **Score-only — never the gate.** Terrain never touches `frontOverlapFail`, `blurFail`, or `inSpec`. It cannot flip OUT OF SPEC.
- **Default flattest** so a fresh plan carries no terrain penalty until the operator declares rougher terrain (declare-your-constraints ethos).
- **Use the framework.** Emit through the contributor interface; don't re-implement aggregation, banding, or the est% cap.
- Engine work: new DOM/`id` for the selector is permitted; preserve all existing `id`s and the binding-highlight `.parentElement` chain. Style the select with brand tokens, 44px target, within one viewport. Single-file/offline/size discipline stand.

## Acceptance Criteria
- [ ] `TERRAIN_TYPES` has 4 options flat→mountainous, each weighted with a rationale comment.
- [ ] A terrain-type `<select>` appears in `#flags`, populated from `TERRAIN_TYPES`, brand-styled, 44px target, repaints on change; existing `#terrain-dynamic` toggle retained.
- [ ] **Static AGL:** `estPct` drops by the selected terrain weight with a specific reason; flat/desert = 0, mountainous = max.
- [ ] **Dynamic AGL:** terrain penalty is ~0 for every terrain type (suppressed); reason reflects the mitigation (or none emitted).
- [ ] Gate booleans unchanged — terrain never produces OUT OF SPEC; verified.
- [ ] `score.active` true (terrain contributes); the Scoring-Engine UI shows the band/est%/reason; one viewport preserved, nothing scrolls.
- [ ] Combinations hand-checked: static × each of the 4 types, and dynamic × each of the 4 types.
- [ ] Existing `id`s and `.parentElement` binding highlight intact; `wc -c prod/aerodeck-planner.html` recorded; CHANGELOG.md entry appended.
