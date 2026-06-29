# Agent 1: Scoring Framework + Spec

**Depends on:** nothing (first pass of the scoring work)
**Blocks:** Agent 2; all contributor folders (Overlap-Quality-Floor, Terrain-Weighting, Sensor-Orientation)

---

## Goal
Author the scoring spec (system-of-record) and implement the **dormant** pure-JS scoring framework — the contributor interface, weight constants, aggregation, band + estimated-success output — with no real contributors and no UI.

## Deliverables

### 1. `Scoring-Spec.md` (in this folder) — the system-of-record
Must define, and state that it **supersedes Architecture §4/§8 on the verdict**:
- **Two-layer model.** Binary physics gate (blur + capture ceiling) is the hard pass/fail; the robustness score is a separate additive layer. They are never merged.
- **Estimated-success semantics.** The score is a heuristic **estimate** of data-collection success likelihood, presented as a band + ≈%. It is **not a guarantee** — define the non-guarantee stance explicitly.
- **Cap below 100%.** A clean plan asymptotes to `MAX_EST_PCT` (e.g. 95%), never 100% — structurally prevents implying certainty.
- **Bands + thresholds.** Band labels and the est% cut points (documented constants).
- **Gate-fail behavior.** When the gate fails (physically OUT OF SPEC), the estimate is **suppressed** (not shown as a number) — an unexecutable plan does not get a success %.
- **Contributor interface.** The shape every contributor produces (see §2), and the `SCORE_WEIGHTS` rationale convention (one line per weight).
- **Decomposition.** Every deduction maps to one named, already-computed value with a short reason string. No black box.

### 2. The framework in `prod/aerodeck-planner.html` `<script>` (pure, dormant)
- `SCORE_WEIGHTS` — a named `const` object for contributor weights, **with rationale comments**. Ships effectively empty, with explicit pickup markers:
  ```js
  // === SCORE CONTRIBUTORS — registered by improvments/* folders ===
  // CONTRIBUTOR[overlap-floor]: Overlap-Quality-Floor folder
  // CONTRIBUTOR[terrain]:       Terrain-Weighting folder
  // CONTRIBUTOR[orientation]:   Sensor-Orientation folder (if it scores)
  ```
- A pure aggregator, e.g. `scorePlan(contributions)` taking an array of `{ id, label, penalty, reason }` and returning `{ estPct, band, reasons, active }`:
  - `estPct` = `MAX_EST_PCT` minus summed penalties, floored at a sane minimum.
  - `band` = derived from `estPct` via the thresholds.
  - `reasons` = the decomposition list (each contribution's `reason`).
  - `active` = whether any contributions exist (drives "hidden until a contributor exists").
- Wire into `compute()`: return the existing gate result **plus** a `score` object from `scorePlan([])`. With zero contributors today, `active:false`, top band, capped est. Comment this dormant state clearly.

### 3. No UI / DOM changes
Agent 2 renders. This pass is logic + the spec doc only.

## Reference Files
- [`Agent0-Overview.md`](Agent0-Overview.md) — locked principles + settled decisions.
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) §4, §8 — what the spec supersedes.
- `prod/aerodeck-planner.html` — target.

## Key traps
- **Two layers never merged.** Score logic must not read or mutate the gate booleans (`frontOverlapFail`, `blurFail`, `inSpec`). Keep `scorePlan` pure and independent.
- **No real contributors, no new physics.** Dormant by decision — leave the pickup comments; don't invent overlap/terrain logic here.
- **Cap + non-guarantee are structural.** Enforce `MAX_EST_PCT < 100` in code; the spec must state the non-guarantee stance.
- **Engine work:** `<script>` changes are fine; preserve existing `id`s and the binding-highlight `.parentElement` chain; single-file/offline/size discipline and brand light theme stand.
- Weights/thresholds are **named constants with rationale**, not magic numbers in `compute()`.

## Acceptance Criteria
- [ ] `Scoring-Spec.md` exists and defines the two-layer model, estimated-success + non-guarantee semantics, the sub-100 cap, bands/thresholds, gate-fail suppression, contributor interface, and decomposition; states it supersedes Architecture §4/§8 on the verdict.
- [ ] `scorePlan(contributions)` is pure and, on **synthetic** contributions, returns correct `estPct` (capped), `band`, `reasons`, and `active`; verified (node/VM).
- [ ] Clean/empty input never yields ≥100% (cap enforced).
- [ ] `SCORE_WEIGHTS` present with rationale comments + explicit contributor pickup markers for overlap/terrain/orientation.
- [ ] `compute()` returns a `score` object alongside the unchanged gate result; dormant `active:false` state when no contributors.
- [ ] Gate booleans unchanged; no UI/DOM change; existing `id`s and `.parentElement` chain intact.
- [ ] `wc -c prod/aerodeck-planner.html` recorded; CHANGELOG.md entry appended.
