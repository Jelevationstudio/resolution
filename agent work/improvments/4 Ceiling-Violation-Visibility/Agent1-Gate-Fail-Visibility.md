# Agent 1: Gate-Fail Visibility

**Depends on:** Scoring-Engine / Agent 2 (the two-layer verdict rendering this extends)
**Blocks:** nothing (single pass; final improvements folder)

---

## Goal
Make an over-ceiling front-overlap violation impossible to miss at a glance — strengthen the gate-fail presentation, naming the specific offending value and by how much — **without clamping the slider** and **without forking a second verdict**.

## Deliverables
1. **Mark the offender, not just the row.** When front overlap exceeds its capture-rate ceiling, beyond the existing `.binding` outline make the offending **value itself** read as out-of-spec (front-overlap value + the ceiling readout in `--color-out-of-spec` / `--color-binding`), so the eye lands on the exact number at fault.
2. **Explicit gap message.** A compact line naming the specific gap, e.g. *"Front overlap 85% exceeds ceiling 78% at this speed — unachievable."* Reuse/extend the existing `#binding-note`, don't add a parallel element.
3. **Brief fix hint (keep to one short line).** Surface the two remedies — *"lower front overlap or raise altitude"* — echoing the coupling. Glance-first; do not bloat into a paragraph.
4. **Consistency with blur.** Give the blur gate-fail the same strength of treatment (name the offender: speed exceeds blur ceiling) so there is no loud-capture / quiet-blur mismatch. Keep both within the existing verdict + binding structure.
5. **No clamping.** The slider stays free (`min/max/step` unchanged); the operator may keep a value above the ceiling. We make it unmissable, we never block or auto-correct it.

## Reference Files
- [`../Scoring-Engine/Scoring-Spec.md`](../Scoring-Engine/Scoring-Spec.md) — two-layer model, gate-fail behavior (gate is the headline).
- [`Agent0-Overview.md`](Agent0-Overview.md) — the no-clamp decision + extend-don't-fork coordination note.
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §4.
- `prod/aerodeck-planner.html` — target. Build on the existing `repaint()`, `setBindingHighlight`, `#binding-note`, and verdict band.

## Key traps
- **Visibility only — never clamp, block, or auto-correct the slider** (Finding #4 decision). This is the central constraint.
- **Extend, don't fork.** Build on the existing verdict band + `#binding-note` + `setBindingHighlight`. Do **not** add a second/competing verdict banner. Align with the Scoring-Engine two-layer verdict, where the **gate is the headline** and the score is secondary — this folder strengthens the gate layer and does not touch the score.
- **One viewport, no scroll.** The stronger treatment and messages must fit; keep messages compact, not multi-line.
- **Glance-first, name the value.** The operator should see *which* number is at fault and by how much, instantly.
- **No motion/animation** that fights `prefers-reduced-motion` or distracts — use static emphasis (color/weight).
- Brand tokens only (no raw hex); preserve existing `id`s and the binding-highlight `.parentElement` chain; single-file/offline/size discipline stand.

## Acceptance Criteria
- [ ] When front overlap exceeds its ceiling: verdict is unmistakably OUT OF SPEC, the offending front-overlap value (and the ceiling) are visually marked as the offender, and a compact message names the gap.
- [ ] A one-line fix hint ("lower front overlap or raise altitude") is present and concise.
- [ ] The slider remains free — value is not clamped, blocked, or auto-corrected.
- [ ] Blur gate-fail receives consistent-strength treatment (offender named); no loud/quiet mismatch between the two gate failures.
- [ ] Built on the existing verdict/binding structure — no competing banner; gate remains the headline alongside the Scoring-Engine score layer.
- [ ] One viewport preserved, nothing scrolls; brand tokens only; existing `id`s and `.parentElement` binding highlight intact.
- [ ] `wc -c prod/aerodeck-planner.html` recorded; CHANGELOG.md entry appended.
