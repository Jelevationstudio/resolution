# Agent 0: Scoring Engine — Overview

> **Finding #5** in [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md). **This folder is the foundation — build it first.** Findings #1 (Overlap Quality Floor) and #2 (Terrain Weighting) register contributors into the framework defined here; #4 (Ceiling Violation Visibility) shares the verdict rendering.

## Read first (every session)
- [`../../CLAUDE.md`](../../CLAUDE.md) — behavioral guidelines.
- [`../../CODE-DISCIPLINE.md`](../../CODE-DISCIPLINE.md) — single-file size/complexity rules and what ships.
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) — current system-of-record. **Note:** this work intentionally extends §4/§8; see "Spec addendum" below.
- [`../../../AeroDeck-Flight-Planner-Executive-Summary.md`](../../../AeroDeck-Flight-Planner-Executive-Summary.md) — the product promise this closes the gap toward.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) — the findings that seeded this work.

## Context

The planner's verdict is currently a binary physics gate (blur + capture-rate ceiling). The product promises more than "the camera can physically take the pictures" — it implies a *resolved, trustworthy plan*. We are adding a **graded robustness score** on top of the binary gate so the tool can say not just "executable" but "executable and how robust."

This is a deliberate evolution past the architecture's "closed-form physics only, no new science" framing. It is allowed **only** if built to the principles below — otherwise it becomes the vendor lookup-table-of-vibes we set out to replace.

## Locked principles (govern this folder and every contributor folder)

1. **Two layers, never merged.** The binary physics gate stays as the hard pass/fail (can the camera execute this). The robustness score is a separate additive layer. A plan can be gate-FAIL (OUT OF SPEC), or gate-PASS but low-score. Collapsing them into one number is forbidden — it destroys the hard constraint.
2. **Fully decompositional — no black box.** Every point deducted traces to exactly one named, already-computed value. The UI shows *"low score — here's why {the specific value}"*. No learned weights, no opaque formula, no invented physics. An expert must be able to audit and disagree with each deduction.
3. **Weights are named, documented constants in one place.** A single `SCORE_WEIGHTS`-style object literal with a one-line rationale per weight. Tunable without hunting through `compute()`.
4. **Score semantics = margin / robustness, never a guarantee.** UI language is field-guide framing ("closer range," "robustness"), never "this will work." We help alleged experts narrow the range, not certify outcomes.
5. **Glance-first, detail-on-demand.** The score reads at a glance; the per-contributor "why" is minimal and scannable, not a wall of text (user directive).

## Scope

**In scope:** the scoring *framework* — the contributor data structure, the aggregation into a score, the decomposition output ("why" list), the `SCORE_WEIGHTS` constants location, and the two-layer verdict rendering (gate state + score + reasons). Provide a clear contributor interface that #1/#2/#3 plug into.

**Not in scope:** the individual contributors' domain logic (their own folders); any new physics; flight-dynamics or trigger modeling (declared out of scope, see Sensor-Orientation folder).

## Spec addendum (required deliverable)

This work contradicts the current Architecture §4 ("verdict intentionally limited to physical executability") and §8 ("no new science"). One agent in this folder **must** author a short architecture addendum / scoring spec that becomes the system-of-record for the two-layer model — otherwise a future cold agent reads §8 and "corrects" the scoring back out.

## Engine note (different from the styling passes)

This is **engine work** — it changes the `<script>` block and will add DOM for the score display. The Brand-Restyle "script byte-identical / no new DOM" contract does **not** apply here. But still: preserve existing `id`s and the binding-highlight `.parentElement` chain; respect single-file/offline/size discipline; keep the existing brand light-theme tokens.

## Decisions settled (do not relitigate)
- **Representation:** a qualitative **band** + an **"estimated" success likelihood** (≈%). Worse inputs lower the estimated chance of successful data collection. **It can never read as a guarantee** (principle 4): the estimate is capped below 100% and always carries non-guarantee framing.
- **Dormant framework.** This folder ships the framework only — **no real contributors** (overlap/terrain/orientation land in their own folders). Leave explicit pickup comments; keep the score UI hidden until ≥1 contributor exists so the interim build shows no misleading number.

## Execution flow

```
Agent 1: Scoring framework (pure JS, dormant) + scoring spec addendum
Agent 2: Two-layer verdict rendering (gate + band + est%, cautious framing)
```
Sequential; Agent 2 depends on Agent 1. No cycles.

Build order across folders: **Scoring-Engine → Overlap-Quality-Floor + Terrain-Weighting → Ceiling-Violation-Visibility**. Sensor-Orientation is semi-independent (see its overview).

## Logging
Append one entry per completed pass to [`../../CHANGELOG.md`](../../CHANGELOG.md) using the README template. Newest first.

## Folder naming (read for cross-folder links)
The improvements folders are now build-order prefixed:
`1 Scoring-Engine`, `2 Overlap-Quality-Floor`, `3 Terrain-Weighting`, `4 Ceiling-Violation-Visibility`, `5 Sensor-Orientation`.
Any brief that links `../Scoring-Engine/...` means `../1 Scoring-Engine/...` (and likewise for the others) — resolve cross-folder links to the prefixed names.
