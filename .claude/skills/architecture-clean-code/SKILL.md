---
name: architecture-clean-code
description: Use when writing or reviewing code structure. Keeps layers separated, removes dead code the diff leaves behind, catches duplicated logic and circular imports, and keeps files within the project's size norm. The Coder applies it while writing; the Reviewer checks the diff. Hard constraints in CLAUDE.md always win.
---

# Architecture and Clean Code

Core belief: technical debt is cheapest to stop at the diff that creates it. This skill
judges **what the change adds**, not the whole codebase.

**Hard constraints win.** If `CLAUDE.md` requires a structure this skill would flag
(for example, the deliverable must be one self-contained HTML file), follow `CLAUDE.md`
and organize inside that structure instead.

## 1. Layers

Adapt to the stack in `CLAUDE.md`. The usual split:

- **UI**: layout, rendering, user events, local display state. No database calls, no
  business rules.
- **Route or controller**: validate input, call the domain layer, shape the response.
- **Domain**: business rules and workflows, free of framework details where practical.
- **Data access**: queries, ORM calls, external APIs.

A rule placed in the wrong layer is a finding when the diff introduces it.

## 2. Dead code left by the diff

- Imports, exports, variables, and types the change made unused.
- Commented-out code (git keeps the history).
- Branches that can no longer run, and feature flags nothing reads.
- Files the change replaced but didn't delete.

## 3. Duplication

When the diff copies existing logic, reuse or extract it. Apply the rule of three: a
second copy is a note, a third copy must be extracted. Don't build an abstraction for a
single use (the Coder's YAGNI ladder still applies).

## 4. Coupling and conventions

- No circular imports (A imports B, B imports A).
- New files follow the folder layout and naming already in the project.
- Configuration and secrets stay out of business logic.

## 5. Size

Treat a file growing past roughly 300 lines, or a function past roughly 50, as a signal
to split by responsibility. These are defaults: `CLAUDE.md` can set others, and a hard
constraint can override them entirely.

## 6. Reviewer check

Findings count only when the diff introduced or worsened them; existing debt is noted
for the user, never used to expand scope. Severity: layer violations, dead code left by
the diff, a third copy of the same logic, or a new circular import = **NEEDS WORK**.

## Done check

- [ ] New code sits in the right layer for this stack
- [ ] No dead code, unused imports, or replaced files left behind
- [ ] No logic copied a third time; no abstraction for a single use
- [ ] No circular imports; project conventions followed
- [ ] Files and functions within the project's size norm, or a stated reason
- [ ] Nothing here overrides a hard constraint in `CLAUDE.md`

> Credits: distilled for this pipeline from a custom clean-code checklist by the
> template author.
