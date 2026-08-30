# [PROJECT NAME] — Project Instructions

[Replace EVERY placeholder in this file before starting. Write 1-2 sentences: what this project is, who it's for, and its goal.]

## ⚠️ First thing in a fresh session — verify this isn't a hijacked template copy

Before any other work, run `git remote -v`. If it shows
`claude-pipeline-template.git` AND the placeholders in this file are already
replaced (real project name above, not `[PROJECT NAME]`) — this folder was
copied or renamed from the template and still carries its full git history +
GitHub remote. Real risk: committing here could push unrelated project code
to the template's public repo.

Fix immediately, before continuing with anything else:
```
rm -rf .git && git init && git add -A && git commit -m "chore: init from _pipeline-template"
```
Then confirm `git remote -v` is empty and `git log --oneline` shows exactly
one commit. Only then proceed with the user's request.

Skip this check if the placeholders above are still unreplaced — that means
this IS the original template, not a copy, and its git history must stay
untouched.

## Stack

- Frontend: [e.g. Next.js / Vite + React]
- Backend: [e.g. Next.js API Routes]
- Database & Auth & Storage: [e.g. Supabase]
- AI Engine: [e.g. Claude API — delete this line if unused]
- Email: [e.g. Resend — delete this line if unused]

## Conventions

- Code & comments: English
- Communication language with user: [e.g. Indonesian / English]
- Never commit `.env*` or any API key to Git
- As the project grows: rules relevant to only ONE folder (e.g. `src/lib/ai/`) go in a `CLAUDE.md` inside that folder — loaded lazily only when Claude works there, keeping this root file lean.
- [Project-specific rule — e.g. multi-tenant `organization_id` + RLS required]

## Agent Reporting Discipline (applies to every agent, every session)

- Report only what you verified **fresh in this session, right now**. If
  you reuse or reference results from an older run/file, say so explicitly
  — never mark something "Completed"/"PASS" in a way that implies it was
  just re-verified when it wasn't.
- Before starting a verification/QA task, delete or clearly disregard any
  stale scratchpad/checklist file left over from a prior failed or
  incomplete run — don't repurpose it by pasting old results into it.
- If a status changes from "blocked"/"in progress" to "done" with no
  visible verification steps in between (no tool calls, no fresh evidence),
  that's not done — go back and actually do it, or report it as blocked.

## /ship pipeline

This project uses 6 subagents (planner, designer, coder, tester, reviewer, deployer) in `.claude/agents/`.
The `/ship [feature description]` command runs them in order (designer is skipped automatically if the feature has no UI surface; deployer is offered only after the reviewer's verdict is SHIP).
See `.claude/commands/ship.md` for the full flow.

Six bundled skills in `.claude/skills/` set the quality bar per stage (full matrix in README):
the **planner** consults `relational-mastery` when the feature touches persisted data (normalization, FK lifecycles, index strategy for the spec's Data model section);
the **designer** consults `premium-design` (palette discipline, typography metrics, intentional motion, context adaptation)
and `ux-research` (user + JTBD grounding, named psychology principles, usability checklist, dark-pattern hard-lines — its 10-question diagnostic is also run by the **tester** on driven UI);
the **coder** and **reviewer** share `secure-coding` — one OWASP-derived bar applied while writing and re-audited on the diff;
the **coder** follows `systematic-debugging` on NEEDS WORK fix rounds (root cause first, 3-strikes rule);
the **tester** follows `testing-discipline` (no PASS without fresh evidence);
the **deployer** applies `secure-coding` §4 to env-var and infra config it prepares post-SHIP.

For small/simple tasks, `/ship` isn't required — just ask Claude Code directly.
