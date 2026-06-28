# Agent 0: Brand Restyle — Overview

## Read first (every session)
Each session = this **Agent 0 overview** + the one **Agent # brief** for that session. These govern all of it, so they're linked here instead of pasted in:
- [`../CLAUDE.md`](../CLAUDE.md) — behavioral guidelines.
- [`../CODE-DISCIPLINE.md`](../CODE-DISCIPLINE.md) — single-file size/complexity rules and what ships.
- [`../../AeroDeck-Flight-Planner-Architecture.md`](../../AeroDeck-Flight-Planner-Architecture.md) §6 — the layout + UX constraints the restyle must preserve.
- **Brand reference assets** (read-only source of the design language — do **not** link to or copy wholesale):
  - [`../style update/web/styles/tokens.css`](../style%20update/web/styles/tokens.css) — Inspired Flight design tokens (color, type, spacing, radius, shadow).
  - [`../style update/web/styles/components.css`](../style%20update/web/styles/components.css) — component conventions (buttons, cards, callouts, inputs).
  - [`../style update/style-demo.html`](../style%20update/style-demo.html) — the brand rendered, for visual reference.

Read these before acting on any Agent # brief.

---

## Context

We are re-skinning the **AeroDeck planner** (`prod/aerodeck-planner.html`) to the **Inspired Flight Technologies (IFT)** brand. The brand assets above come from a separate IFT support-site project; we are adopting their *design language* (tokens + component treatment), not their markup or their CDN dependencies.

This is a **visual restyle only**. The planner's behavior, computation, and DOM are finished and locked — see the Scope contract below. We are replacing the dark instrument `<style>` with a brand-aligned one.

## Locked decisions (do not relitigate)

1. **Full brand light theme.** White background, light surfaces, IFT orange accent (`#E8722C`). The existing dark instrument theme is replaced, not preserved. (Trade-off acknowledged: less ideal in direct sun; the decision stands.)
2. **Fonts: closest-match stack, no CDN, no embedding.** AeroDeck is offline (`file://`), so the Google Fonts `<link>` from the demo is forbidden, and embedding Inter as base64 is forbidden (size discipline). Use:
   `font-family: 'Inter', 'Roboto', system-ui, -apple-system, 'Segoe UI', sans-serif;`
   On the Chrome/Android target this resolves to Roboto — visually close to Inter. Zero added bytes.
3. **Single inlined file, preserved.** Everything stays in `prod/aerodeck-planner.html`. No external CSS links, no new files shipped. Port only the *subset* of tokens AeroDeck actually uses — not all of `tokens.css`, and none of the chat/rail/citation component CSS, which is irrelevant here.
4. **One-viewport, no-scroll, large touch targets stay.** Architecture §6 still governs layout. The brand changes color/type/treatment, not the fixed field-tool layout philosophy.

## Scope contract (the CSS-only boundary — same as it sounds)

**You MAY change:** the `<head>` `<style>` block (replace it wholesale) and the viewport `<meta>`; `class` attributes on **existing** body elements as styling hooks.

**You MUST NOT change:**
- The `<script>` block — **byte-identical** to the current `prod/aerodeck-planner.html`. Not one character.
- The body's **element structure** or any `id`. The JS reaches the DOM by `id` and by `.parentElement` (e.g. `setBindingHighlight(el.speed.parentElement, …)`); inserting a wrapper `<div>` will silently break the binding highlight. Style the existing tree.

If the brand can't be applied within these rules, **STOP and flag it** — do not edit the engine.

### Self-verification before finishing any pass
1. `diff` the `<script>` block against the pre-pass version — must be identical; revert it if not.
2. Confirm every `id` is still present and unmoved; element/`<div>` counts unchanged.
3. Record byte size + "script block byte-identical — verified by diff" in the CHANGELOG entry.

## Execution flow

```
Agent 1: Token adoption   → inline the curated IFT token subset + planner semantic tokens into :root
Agent 2: Apply the skin    → rewrite the component CSS to the light brand using those tokens
```

Sequential. Agent 2 depends on Agent 1's tokens existing. Both obey the Scope contract. No agent reopens after the next starts.

## Logging
Append one entry per completed pass to [`../CHANGELOG.md`](../CHANGELOG.md) using the README template. Newest first.
