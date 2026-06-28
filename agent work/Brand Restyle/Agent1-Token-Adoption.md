# Agent 1: Token Adoption

**Depends on:** nothing (first pass of the restyle)
**Blocks:** Agent 2

---

## Goal
Inline a curated subset of the IFT design tokens — plus the few semantic tokens AeroDeck needs that the brand doesn't define — into the planner's `<style>` `:root`, establishing the light-theme palette. No component restyling yet.

## Deliverables
1. In `prod/aerodeck-planner.html`, replace the current `:root` custom-property block with a brand-aligned one. Port **only what AeroDeck uses** from [`../style update/web/styles/tokens.css`](../style%20update/web/styles/tokens.css):
   - **Color:** `--color-bg` (#ffffff), `--color-surface` (#f0f0f0), `--color-rule` (#e5e5e5), `--color-ink-heading`, `--color-ink`, `--color-ink-muted`, `--color-accent` (#E8722C), `--color-accent-hover`, `--color-on-accent`, `--color-link`.
   - **Type:** the font stack from Agent 0 decision #2; the weight tokens (`--fw-regular…--fw-extrabold`); `--letter-caps`.
   - **Spacing / radius / shadow:** the handful actually needed (`--space-*` you'll use, `--radius-sm/md/lg/full`, `--shadow-subtle`, `--shadow-card-hover`).
2. Add **planner-only semantic tokens** the brand lacks (the planner has pass/fail states a marketing page doesn't). Derive them to harmonize with the light palette:
   - `--color-in-spec` (brand-consistent green, e.g. from `--color-status-online` #22c55e) + a light `--color-in-spec-bg`.
   - `--color-out-of-spec` (a brand-consistent red) + a light `--color-out-of-spec-bg`.
   - `--color-binding` — the constraint highlight; map to the accent orange family (`#E8722C`) or a warm amber + a light `--color-binding-bg`.
3. Do **not** restyle components in this pass. Existing selectors may render oddly mid-transition — that's expected; Agent 2 fixes them. (If leaving them fully broken is undesirable, you may point existing color/background declarations at the new tokens, but no layout/treatment changes.)

## Reference Files
- [`../style update/web/styles/tokens.css`](../style%20update/web/styles/tokens.css) — token source (copy values, do not link the file).
- `prod/aerodeck-planner.html` — the target.

## Key traps
- **Inline only — never link.** The deliverable stays one offline file. No `<link>` to `tokens.css` or Google Fonts.
- **Curate, don't dump.** Porting all of `tokens.css` (chat tokens, rail layout, motion, container breakpoints) violates size discipline. Bring only what the planner uses.
- **Scope contract (Agent 0).** `<script>` byte-identical; no DOM/`id` changes. This pass touches `:root` inside `<style>` only.
- Keep the planner's existing `clamp()` sizing approach for glanceable readouts — adopt brand *values/scale*, don't rip out responsive typography.

## Acceptance Criteria
- [ ] `:root` holds the curated IFT light-theme tokens + the three planner semantic token pairs (in-spec / out-of-spec / binding).
- [ ] Font stack matches Agent 0 decision #2; no Google Fonts `<link>` anywhere in the file.
- [ ] No tokens ported that the planner doesn't use; no chat/rail/citation tokens present.
- [ ] `<script>` byte-identical to pre-pass (verified by `diff`); no DOM/`id` changes.
- [ ] `wc -c prod/aerodeck-planner.html` and the byte-identical confirmation recorded in CHANGELOG.
- [ ] CHANGELOG.md entry appended.
