# AeroDeck Flight Planner — Executive Summary

**Product.** An offline-native field tool that gives drone operators their optimal flight-planning settings for the conditions in front of them, replacing on-the-spot mental math and spec-sheet lookups with immediate, deterministic outputs.

**Problem.** Operators repeatedly ask "what altitude and overlap do you recommend?"—a question that has no answer until their own constraints are supplied. The best vendor guidance is an unconditioned range (e.g. a 15–45 mph envelope) that is simultaneously correct and useless: correct because it spans every job, sensor, and quality target; useless because exactly one narrow band inside it is right for the operator's specific job, and the table cannot say which without their GSD spec, light, and blur tolerance. Aerial survey quality rests on a tight coupling between altitude, ground speed, and image overlap, mediated by the sensor and the light—relationships that are well documented but rarely computed in the field, so plans silently degrade and the failure stays invisible until the data is processed.

**Approach.** The tool is the function that collapses the vendor's broad input space to a single point once the operator spends their constraints. It exposes the minimum honest input set and derives everything else from known photogrammetric relationships—no learned model, no new science, just fast resolution of a solved problem packaged for speed of use. The genuine degrees of freedom are fewer than they appear, because GSD and altitude are not independent—they are the same axis viewed from opposite ends, locked by the sensor:

- **The altitude/GSD axis with a pinned-end toggle** — GSD = pixel pitch × altitude / focal length, a rigid one-to-one lock. The operator pins one end (the job's resolution spec, or a hard altitude) and the other is derived. The toggle picks which end is held; the held end is a field, the derived end is a live readout.
- **Speed** — the always-free knob. Altitude does not derive speed; it sets speed's blur *ceiling* (blur ≈ speed × shutter / GSD). Speed stays free underneath that clamp, slid against the budget.
- **Front and side overlap as separate sliders** — not one control. Front (along-track) is coupled to speed through the trigger interval, so pushing speed up eats front overlap—it shares speed's failure mode and reads as part of the speed cluster. Side (cross-track) is an independent coverage/robustness knob set by line spacing.
- **One sensor profile dropdown** — presets (e.g., X10 wide, Mavic 3E) that resolve focal length and pixel pitch behind the scenes, so operators pick gear they recognize, not optical specs.
- **One terrain toggle** — static AGL versus dynamic. Static flags the "assumes flat ground" risk; dynamic asserts constant AGL. A declaration of intent, not terrain math.
- **A categorical light selector** — overcast / bright midday / low-angle — that maps to how operators actually read conditions, resolving to a lighting-quality flag and a shutter-floor implication.

**Scope discipline.** The camera owns exposure (ISO/shutter); the tool does not control it. Secondary factors—light, terrain mode—never emit a number, only a flag or a clamp on a primary output. This keeps the product a planning guide, not a camera controller or a mission executor.

**Constraints.** Fully offline, no DEM dependency, minimal data footprint. Solar and exposure consequences are approximated through categorical selection rather than stored ephemeris or live data.

**Value.** The difference being sold is lookup-table vs. answer—a categorical improvement, not an incremental convenience. The operator stops asking "what do you recommend?" and starts declaring "I need 2cm, cloudy, prioritizing speed," then walks to the aircraft in seconds with a resolved plan and a clear view of the one binding constraint—blur ceiling, light, or terrain risk—that would otherwise have quietly ruined the collection.

**Output.** The four values the operator dials into the flight controller are front overlap, side overlap, speed, and altitude—so the tool inverts the usual flow: the operator declares the fixed constraint (target GSD, or the conditions they're stuck with) and the tool resolves altitude, speed, and front/side overlap as the recommended settings, rather than making them guess and check.
