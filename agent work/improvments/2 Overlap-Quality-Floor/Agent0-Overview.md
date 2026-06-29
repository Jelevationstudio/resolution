# Agent 0: Overlap Quality Floor — Overview

> **Finding #1** in [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md). **Depends on the Scoring-Engine folder** (registers a contributor into its framework). Build after Scoring-Engine.

## Read first (every session)
- [`../../CLAUDE.md`](../../CLAUDE.md) — behavioral guidelines.
- [`../../CODE-DISCIPLINE.md`](../../CODE-DISCIPLINE.md) — single-file rules.
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) §3–§5 — variable model and coupling.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §1 — the gap.
- [`../Scoring-Engine/Agent0-Overview.md`](../Scoring-Engine/Agent0-Overview.md) — the framework + locked principles this contributor obeys. Plus the scoring spec addendum once it exists.

## Context

Today a plan with 50% / 50% overlap can show a clean green "IN SPEC" — there is only an *upper* ceiling on front overlap and **no floor**. Side overlap never enters `compute()` at all. This directly betrays the product mission ("prevent a plan that looks valid and collects poor data"); the user called it cheap and unfinished. This folder adds a **minimum-overlap quality signal**.

## Scope

**In scope:**
- Recommended-minimum thresholds for **front** and **side** overlap (named, documented constants).
- A **score contributor** (per the Scoring-Engine framework) that lowers the robustness score as overlap drops toward/below those minimums, with a decompositional reason (e.g. *"score lowered: side overlap 50% below recommended 60%"*).
- Make **side overlap participate** in the logic for the first time (it currently only displays).

**Not in scope:**
- Clamping or blocking the sliders (leave inputs free — see Ceiling-Violation-Visibility / Finding #4 philosophy).
- New physics. Thresholds are documented planning conventions, not derived science.

## Decision settled (do not relitigate)
- **Score-only, never the gate.** The binary physics gate stays *physics-only* (blur + capture). Overlap quality is a **score** contributor — a low-overlap plan is "executable but weak," not "impossible." Even extreme low overlap only lowers the estimated success, it never flips the physics gate. This is the only reading consistent with the locked two-layer model.

## Engine note
Changes `<script>` (new constants, a contributor, side-overlap into `compute()`). Preserve existing `id`s and the binding-highlight `.parentElement` chain. Single-file/offline/size discipline and the brand light theme stand.

## Execution flow
```
Agent 1: Overlap-floor contributor — thresholds, front + side penalties, side overlap into compute()
```
Single pass (pure logic; the Scoring-Engine UI renders the result generically). Depends on the Scoring-Engine framework (Agent 1) shipped.

Logging: append to [`../../CHANGELOG.md`](../../CHANGELOG.md), newest first.

## Folder naming (read for cross-folder links)
The improvements folders are now build-order prefixed:
`1 Scoring-Engine`, `2 Overlap-Quality-Floor`, `3 Terrain-Weighting`, `4 Ceiling-Violation-Visibility`, `5 Sensor-Orientation`.
Any brief that links `../Scoring-Engine/...` means `../1 Scoring-Engine/...` (and likewise for the others) — resolve cross-folder links to the prefixed names.
