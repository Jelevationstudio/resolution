# Agent 1: Orientation Toggle

**Depends on:** nothing hard (semi-independent; touches `compute()` / `footprint()`)
**Blocks:** nothing (single pass for this folder)

---

## Goal
Add a landscape/portrait orientation toggle that selects which sensor axis is treated as **along-track** in `footprint()`, so the front-overlap ceiling is correct for how the camera is actually mounted. Default reproduces today's behavior exactly. Not a score contributor.

## The axis mapping (must be unambiguous)
Today `footprint()` is fixed: `alongTrackM = gsdM × heightPx`, `crossTrackM = gsdM × widthPx` (the wide dimension is cross-track — max swath). The toggle picks which sensor dimension is along-track:

| Orientation | along-track px | cross-track px | Meaning |
|---|---|---|---|
| **Landscape** (default = current behavior) | `heightPx` | `widthPx` | wide dimension across the flight line |
| **Portrait** (rotated 90°) | `widthPx` | `heightPx` | wide dimension along the flight line |

GSD is **orientation-independent** — only the footprint dimensions swap.

## Deliverables
1. **State** — `state.orientation` (`'landscape' | 'portrait'`), default `'landscape'` (= current behavior).
2. **`footprint()` change** — add an `orientation` parameter (keep the function **pure** — do not read global state inside it). Select along/cross-track px per the table above. `compute()` passes `state.orientation`.
3. **New DOM control** — an orientation toggle in the `#flags` area, modeled on an existing control pattern (the `#terrain-dynamic` checkbox or the `#pin-toggle` 2-button pattern). New `id` (e.g. `#orientation`). Concise UI labels (Landscape / Portrait) with the axis meaning clear; keep one viewport.
4. **Wiring** — read in `readInputs()`, `onchange` → `repaint()` via the existing reactive path.
5. **Effect** — the front-overlap ceiling (`maxFrontOverlapAtSpeed` via `fp.alongTrackM`) and `maxSpeedForFrontOverlap` recompute for the chosen orientation; the existing ceiling readout reflects it. No score contribution.

## Reference Files
- [`../../../AeroDeck-Flight-Planner-Architecture.md`](../../../AeroDeck-Flight-Planner-Architecture.md) §4 — footprint / GSD.
- [`Agent0-Overview.md`](Agent0-Overview.md) — the settled items table (what NOT to touch) + "not a score contributor."
- [`../Configurator-Reliability-Findings.md`](../Configurator-Reliability-Findings.md) §3.
- `prod/aerodeck-planner.html` — target. Model the new control on `#terrain-dynamic` / `#pin-toggle`; model wiring on the existing flag handlers.

## Key traps
- **No regression on default.** With `'landscape'` selected, `footprint()` and the front-overlap ceiling must be **behavior-identical to today** (along-track = `heightPx`). Verify before/after match in the default state.
- **Keep `footprint()` pure.** Pass `orientation` as an argument; do not reach into global `state` from the pure function.
- **GSD is orientation-independent.** Do not touch GSD math — only the footprint dimension assignment swaps.
- **Not a score contributor** (Agent 0). Orientation makes the ceiling *correct*; it is not a quality risk and adds no penalty. If a scoring touchpoint seems tempting, stop — it's out of scope for this folder.
- **Do not touch the settled non-items:** blur budget (`MAX_BLUR_PX`), capture rate, flight-dynamics assumptions, trigger behavior. All decided; leave them.
- Engine work: new DOM/`id` for the toggle is permitted; preserve all existing `id`s and the binding-highlight `.parentElement` chain. Style with brand tokens, 44px target, within one viewport. Single-file/offline/size discipline stand.

## Acceptance Criteria
- [ ] Orientation toggle present in `#flags`, brand-styled, 44px target, repaints on change.
- [ ] **Default (`landscape`) reproduces current footprint and front-overlap ceiling exactly** — no behavior change when untouched.
- [ ] Switching to `portrait` swaps along/cross-track px; the front-overlap ceiling and `maxSpeedForFrontOverlap` recompute accordingly (hand-checked to move in the expected direction).
- [ ] `footprint()` remains pure (orientation via parameter, not global read).
- [ ] GSD is unaffected by orientation.
- [ ] No score contribution added; gate logic otherwise unchanged; settled non-items untouched.
- [ ] Existing `id`s and `.parentElement` binding highlight intact; `wc -c prod/aerodeck-planner.html` recorded; CHANGELOG.md entry appended.
