# Agent 3: Visual Design + Polish

**Depends on:** Agent 2 (functional, correct, unstyled file)
**Blocks:** nothing (final phase)

---

## Goal
Apply the full visual treatment to the working file — styling only, zero changes to computation or reactivity.

## Deliverables
1. In the same `prod/aerodeck-planner.html`, add the inlined `<style>` per architecture §6:
   - **One viewport, no scroll.** `dvh` units to handle the mobile address bar; fixed layout so nothing scrolls during input.
   - **Glanceable typography** via `clamp()` for the readouts.
   - **High contrast, large touch targets**; readability decided at design time.
   - **Verdict band color-coded** for in-spec / out-of-spec, large, the first thing the eye lands on.
   - Front overlap visually adjacent to speed (shared failure mode); side overlap visually standalone.
   - Binding constraint visually highlighted in the readout.

## Reference Files
- [`../../AeroDeck-Flight-Planner-Architecture.md`](../../AeroDeck-Flight-Planner-Architecture.md) §6 (layout + UX constraints).
- Any separately supplied styling spec (architecture §6 notes styling "to be supplied separately and applied within these structural rules") — apply within the existing structure if provided; otherwise use the §6 rules directly.

## Key traps
- **Do not touch logic.** No edits to computation functions or the `oninput` wiring. If achieving the layout seems to require a logic change, stop and flag it — it means Phase 2 was incomplete.
- **Single file, no assets.** No external fonts, images, CSS, or CDN links. All CSS inlined in `<style>`.
- **Fixed, non-scrolling layout.** The whole interface must live in one viewport on Chrome/Android.
- Keep CSS lean (CODE-DISCIPLINE.md) — every byte ships.

## Acceptance Criteria
- [ ] Entire interface fits one viewport on mobile; nothing scrolls during input (`dvh`, fixed layout).
- [ ] Verdict band is prominent and color-coded for in/out-of-spec.
- [ ] Readout typography uses `clamp()`; high contrast, large hit areas throughout.
- [ ] Front overlap sits adjacent to speed; side overlap stands alone; binding constraint highlighted.
- [ ] No computation/reactivity changes vs. Agent 2 (diff is style-only).
- [ ] No external assets or libraries; all CSS inlined.
- [ ] `wc -c prod/aerodeck-planner.html` recorded in the CHANGELOG entry.
- [ ] CHANGELOG.md entry appended.
