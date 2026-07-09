<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-Template-D97757?logo=anthropic&logoColor=white" alt="Claude Code Template">
  <img src="https://img.shields.io/badge/Pipeline-5%20Agents-informational" alt="5-Agent Pipeline">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License">
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.id.md">Bahasa Indonesia</a>
</p>

---

# _pipeline-template

A template framework for **apps with complex CRUD / logic** — layered features, backend + frontend, needing a quality gate before release. Sibling of [`_landing-template`](https://github.com/redhoram/claude-landing-template) (for landing pages & simple apps), but optimized for structured feature work.

### Philosophy: batch pipeline with a quality gate

Complex features need planning, implementation, testing, and review kept clearly separate — so each stage is auditable and nothing slips. The flow is doc-driven: each subagent writes an artifact to `.pipeline/`, then hands off to the next stage.

- **`/ship [feature description]`**: runs the 5-subagent pipeline end-to-end — planner → designer → coder → tester → reviewer. The reviewer gives the final verdict (SHIP / NEEDS WORK / BLOCK).
- **Open-questions gate**: if the planner's spec has unresolved open questions, the pipeline stops and asks you before designer/coder ever runs — no stage inherits a guess.
- **Model tiering**: planner & reviewer run on `opus` (the leverage is in planning and judgment), coder & tester run on `sonnet` (cheap execution against a spec that's already unambiguous).
- For small fixes/tweaks: `/ship` isn't required — just ask Claude Code directly.

### Difference from `_landing-template`

| | `_pipeline-template` (complex CRUD) | `_landing-template` (landing & simple apps) |
|---|---|---|
| Mode | Batch, doc-driven, fire-and-forget | Visual loop + interactive |
| Agents | planner → designer → coder → tester → reviewer | designer → builder → qa |
| Final gate | Reviewer (SHIP/BLOCK, security-focused) | QA (punch-list, UI/UX-focused) |
| Command | `/ship` | `/craft` (0→1) + `/polish` (1→great) |
| Verification | build/lint/test + scenarios | live preview: render, responsive, console |

### Contents

- `CLAUDE.md` — project instructions: stack, conventions, pipeline flow. **Fill in the placeholders first.**
- `.claude/agents/` — `planner`, `designer`, `coder`, `tester`, `reviewer`
- `.claude/commands/` — `ship`
- `.claude/skills/` — `premium-design` (bundled design-quality skill the designer enforces)
- `.pipeline/` — handoff workspace (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`)

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed — `/ship` is a **Claude Code slash command**, typed inside a Claude Code session in the project folder (it's not an npm script).
- Your stack's toolchain (e.g. Node.js + npm) so the tester can actually run build/lint/test.

### Start a new project

1. Copy `_pipeline-template` → rename the folder
2. Fill in `CLAUDE.md` (replace all placeholders: stack, conventions, project rules)
3. Scaffold your stack
4. `git init`
5. Open Claude Code in the folder → run `/ship [feature description]`

### Standard

What sets the output apart from instant generators: **planned, tested, and reviewed** — not just "it runs". Details in `CLAUDE.md` + each agent file.

And **fail loud, not silent**: the coder stops on a missing or ambiguous spec instead of guessing, the tester reports failures verbatim without patching over them, and the verdict is mechanical — any FAIL or NOT TESTED criterion means it cannot be SHIP.

### Credit

**Author**
- Redho Ramadhani — [linkedin.com/in/redhoramadhanihamid](https://id.linkedin.com/in/redhoramadhanihamid) · [github.com/redhoram](https://github.com/redhoram)

Built with [Claude Code](https://claude.com/claude-code).
