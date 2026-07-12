# [PROJECT NAME] — Project Instructions

[Replace EVERY placeholder in this file before starting. Write 1-2 sentences: what this project is, who it's for, and its goal.]

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
- [Project-specific rule — e.g. multi-tenant `organization_id` + RLS required]

## /ship pipeline

This project uses 5 subagents (planner, designer, coder, tester, reviewer) in `.claude/agents/`.
The `/ship [feature description]` command runs them in order (designer is skipped automatically if the feature has no UI surface).
See `.claude/commands/ship.md` for the full flow.

Four bundled skills in `.claude/skills/` set the quality bar per stage (full matrix in README):
the **designer** consults `premium-design` (palette discipline, typography metrics, intentional motion, context adaptation);
the **coder** and **reviewer** share `secure-coding` — one OWASP-derived bar applied while writing and re-audited on the diff;
the **coder** follows `systematic-debugging` on NEEDS WORK fix rounds (root cause first, 3-strikes rule);
the **tester** follows `testing-discipline` (no PASS without fresh evidence).

For small/simple tasks, `/ship` isn't required — just ask Claude Code directly.
