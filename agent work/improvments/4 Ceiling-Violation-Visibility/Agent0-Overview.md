# Agent 0: Ceiling Violation Visibility — Overview

> **Finding #4** in [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md). **Shares the verdict rendering with the Scoring-Engine folder** — build after it (or coordinate closely) so both don't rewrite the verdict band.

## Read first (every session)
- [`../../CLAUDE.md`](../../CLAUDE.md) — behavioral guidelines.
- [`../../CODE-DISCIPLINE.md`](../../CODE-DISCIPLINE.md) — single-file rules.
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) §4–§5 — the ceiling and coupling.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §4 — the gap.
- [`../Scoring-Engine/Agent0-Overview.md`](../Scoring-Engine/Agent0-Overview.md) — two-layer verdict rendering this must align with.

## Context

The front-overlap slider permits values above the computed capture-rate ceiling. The violation is highlighted and flips the verdict to OUT OF SPEC, but the unachievable value is **retained**, not blocked.

**Decided:** do **not** clamp or block the slider. The operator (an alleged expert) may explore above the ceiling; it is ultimately their decision to act on the abstract info we surface. Hard-clamping would hide the coupling that is the product's teaching moment.

## Scope

**In scope:** make an over-ceiling violation **impossible to miss** at a glance — strengthen the OUT OF SPEC / binding presentation when the front-overlap target exceeds its ceiling (clear, glanceable, unambiguous which value is the offender). This is a **gate-layer presentation** improvement.

**Not in scope:**
- Clamping, blocking, or auto-correcting the slider input.
- The robustness score (that's the Scoring-Engine + contributor folders). This folder is strictly about the binary gate's legibility.

## Coordination note
The Scoring-Engine folder rewrites the verdict into two layers (gate + score). This folder emphasizes the *gate* violation. To avoid both touching the verdict band independently, build this **after** the Scoring-Engine verdict rendering exists and extend it — do not fork a competing verdict treatment.

## Engine note
May change `<script>` (presentation logic) and CSS. Preserve existing `id`s / `.parentElement` chain. Brand light-theme tokens, single-file/offline/size discipline stand.

## Execution flow
```
Agent 1: Strengthen gate-fail visibility — name the offending value + gap, no clamping; extend the existing verdict/binding treatment
```
Single pass. Build **after** the Scoring-Engine verdict rendering exists, and extend it — do not fork a competing verdict treatment.

Logging: append to [`../../CHANGELOG.md`](../../CHANGELOG.md), newest first.

## Folder naming (read for cross-folder links)
The improvements folders are now build-order prefixed:
`1 Scoring-Engine`, `2 Overlap-Quality-Floor`, `3 Terrain-Weighting`, `4 Ceiling-Violation-Visibility`, `5 Sensor-Orientation`.
Any brief that links `../Scoring-Engine/...` means `../1 Scoring-Engine/...` (and likewise for the others) — resolve cross-folder links to the prefixed names.
