# Agent 2: Two-Layer Verdict Rendering

**Depends on:** Agent 1 (scoring framework + spec must exist)
**Blocks:** Ceiling-Violation-Visibility folder (shares this verdict rendering)

---

## Goal
Render the two-layer verdict — the existing binary gate plus the new robustness band + estimated-success % — glance-first, within one viewport, with cautious non-guarantee framing.

## Deliverables
1. **DOM (engine work — new elements permitted):** add a score layer near the verdict band:
   - **Band** (e.g. Strong / Adequate / Weak) — the glance.
   - **Estimated success** ≈% with a persistent **`Est.`** prefix and brief non-guarantee microcopy (e.g. *"estimate · not a guarantee"*).
   - **Reasons** — a compact, capped decomposition list (each item = one named value, minimal text), placed in the readout zone, not a wall of text.
2. **Wire to `compute().score`** in `repaint()`: paint band + est% + reasons every frame from the framework output. Reuse the existing reactive path; don't re-derive scoring in the UI.
3. **Hidden until active.** When `score.active` is false (no contributors yet — current state), **render no score layer at all** — the verdict looks exactly as it does today. This prevents shipping a misleading static "~95%."
4. **Gate dominates.** When the gate fails (OUT OF SPEC), suppress the est% per the spec (gate-fail behavior) and keep the binary state the visual headline; the score is always secondary to the gate.
5. **Style** with brand light-theme tokens (band may reuse `--color-in-spec/out-of-spec/binding` or a graded ramp built from existing tokens). 44px targets, one-viewport/no-scroll preserved.

## Reference Files
- [`Scoring-Spec.md`](Scoring-Spec.md) — semantics, bands, gate-fail behavior, non-guarantee stance.
- [`Agent0-Overview.md`](Agent0-Overview.md) — locked principles + settled decisions.
- `prod/aerodeck-planner.html` — target (Agent 1's framework already returns `compute().score`).

## Key traps
- **Two layers visually distinct.** Gate is the headline; band/est% is secondary. Never blend them into one number or one color blob.
- **Non-guarantee framing is mandatory** (user directive). No absolute/certainty language anywhere; `Est.` prefix + microcopy always present when the score shows. The sub-100 cap from Agent 1 backs this up.
- **One viewport, no scroll.** This is the tight constraint — the reasons list must be compact and capped, not unbounded. If it can't fit, shorten the treatment, don't add scroll.
- **Glance-first.** Band readable instantly; reasons scannable, minimal.
- **Hidden-when-dormant** must actually hold — verify the current zero-contributor build shows the unchanged verdict.
- Preserve existing `id`s and the binding-highlight `.parentElement` chain; reuse Agent 1's output, don't recompute.

## Acceptance Criteria
- [ ] Two-layer verdict renders when `score.active`: gate state (headline) + band + `Est.` ≈% (with non-guarantee microcopy) + compact reasons.
- [ ] With the current dormant framework (`active:false`), **no score layer shows** — verdict is byte-for-behavior identical to today.
- [ ] Gate-fail suppresses the est% per spec; gate stays the visual headline.
- [ ] No absolute/guarantee language; `Est.` + microcopy present whenever the score shows.
- [ ] One viewport, nothing scrolls during input; 44px targets retained; brand tokens only (no raw hex).
- [ ] Existing `id`s and `.parentElement` binding highlight intact and working.
- [ ] `wc -c prod/aerodeck-planner.html` recorded; CHANGELOG.md entry appended.
