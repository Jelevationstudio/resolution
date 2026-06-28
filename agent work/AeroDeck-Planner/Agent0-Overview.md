# Agent 0: AeroDeck Planner — Overview

## Read first (every session)
Each session = this **Agent 0 overview** + the one **Agent # brief** for that session. These govern all of it, so they're linked here instead of pasted in:
- [`../CLAUDE.md`](../CLAUDE.md) — behavioral guidelines (Think Before Coding · Simplicity First · Surgical Changes · Goal-Driven Execution).
- [`../CODE-DISCIPLINE.md`](../CODE-DISCIPLINE.md) — single-file size/complexity rules and what ships.
- [`../../AeroDeck-Flight-Planner-Architecture.md`](../../AeroDeck-Flight-Planner-Architecture.md) — the line-level spec. The system-of-record for variable roles, math, and layout.
- [`../../AeroDeck-Flight-Planner-Executive-Summary.md`](../../AeroDeck-Flight-Planner-Executive-Summary.md) — product intent and the "lookup-table vs. answer" framing.

Read all four before acting on any Agent # brief. The architecture doc wins any conflict with a brief; if they disagree, stop and flag it.

---

## Context

AeroDeck is an offline-native drone flight-planning tool: a deterministic physics calculator that collapses vendor spec ranges to a single actionable plan once the operator declares their constraints. The deliverable is **one self-contained HTML file** opened from local storage on Chrome/Android. No build step, no libraries, no network.

The product **is** the coupling logic — the visible, real-time relationship between altitude/GSD, speed, and front/side overlap. The single most common way an agent will get this wrong is by softening that coupling: treating altitude and speed as co-equal inputs, making altitude/GSD both editable, or letting a secondary factor (light, terrain) emit a number. Don't.

## The build artifact

Everything is built **directly in `prod/aerodeck-planner.html`**. Each phase grows that one file; there are no separate working copies. `prod/` is the only thing that ships (see [`../CODE-DISCIPLINE.md`](../CODE-DISCIPLINE.md)). Watch size with `wc -c prod/aerodeck-planner.html`.

## Scope

In scope: the three-DOF variable model, closed-form computation core, slider/readout reactivity, pinned-end toggle, flags (light, terrain), and the mobile-first one-viewport layout.

NOT in scope (architecture §8): flight-time calculation, exposure control, DEM/terrain drape, mission execution / waypoint export / aircraft comms, any network or stored ephemeris.

## Variable model (the contract, from architecture §3)

- **Altitude / GSD** — one axis, bijective lock (`GSD = pixel pitch × altitude / focal length`). A toggle picks which end is held; held end is a field/slider, the other is a derived readout. **Never both live.**
- **Speed** — always-free slider sitting under **two** independent ceilings: the **blur ceiling** (shutter × altitude) and the **capture-rate wall** (set by the front-overlap target). Both are clamps, not derivations; either can be the binding one.
- **Front overlap** — **free input slider** the operator spends (lower it for faster plans, raise it for robustness), living under a **derived ceiling** = max achievable overlap at the current speed/altitude via capture rate. A **soft clamp, not a rigid lock**: the overlap *number* is input; the *ceiling* is derived and the input is checked against it. Distinct from altitude↔GSD, which is a rigid bijective lock where one end is exactly determined by the other.
- **Side overlap** — free input slider, independent (line spacing; pure coverage knob, no speed interaction).
- **Light** — flag/clamp only → shutter-floor implication. Never a number.
- **Terrain** — flag only (static vs. dynamic AGL). No terrain math.

## Execution flow

```
Phase 1 → Agent 1: Computation Core      (pure JS + sensor const; no UI)
Phase 2 → Agent 2: Structure + Reactivity (HTML, sliders, toggle, flags, oninput wiring)
   ↳ optional planning beat: re-look at Agent 3 brief once coupling is testable on a device
Phase 3 → Agent 3: Visual Design + Polish (styling only; no logic changes)
```

Strictly sequential. Each agent consumes the prior agent's `prod/aerodeck-planner.html` and hands off the same single file. **No agent waits on anything other than the previous one completing.** No cyclical dependencies — Agent 1 never reopens to "finish" after Agent 3.

## Handoff contract

- **Agent 1 → 2:** computation functions + sensor profile `const` exist and return correct values for known inputs. No HTML/UI yet (functions may live in the `<script>` of the file with a minimal/no `<body>`).
- **Agent 2 → 3:** the file is functional and unstyled — every slider moves, the toggle swaps slider↔readout, derived values repaint live, the verdict reflects in/out-of-spec. Behavior is correct; only appearance is unfinished.
- **Agent 3 → done:** full visual treatment applied; zero changes to computation or reactivity. If Agent 3 is editing math or wiring, something upstream was incomplete — stop and flag.

## Logging

Append one entry per completed pass to [`../CHANGELOG.md`](../CHANGELOG.md) using the README template. Newest first. Do not rewrite prior entries.
