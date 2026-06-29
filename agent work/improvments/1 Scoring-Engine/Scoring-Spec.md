# AeroDeck Scoring Spec — Two-Layer Verdict Model

**Status:** system-of-record for scoring semantics.
**Supersedes:** `AeroDeck-Flight-Planner-Architecture.md` §4 and §8 on the verdict.

This document defines the two-layer verdict model and the scoring framework contract. It is the authoritative reference; Architecture §4 ("verdict intentionally limited to physical executability") and §8 ("no new science") are updated by this spec for the verdict surface only.

---

## 1. Two-Layer Model

The verdict consists of two independent layers that are **never merged**:

1. **Binary physics gate** (existing): hard pass/fail for physical executability.
   - Driven by `frontOverlapFail` and `blurFail`.
   - Produces `inSpec: boolean`.
   - A plan is either physically executable or OUT OF SPEC.
   - This layer is unchanged by scoring.

2. **Robustness score** (additive): graded estimate of data-collection success likelihood.
   - Separate from the gate.
   - A plan can be gate-PASS but low-score.
   - A plan can be gate-FAIL (OUT OF SPEC) with no score shown.

**Contract:** score logic must not read or mutate gate booleans (`frontOverlapFail`, `blurFail`, `inSpec`). The gate result and score object are returned side-by-side from `compute()`.

---

## 2. Estimated-Success Semantics

The score is a **heuristic estimate** of the likelihood of successful data collection (reconstruction-quality data), presented as:

- A qualitative **band** (glanceable label).
- An **estimated success likelihood** shown as `Est. ≈X%`.

**Non-guarantee stance (mandatory):**
- The value is an estimate, not a prediction or certification.
- Language is always cautious: "estimate", "likelihood", "robustness".
- The UI carries a persistent non-guarantee microcopy whenever the score is shown.
- No absolute or certainty language is permitted.

**Cap below 100% (structural):**
- Even a clean plan asymptotes to `MAX_EST_PCT` (95%).
- `MAX_EST_PCT < 100` is enforced in code; clean input never yields ≥100%.
- This prevents any reading of certainty.

---

## 3. Bands and Thresholds

Bands are derived from `estPct` using documented constants. Thresholds are lower-inclusive.

```js
const MAX_EST_PCT = 95;
const MIN_EST_PCT = 20;

const SCORE_BANDS = [
  { band: 'Strong',   min: 80 },
  { band: 'Adequate', min: 60 },
  { band: 'Weak',     min: 0  }
];
```

- `estPct` is computed as `Math.max(MIN_EST_PCT, MAX_EST_PCT - totalPenalty)`.
- Band lookup: first entry where `estPct >= min`.
- Bands are named constants; changing cut points requires updating the definition here and the implementation.

---

## 4. Gate-Fail Behavior

When the gate fails (`inSpec === false`):

- The robustness estimate is **suppressed** (not shown as a number or band).
- The binary state (OUT OF SPEC) remains the visual headline.
- An unexecutable plan does not receive a success percentage.

Rationale: a plan that cannot be physically executed has no meaningful "likelihood of successful data collection" to report.

---

## 5. Contributor Interface

Each scoring contributor produces a contribution object:

```js
{
  id: string,        // stable identifier (e.g., 'overlap-floor', 'terrain')
  label: string,     // human name for the contributor
  penalty: number,   // points to deduct from MAX_EST_PCT
  reason: string     // short, scannable explanation referencing one named value
}
```

**Decomposition rule:** every penalty traces to exactly one named, already-computed value. The `reason` string identifies that value and the deduction. No learned weights, no opaque formulas.

**Weights:** contributor influence is expressed via `SCORE_WEIGHTS`, a single named const with one-line rationale comments per weight. Tunable without hunting through logic.

---

## 6. SCORE_WEIGHTS Convention

All numeric influence constants live in one place:

```js
const SCORE_WEIGHTS = {
  // === SCORE CONTRIBUTORS — registered by improvments/* folders ===
  // CONTRIBUTOR[overlap-floor]: Overlap-Quality-Floor folder
  // CONTRIBUTOR[terrain]:       Terrain-Weighting folder
  // CONTRIBUTOR[orientation]:   Sensor-Orientation folder (if it scores)
};
```

Each registered contributor adds documented weight entries with a one-line rationale.

---

## 7. Aggregation (`scorePlan`)

Pure function:

```js
function scorePlan(contributions) {
  // returns { estPct, band, reasons, active }
}
```

- `estPct`: `MAX_EST_PCT` minus sum of penalties, floored at `MIN_EST_PCT`.
- `band`: derived from `estPct` via `SCORE_BANDS`.
- `reasons`: array of each contribution's `reason` (order preserved).
- `active`: `contributions.length > 0`.

On empty input (dormant state): `{ estPct: MAX_EST_PCT, band: top band, reasons: [], active: false }`.

---

## 8. Dormant Framework

This spec ships with a **dormant** framework:

- `scorePlan([])` is wired into `compute()`.
- With zero contributors: `active: false`, top band, capped est%.
- Explicit pickup comments mark where future contributors register.
- UI rendering of the score layer is suppressed while `active === false`.

Dormancy prevents a misleading static percentage from appearing before real contributors exist.

---

## 9. Integration Contract

`compute()` returns the existing gate result plus a score object:

```js
{
  // ... existing gate fields (frontOverlapFail, blurFail, inSpec, ...)
  score: { estPct, band, reasons, active }
}
```

- Gate fields are unchanged in name, meaning, and derivation.
- `score` is always present (may be dormant).
- `repaint()` (Agent 2) consumes `compute().score` for display.

---

## 10. Non-Goals / Out of Scope (this spec)

- No new physics or flight-dynamics modeling.
- No terrain drape or DEM math.
- Individual contributor domain logic lives in their own folders.
- UI/DOM for the score layer is Agent 2.

---

## 11. Acceptance for the Framework (Agent 1)

- `Scoring-Spec.md` defines the two-layer model, estimated-success + non-guarantee semantics, sub-100 cap, bands/thresholds, gate-fail suppression, contributor interface, and decomposition.
- `scorePlan` is pure and returns correct values on synthetic contributions.
- Clean/empty input never yields ≥100%.
- `SCORE_WEIGHTS` present with rationale comments and explicit contributor pickup markers.
- `compute()` returns a `score` object; dormant `active:false` when no contributors.
- Gate booleans unchanged; no UI/DOM changes in this pass.

---

## 12. Score Band Color Mapping (Rendering Contract)

The secondary score band label color is graded by band value. This mapping is part of the official rendering contract and is the system-of-record for any work that styles or extends the verdict band (including Ceiling-Violation-Visibility).

| Band     | Token                  | Rationale |
|----------|------------------------|-----------|
| Strong   | `--color-in-spec`      | Good robustness; reuses the established "in spec" green. |
| Adequate | `--color-binding`      | Marginal; the binding/attention orange signals "pay attention". |
| Weak     | `--color-out-of-spec`  | Poor robustness; the failure red makes the quality problem visible at a glance. |

**Rules:**
- Implementation uses only the named tokens above. No raw hex literals for band color.
- The color is driven from the live `score.band` string (e.g. via `[data-band="Weak"]` attribute selector or equivalent class on the band element).
- The gate layer is always the visual headline: larger typography on `#verdict-state`, its `in-spec`/`out-of-spec` classes, and the `#verdict:has(...)` background/border. The band is rendered in a subordinate flex line below the gate with `font-weight: var(--fw-semibold)` and smaller clamp sizing.
- When the score layer is suppressed (dormant or gate-fail), no band color is shown.
- This prevents the false-reassurance case where a Weak plan would otherwise render its label in reassuring green.

This section supersedes any prior hard-coded token on the band element.

---

## 13. Acceptance for Band Color Grading (Agent 3)

- Band color follows the table in §12 using only existing `--color-*` tokens.
- Strong = in-spec green, Adequate = binding orange, Weak = out-of-spec red.
- No raw hex introduced.
- Gate remains headline; band is secondary.
- `Scoring-Spec.md` now contains the mapping as the rendering contract.
- Existing ids, `.parentElement` binding chain, one-viewport layout, and token discipline preserved.
- `wc -c` recorded and CHANGELOG entry added as Scoring Engine / Agent 3.
