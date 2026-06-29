# Agent 1: Overlap-Floor Contributor

**Depends on:** Scoring-Engine / Agent 1 (the scoring framework + `Scoring-Spec.md` + contributor interface must exist)
**Blocks:** nothing (single pass for this folder)

---

## Goal
Register **front** and **side** overlap quality as score contributors in the existing framework, bringing side overlap into `compute()` for the first time, so low overlap lowers the estimated success with a specific reason — without touching the physics gate.

## Deliverables
1. **Threshold constants** — a named, documented `const` (e.g. `OVERLAP_MIN`) with a per-axis **recommended** value and a **low** value below which quality degrades. Defaults (standard mapping conventions — tunable, *not* derived science; comment as such):
   - Front: recommended ≈ 75%, low ≈ 60%.
   - Side: recommended ≈ 65%, low ≈ 50%.
2. **Weights** — register `overlap-front` and `overlap-side` entries in `SCORE_WEIGHTS` (per the Scoring-Engine convention) with one-line rationale each.
3. **Graded penalty per axis** — 0 at/above `recommended`, ramping to the axis's max weight as overlap falls to/below `low`. Pure function(s); no aggregation logic here (the framework's `scorePlan` owns that).
4. **Side overlap into `compute()`** — produce a front and a side contribution `{ id, label, penalty, reason }` and include them in the contributions array passed to `scorePlan()`. Side overlap currently only displays; it now participates in scoring.
5. **Decompositional reasons** — minimal strings that name the value, e.g. *"side overlap 50% below recommended 65%"*. No penalty → no reason emitted for that axis.

## Reference Files
- [`../Scoring-Engine/Scoring-Spec.md`](../Scoring-Engine/Scoring-Spec.md) — contributor interface, decomposition format, weight convention.
- [`../Scoring-Engine/Agent0-Overview.md`](../Scoring-Engine/Agent0-Overview.md) — locked principles.
- [`Agent0-Overview.md`](Agent0-Overview.md) — score-only decision.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §1.
- `prod/aerodeck-planner.html` — target.

## Key traps
- **Score-only — never the gate.** Do not touch `frontOverlapFail`, `blurFail`, or `inSpec`. Low overlap must never flip the physics verdict.
- **Front overlap is now bounded both ways, in different layers.** Its *upper* capture-rate ceiling is the existing **gate** (unchanged); the new *lower* quality floor is a **score** contribution. Keep them distinct — do not merge the floor into the ceiling logic.
- **Side overlap enters scoring only.** It remains independent of blur/capture physics (Architecture §3) — no gate, no footprint/ceiling interaction.
- **No slider clamping** (Finding #4 philosophy) — the operator may set any value; we score it, we don't block it.
- **Use the framework.** Emit contributions through the contributor interface; don't re-implement aggregation, banding, or the est% cap.
- Thresholds/weights are **named constants with rationale**, conventions not science.
- Engine work: `<script>` changes fine; preserve existing `id`s and the binding-highlight `.parentElement` chain; single-file/offline/size discipline and brand light theme stand.

## Acceptance Criteria
- [ ] `OVERLAP_MIN` thresholds + `overlap-front` / `overlap-side` weights present, as named constants with rationale comments.
- [ ] Lowering front or side overlap below its recommended value lowers `estPct` and adds a specific decompositional reason; at/above recommended, that axis adds no penalty and no reason.
- [ ] Side overlap participates in `compute()` (no longer display-only).
- [ ] Gate booleans unchanged — low overlap alone never produces OUT OF SPEC; verified.
- [ ] With these contributions present, `score.active` is true, so the Scoring-Engine UI now shows the band / est% / reasons.
- [ ] Penalty behavior hand-checked (node/VM or in-browser) for: above recommended (no penalty), between recommended and low (partial), at/below low (max weight).
- [ ] Existing `id`s and `.parentElement` binding highlight intact; `wc -c prod/aerodeck-planner.html` recorded; CHANGELOG.md entry appended.
