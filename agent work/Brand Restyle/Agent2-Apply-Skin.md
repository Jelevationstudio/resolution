# Agent 2: Apply the Skin

**Depends on:** Agent 1 (brand tokens must exist in `:root`)
**Blocks:** nothing (final pass)

---

## Goal
Rewrite the planner's component CSS to the full IFT light brand using Agent 1's tokens, preserving the one-viewport / no-scroll / large-touch-target layout.

## Deliverables
1. In `prod/aerodeck-planner.html`, restyle every component to the light brand via the tokens — no raw hex, use the `--color-*`/`--space-*`/`--radius-*` tokens:
   - **Body / layout:** light `--color-bg`; keep `100dvh` grid, `overflow:hidden`, `minmax(0,1fr)` slider track, `env(safe-area-inset-bottom)`. Brand font stack on `body`.
   - **Verdict band:** large, the focal point. In-spec → `--color-in-spec` + `--color-in-spec-bg`; out-of-spec → `--color-out-of-spec` + `--color-out-of-spec-bg`. Keep `:has()` color-coding and `clamp()` type.
   - **Sliders / cards:** brand surfaces (`--color-surface`, `--color-rule`, `--radius-md/lg`, `--shadow-subtle`). Range thumbs/tracks in the accent. Speed + front overlap share an accent left-border (coupled cluster); side overlap stands apart with a neutral border.
   - **Derived readout:** brand muted ink labels, heading ink values, `tabular-nums` preserved.
   - **Binding highlight:** `.binding` uses `--color-binding` / `--color-binding-bg` (replaces the dark amber). Must still visibly mark the bound control(s).
   - **Flags:** brand select/checkbox styling; 44px min touch targets retained.
   - **Buttons/toggles:** follow the brand `.btn` / pill-radius conventions from `components.css` where the pin-toggle and controls map onto them (accent for active/primary, ghost/surface for inactive).
2. Match the brand's treatment language (radii, uppercase `--letter-caps` labels, accent on interaction) so the planner reads as the same family as the demo — adapted to a dense single-screen tool, not a marketing page.

## Reference Files
- [`../style update/web/styles/components.css`](../style%20update/web/styles/components.css) — treatment conventions to echo (buttons, cards, callouts, inputs). Reference only; do not copy irrelevant chat/rail rules.
- [`../style update/style-demo.html`](../style%20update/style-demo.html) — the brand rendered.
- `prod/aerodeck-planner.html` — the target (Agent 1's tokens already in `:root`).

## Key traps
- **Scope contract (Agent 0).** `<script>` byte-identical; no DOM/`id` changes; no wrapper `<div>`s (breaks `.parentElement` binding highlight). Style the existing tree with classes on existing elements.
- **Preserve §6 layout.** This is a re-skin, not a re-layout: one viewport, no scroll, large hit areas stay. If the light theme tempts a structural change, STOP and flag it.
- **Tokens, not hex.** All values via Agent 1's custom properties so the palette stays single-sourced.
- **Sunlight contrast.** Light theme is the locked decision, but keep text/element contrast high (lean on `--color-ink-heading` for key values) — don't render critical readouts in low-contrast muted ink.
- Keep CSS lean (CODE-DISCIPLINE) — every byte ships; don't port component rules the planner doesn't have.
- **Agent 1 leftovers (must replace):** `#121820` on `#derived` (dark patch on light bg), `#1e3a5f` on checked `#pin-toggle` (use accent/tint), `#6b7280` on `.side-overlap-row` (use `--color-rule` or muted), `#fff` ×2 on range-thumb borders (use `--color-bg` or `--color-rule` — invisible against white now).

## Acceptance Criteria
- [ ] Planner renders in the full IFT light theme (white bg, orange accent, brand type) and reads as the same family as `style-demo.html`.
- [ ] One viewport, nothing scrolls during input; 44px touch targets retained.
- [ ] Verdict band color-codes in/out-of-spec via brand semantic tokens; binding highlight still marks the bound control.
- [ ] Speed + front overlap visually coupled (accent border); side overlap standalone.
- [ ] All colors via tokens (no raw hex in component rules); no chat/rail CSS present.
- [ ] `<script>` byte-identical to pre-pass (verified by `diff`); no DOM/`id` changes; binding highlight confirmed still working.
- [ ] `wc -c prod/aerodeck-planner.html` + byte-identical confirmation recorded in CHANGELOG.
- [ ] CHANGELOG.md entry appended.
