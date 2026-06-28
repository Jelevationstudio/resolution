# Agent Work

Structured task planning for multi-step features. Each feature gets its own folder with numbered agent files that define scope, dependencies, and acceptance criteria.

## Format

```
Agent Work/
  CHANGELOG.md              # Living handoff log (append-only; see Changelog below)
  README.md                 # Conventions + changelog entry template
  Feature-Name/
    Agent0-Overview.md        # Context, execution order, scope, what's NOT in scope
    Agent1-Task-Name.md       # First task
    Agent2-Task-Name.md       # Second task
    ...
```

## Agent File Structure

Each agent file follows this template:

```markdown
# Agent N: Task Name

**Depends on:** Agent X (reason)
**Blocks:** Agent Y, Z

---

## Goal
One sentence on what this agent delivers.

## Deliverables
Numbered list of concrete outputs (files, endpoints, tests).

## Reference Files
Paths to existing code this agent reads or modifies.

## Acceptance Criteria
- [ ] Checkboxes for done/not-done
```

## Agent 0 Overview

`Agent0-Overview.md` is read first in every session (see Rules). Beyond context, scope, and execution order, it **must** open with a `Read first (every session)` block that links the governing docs — so they're referenced once instead of pasted into each session:

```markdown
## Read first (every session)
Each session = this **Agent 0 overview** + the one **Agent # brief** for that session. These govern all of it, so they're linked here instead of pasted in:
- [`../CLAUDE.md`](../CLAUDE.md) — behavioral guidelines (Think Before Coding · Simplicity First · Surgical Changes · Goal-Driven Execution).
- `<folder's reviewed reference doc, if any>` — the spec / manifest / README carrying the line-level detail and sign-off for this feature.

Read both before acting on any Agent # brief.
```

This lets you paste only Agent 0 + the one Agent # brief per session; the agent reads `../CLAUDE.md` and the folder's reference doc on its own. Drop the second bullet if the folder has no separate reference doc; keep the `../CLAUDE.md` link always.

## Execution Order

Define in Agent0-Overview.md using this format:

```
Execution flow:
* Agent 1 + Agent 2 run in parallel (no dependencies)
* Agent 3 waits for Agent 1+2 (reason)
* Agent 4 + Agent 5 run in parallel (both depend on Agent 3, not each other)
* Agent 6 waits for Agent 4+5 (reason)
```

## Changelog

[`CHANGELOG.md`](CHANGELOG.md) is the cross-session handoff log. It mirrors each agent brief (goal, dependencies, deliverables, acceptance) but records **what actually shipped** after a pass—not full specs. Feature READMEs (e.g. `Utility Detail/README.md`) stay the system-of-record for design; the changelog is for the next agent with no chat history.

### Agent instructions (read before every pass)

1. Read [`CHANGELOG.md`](CHANGELOG.md) (newest entry first), then open the relevant feature folder / agent brief.
2. When starting a pass, you may be told: *“Read `Agent Work/README.md` § Changelog; append an entry when done.”*
3. **Append one entry per completed pass** to `CHANGELOG.md` immediately below the pointer block at the top of that file. **Do not rewrite or delete** prior entries.
4. **Order:** newest entry first (reverse chronological).
5. **When to log:** Any pass that changes repo code, contracts/APIs, configs, or materially updates specs under `Agent Work/`. Skip pure Q&A with no artifacts.
6. **Paths:** Use repo-root paths (`src/...`, `prometheus_engine/...`, `Agent Work/Feature/...`).
7. **Feature context:** If work maps to `Agent Work/<Feature>/`, name the folder and agent brief path (e.g. `Agent Work/Utility Detail/agents/03-geometry-engine.md`). Cross-cutting work with no feature folder → `Feature: (cross-cutting)`.
8. **Before deleting a completed feature folder** (see Rules), ensure its final state is captured in `CHANGELOG.md`.

### Changelog entry template

Copy this block into [`CHANGELOG.md`](CHANGELOG.md) for each pass. Fill every section; `Next agent should` and `Deferred` are required.

```markdown
## <Feature> — <short title> (<YYYY-MM-DD>)

| Field | Value |
|-------|--------|
| **Agent / pass** | e.g. `Utility Detail / Agent 3` or `drone-registry task` |
| **Agent brief** | Path to brief, or `—` if cross-cutting / none |
| **Status** | `shipped` \| `partial` \| `blocked` \| `spec-only` |
| **Depends on** | Prior passes or files this assumed done |
| **Unblocks** | What a later pass can now do |

### Goal (one sentence)
What this pass was meant to deliver.

### Changed
- **Agent Work:** `Agent Work/...` (specs, configs, DevWork — note if throwaway)
- **App / engine:** `src/...`, `prometheus_engine/...`, etc.

### Behavior / contract delta
What changed for users, APIs, file formats, or defaults (not a file list).

### Verify
- Commands run: `pytest ...`, `npm test ...`, manual step
- Result: `pass` \| `fail` \| `not run`

### Deferred
- Item — **why** deferred — **suggested owner pass** (e.g. sensor pass, Agent 8)

### Pitfalls / do not redo
Wrong approaches tried, confirmed assumptions, or “looks broken but intentional”.

### Next agent should
1–3 bullets: exact next task, files to open first, and what *not* to touch yet.
```

## Reference Docs

- [`CLAUDE.md`](CLAUDE.md) — behavioral guidelines (Think Before Coding · Simplicity First · Surgical Changes · Goal-Driven Execution)
- [`CODE-DISCIPLINE.md`](CODE-DISCIPLINE.md) — file size and code complexity rules specific to AeroDeck's single-file constraint

Both docs apply to every agent pass. `Agent0-Overview.md` links them in the `Read first` block so agents load them at session start.

## Rules

- **One folder per feature.** Don't mix unrelated work.
- **Specs go in the agent that implements them.** Don't create separate spec-only agents — append the specification directly into the implementation agent's deliverables.
- **Agent 0 is always the overview.** It defines execution order, scope, and what's excluded, and opens with the `Read first (every session)` block linking `../CLAUDE.md` + any folder reference doc (see [Agent 0 Overview](#agent-0-overview)).
- **Keep agent count low.** If you need more than 7 agents, the feature should be split into separate folders.
- **Log every pass in `CHANGELOG.md`.** Use the [Changelog entry template](#changelog-entry-template) above.
- **Delete completed folders.** Once all agents in a folder are done and verified, log the final state in `CHANGELOG.md`, then remove the folder. Don't accumulate stale plans.
