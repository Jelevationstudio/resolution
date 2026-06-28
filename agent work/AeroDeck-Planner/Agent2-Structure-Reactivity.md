# Agent 2: Structure + Reactivity

**Depends on:** Agent 1 (computation core + sensors must exist and be correct)
**Blocks:** Agent 3

---

## Goal
Wrap Agent 1's computation core in functional HTML — sliders, readouts, the pinned-end toggle, and flags — with live `oninput` reactivity. Correct behavior, no styling.

## Deliverables
1. In the same `prod/aerodeck-planner.html`, add `<body>` structure per architecture §6:
   - **Verdict band** — pinned spec + live in-spec / out-of-spec state.
   - **Pinned-end toggle** — selects which end of the altitude/GSD axis is held; **swaps which control is a live slider and which is a derived readout** in that slot.
   - **Sliders** — the held end of altitude/GSD, plus speed, front overlap, side overlap. The derived end of altitude/GSD shows as a readout.
   - **Derived readout** — derived altitude or GSD, blur margin, and the **front-overlap ceiling** (`maxFrontOverlapAtSpeed`) shown next to the front-overlap slider as the headroom the target must stay under, with the **binding constraint highlighted**.
   - **Flags** — sensor profile dropdown, light category selector, terrain toggle.
2. Reactivity per architecture §7:
   - Sliders drive an `oninput` handler that recomputes and repaints the readout every frame.
   - **No calculate button.** Roles are fixed: sliders independent, readout dependent.
   - The pinned-end toggle re-binds slider↔readout, then repaints.
   - Light/terrain flags update state and trigger the same repaint.

## Reference Files
- [`../../AeroDeck-Flight-Planner-Architecture.md`](../../AeroDeck-Flight-Planner-Architecture.md) §5 (coupling logic), §6 (layout), §7 (reactivity).
- Agent 1's functions in `prod/aerodeck-planner.html`.

## Key traps
- **Altitude/GSD are never both editable.** The toggle decides which is the slider; the other is a readout the operator cannot type into. The layout slot must support the slider↔readout swap.
- **The operator cannot edit derived values.** Readouts are output only — discovered by moving knobs.
- **Coupling must be visible.** Front overlap is a **free input slider** the operator spends, not a derived readout — its value is what they dial into the flight controller. Pushing speed up does not change the front-overlap number; instead it **lowers the derived overlap ceiling** (`maxFrontOverlapAtSpeed`) shown beside the slider, and once that ceiling drops below the target, the plan goes **out-of-spec with capture rate as the binding constraint**. Speed has two simultaneous consequences that light up together: the blur ceiling and the front-overlap ceiling. The operator can resolve an out-of-spec front overlap two ways — lower the overlap target, or raise altitude (which lifts the ceiling) — and the UI should make both visibly work. Side overlap is independent — it never reacts to speed. Verify both walls actually trigger.
- **Secondary factors emit flags/clamps, not numbers.** Light changes the shutter floor feeding the blur ceiling; terrain only annotates. Neither prints a recommended number.
- Reuse Agent 1's functions as-is. Do not re-derive math in the UI layer.

## Acceptance Criteria
- [ ] All sliders move and repaint the readout live, every frame, no calculate button.
- [ ] Pinned-end toggle swaps slider↔readout for altitude/GSD and repaints correctly.
- [ ] Front and side overlap are free input sliders the operator reads off directly; neither overlap *value* is recomputed by the tool.
- [ ] The derived front-overlap ceiling (`maxFrontOverlapAtSpeed`) is shown beside the front-overlap slider and drops as speed rises / rises as altitude rises.
- [ ] When the front-overlap target exceeds its ceiling, the plan flips out-of-spec with capture rate as the binding constraint; lowering the target OR raising altitude clears it.
- [ ] Crossing the blur ceiling flips out-of-spec for blur independently; side overlap never reacts to speed.
- [ ] Verdict shows in-spec vs out-of-spec against the pinned end; binding constraint is highlighted.
- [ ] Sensor dropdown, light selector, terrain toggle all update state and repaint.
- [ ] Derived values are not user-editable.
- [ ] No libraries; no logic duplicated from Agent 1 (CODE-DISCIPLINE.md).
- [ ] CHANGELOG.md entry appended.
