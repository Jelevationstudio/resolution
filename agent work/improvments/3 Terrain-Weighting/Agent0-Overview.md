# Agent 0: Terrain Weighting — Overview

> **Finding #2** in [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md). **Depends on the Scoring-Engine folder** (registers a contributor). Build after Scoring-Engine.

## Read first (every session)
- [`../../CLAUDE.md`](../../CLAUDE.md) — behavioral guidelines.
- [`../../CODE-DISCIPLINE.md`](../../CODE-DISCIPLINE.md) — single-file rules.
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) §3, §8 — terrain is a flag, does no terrain math.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §2 — the gap.
- [`../Scoring-Engine/Agent0-Overview.md`](../Scoring-Engine/Agent0-Overview.md) — framework + locked principles. Plus the scoring spec addendum once it exists.

## Context

The Dynamic AGL checkbox is currently decorative — `terrainDynamic` is read but never enters `compute()`; it only swaps a line of text. Per the user: static-vs-dynamic AGL *does* affect the real-world result (however negligible, the relief risk is a real possibility), so it **must carry weight** in the robustness score.

## Scope

**In scope:**
- A **categorical terrain-type selector** — a new `<select>` modeled on the existing light selector, with **4 options from flat to mountainous**, each resolving to its own documented relief weight. This is the architecture's endorsed pattern: a categorical box resolving to a flag/clamp (cf. the light selector), *not* terrain math.
- An **interaction model with the existing Dynamic-AGL toggle:** **static AGL applies the selected terrain-type weight** (flat-ground assumption violated in proportion to relief); **dynamic AGL / terrain-follow suppresses it to ~0** (constant AGL mitigates relief). That combination is the score contribution.
- Weights are **arbitrary-but-documented** named constants, tunable, aligned to the Scoring-Engine penalty scale, and **escalate non-linearly with relief severity** — mountainous is heavily/dominantly weighted (static AGL over mountains can mean hundreds of feet of uncompensated vertical change), not a linear step above hilly.
- A **decompositional reason** naming the active values, e.g. *"static AGL over mountainous terrain — flat-ground assumption likely violated."*

**Not in scope:**
- **No terrain / DEM computation, no relief parameter, no GSD-range math.** Categorical selection is a declaration of intent (Architecture §8), like the light selector — it reflects *risk of an unmodeled factor*, it does not model the factor.

## Engine note
Changes `<script>` and **adds one DOM control** (the terrain-type `<select>`, modeled on the light selector) plus its state/wiring — permitted engine work. Keep the existing `#terrain-dynamic` toggle. Preserve existing `id`s and the `.parentElement` chain; style the new control with brand tokens; keep one viewport. Single-file/offline/size discipline stand.

## Execution flow
```
Agent 1: Terrain contributor — terrain-type selector (4 options) + AGL-mode interaction + score contribution
```
Single pass (the Scoring-Engine UI renders the score result generically). Depends on the Scoring-Engine framework (Agent 1) shipped.

Logging: append to [`../../CHANGELOG.md`](../../CHANGELOG.md), newest first.
