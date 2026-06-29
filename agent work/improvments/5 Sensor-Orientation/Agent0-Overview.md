# Agent 0: Sensor Orientation — Overview

> **Finding #3** in [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md). The actionable residue of Finding #3. **Semi-independent** — it can be built without the Scoring-Engine, but touches the same `compute()` / `footprint()` area, so coordinate if scoring lands first.

## Read first (every session)
- [`../../CLAUDE.md`](../../CLAUDE.md) — behavioral guidelines.
- [`../../CODE-DISCIPLINE.md`](../../CODE-DISCIPLINE.md) — single-file rules.
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) §4 — footprint / GSD math.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §3 — the assumptions.

## Context

Finding #3 lists several hidden assumptions. **Most are deliberately settled — recorded here so they are not reopened:**

| Assumption | Decision |
|---|---|
| Blur budget (`MAX_BLUR_PX = 1`) | **Keep as-is.** No change. |
| Capture rate ("sustained") | **Keep as-is** — placeholder, real per-sensor values come later. |
| Flight dynamics (accel / turns / turnarounds) | **Out of scope.** Assume static, straight-line flight; turns are not needed in this tool. |
| Trigger behavior (real-world interval constraints) | **Out of scope.** This is a field guide, not an exhaustive modeling simulator. |
| **Orientation (landscape / portrait)** | **Actionable — this folder.** |

## Scope

**In scope:** add a **landscape / portrait orientation toggle** that swaps which sensor axis is treated as along-track in `footprint()`. Today `footprint()` always assigns `heightPx` to along-track; orientation directly changes the along-track footprint and therefore the **front-overlap ceiling**. This is a correctness/completeness feature.

**Not in scope:** the four settled items above; any score penalty for orientation itself (orientation just makes the ceiling *correct* — it isn't a quality risk). If a scoring touchpoint emerges, defer to the Scoring-Engine framework rather than inventing one here.

## Engine note
Changes `<script>` (`footprint()` axis selection + a new toggle handler) and **adds one DOM control** (the orientation toggle, modeled on the existing terrain toggle) — permitted, this is engine work. Preserve existing `id`s and the binding-highlight `.parentElement` chain. Style the new control with existing brand tokens. Single-file/offline/size discipline stands.

## Execution flow
```
Agent 1: Orientation toggle — landscape/portrait selects the along-track axis in footprint(); recomputes the front-overlap ceiling
```
Single pass. Semi-independent — buildable without the Scoring-Engine; it only corrects footprint/ceiling math and adds no score contribution.

Logging: append to [`../../CHANGELOG.md`](../../CHANGELOG.md), newest first.

## Folder naming (read for cross-folder links)
The improvements folders are now build-order prefixed:
`1 Scoring-Engine`, `2 Overlap-Quality-Floor`, `3 Terrain-Weighting`, `4 Ceiling-Violation-Visibility`, `5 Sensor-Orientation`.
Any brief that links `../Scoring-Engine/...` means `../1 Scoring-Engine/...` (and likewise for the others) — resolve cross-folder links to the prefixed names.
