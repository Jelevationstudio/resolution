# Agent 3: Visual Design + Polish

**Depends on:** Agent 2 (functional, correct, unstyled file)
**Blocks:** nothing (final phase)

---

## Goal
Apply the full visual treatment to the working file — styling only, zero changes to computation or reactivity.

## Scope contract (read first — this is the whole point of this phase)

This pass is **CSS-only**. The boundary is mechanical, not a matter of judgment:

**You MAY change:**
- The `<head>`: add the `<style>` block and the viewport `<meta>`.
- `class` attributes on **existing** body elements (add classes purely as styling hooks).

**You MUST NOT change:**
- The entire `<script>` block — it must be **byte-identical** to the Agent 2 handoff. No new constants, no edits to `compute`, `bindAxisSlider`, handlers, slider `min`/`max`/`step`, verdict logic, or comments. Not one character.
- The body's **element structure**: do not add, remove, reorder, or re-nest elements, and do not change any `id`. The JS reaches into the DOM by `id` and by `.parentElement` (e.g. `setBindingHighlight(el.speed.parentElement, …)`); inserting a wrapper `<div>` for layout will silently retarget those lookups and break the binding highlight. Style with CSS on the existing tree — Grid/Flex on existing containers, not new ones.

**If you cannot achieve the layout within these rules, STOP and flag it.** A required structural or logic change means Phase 2 was incomplete — that is a finding to report, not a change to make. Do not "improve" the engine while you're in here, even if you spot something; note it for the user instead.

### Self-verification before you finish
1. Extract the `<script>…</script>` block from the file as it was at the Agent 2 handoff and from your result; confirm they are **identical** (e.g. `diff` the two). If they differ, you have exceeded scope — revert the script to the handoff version.
2. Confirm every `id` present at handoff is still present and unmoved.
3. Record both the byte size and the words "script block byte-identical to Agent 2 handoff — verified by diff" in your CHANGELOG entry.

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
- **Do not touch logic — see the Scope contract above.** No edits to the `<script>` block (byte-identical), no DOM structure or `id` changes. Style the existing tree only.
- **Single file, no assets.** No external fonts, images, CSS, or CDN links. All CSS inlined in `<style>`.
- **Fixed, non-scrolling layout.** The whole interface must live in one viewport on Chrome/Android.
- Keep CSS lean (CODE-DISCIPLINE.md) — every byte ships.

## Acceptance Criteria
- [ ] Entire interface fits one viewport on mobile; nothing scrolls during input (`dvh`, fixed layout).
- [ ] Verdict band is prominent and color-coded for in/out-of-spec.
- [ ] Readout typography uses `clamp()`; high contrast, large hit areas throughout.
- [ ] Front overlap sits adjacent to speed; side overlap stands alone; binding constraint highlighted.
- [ ] **`<script>` block is byte-identical to the Agent 2 handoff** (verified by `diff`); no DOM structure or `id` changes — only `<head>`/`<style>`, viewport meta, and `class` hooks on existing elements.
- [ ] Binding highlight still works after styling (confirms no wrapper-`div` broke `.parentElement` lookups).
- [ ] No external assets or libraries; all CSS inlined.
- [ ] `wc -c prod/aerodeck-planner.html` recorded in the CHANGELOG entry, plus the "script block byte-identical — verified by diff" confirmation.
- [ ] CHANGELOG.md entry appended.
