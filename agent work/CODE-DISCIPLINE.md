# Code Discipline

Rules specific to AeroDeck's single-file constraint. These extend `CLAUDE.md` — both apply.

## The Constraint

The deliverable is one self-contained `.html` file opened from local storage on an Android device. There is no build step, no minifier, no CDN. What you write is what ships. Verbose-but-correct is a bug, not a style preference.

## Rules

**No libraries or frameworks.** No imports, no CDN links, no `<script src>`. Vanilla JS and CSS only. If you find yourself reaching for a utility, write the three lines it would have been.

**No single-use abstractions.** Don't extract a helper function that is called exactly once. Inline it. Don't create a class to wrap a handful of related variables. Use a plain object literal or just the variables.

**No speculative structure.** Don't scaffold folders, modules, or comment blocks for code that doesn't exist yet. The file structure is: one file.

**Inline only what ships.** CSS in `<style>`, JS in `<script>`, sensor profiles as a `const` object literal. Nothing else.

**If it can be a constant, make it a constant.** Sensor profiles, unit conversions, categorical mappings — these are data, not logic. A `const` object is smaller and clearer than a function that returns a value based on a switch.

**No defensive padding.** No null checks for values the UI guarantees are set. No fallback defaults for impossible states. No error boundaries around math that cannot throw.

**The size test.** Before submitting, ask: could this be shorter without losing correctness or readability? If yes, shorten it.

## What Ships

**Only `prod/` ships.** The deliverable is `prod/aerodeck-planner.html` and nothing else. Everything outside `prod/` — design docs, agent briefs, changelog, these discipline docs — is dev-only and removed at ship.

**Build directly in `prod/aerodeck-planner.html`.** Each phase grows that one file; there are no separate working copies to re-inline later. This keeps the shipped artifact's size live and honest throughout dev.

**The size number that matters** is `wc -c prod/aerodeck-planner.html`. Nothing else counts toward footprint.
