<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-Template-D97757?logo=anthropic&logoColor=white" alt="Claude Code Template">
  <img src="https://img.shields.io/badge/Pipeline-6%20Agents-informational" alt="6-Agent Pipeline">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License">
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.id.md">Bahasa Indonesia</a>
</p>

---

# _pipeline-template

A template framework for **apps with complex CRUD / logic**: layered features, backend + frontend, needing a quality gate before release. Sibling of [`_landing-template`](https://github.com/redhoram/claude-landing-template) (for landing pages & simple apps), but optimized for structured feature work.

### Philosophy: batch pipeline with a quality gate

Complex features need planning, implementation, testing, and review kept clearly separate, so each stage is auditable and nothing slips. The flow is doc-driven: each subagent writes an artifact to `.pipeline/`, then hands off to the next stage.

- **`/ship [feature description]`**: runs the pipeline end-to-end (planner → designer → coder → tester → reviewer). The reviewer gives the final verdict (SHIP / NEEDS WORK / BLOCK); on SHIP, an optional **deployer** stage prepares the change for production (env-var inventory, infra config, deploy report).
- **Open-questions gate**: if the planner's spec has unresolved open questions, the pipeline stops and asks you before designer/coder ever runs, so no stage inherits a guess.
- **`/fix [change request]`**: the lighter path for revising a feature that already exists (rename, copy change, layout tweak, bug fix). The main session writes a short spec, then coder → tester → reviewer run. Revisions still leave a spec, a test report, and a verdict behind instead of slipping through as untracked edits.
- **Model tiering**: planner, designer & reviewer run on `opus` (the leverage is in planning and judgment), coder, tester & deployer run on `sonnet` (cheap execution against a spec that's already unambiguous). Override any agent per project in the `Model override` table of `CLAUDE.md`, without editing the agent file.
- **Visual check**: on any UI change the tester renders each touched screen at the widths in `CLAUDE.md`, looks at the screenshots, and reports what's broken even if no criterion mentions it. Severity 3+ blocks SHIP.
- **Audit trail**: each run moves the previous `.pipeline/` artifacts (including `shots/`) to `.pipeline/archive/<timestamp>/` instead of deleting them.
- **Build gate and fatal stops**: the coder must pass the build with at most 3 fix attempts, otherwise it writes `## BLOCKED` into the changelog. `/ship` and `/fix` check every stage's output and stop the chain on a missing file, unresolved questions, or a `BLOCKED` section, instead of pushing on.
- **Commit on SHIP**: the pipeline offers to commit only the files in the changelog, staged by explicit path. It never pushes without your confirmation.
- For one-line tweaks: neither command is required; just ask Claude Code directly.

### Difference from `_landing-template`

| | `_pipeline-template` (complex CRUD) | `_landing-template` (landing & simple apps) |
|---|---|---|
| Mode | Batch, doc-driven, fire-and-forget | Visual loop + interactive |
| Agents | planner → designer → coder → tester → reviewer (→ deployer after SHIP) | writer → designer → builder → qa |
| Final gate | Reviewer (SHIP/BLOCK, security-focused) | QA (punch-list, UI/UX-focused) |
| Command | `/ship` (new feature) + `/fix` (revision) | `/craft` (0→1) + `/polish` (1→great) |
| Verification | build/lint/test + scenarios | live preview: render, responsive, console |

### Contents

- `CLAUDE.md`: project instructions covering stack, hard constraints, final decisions, final files, content sources, verification commands, deployment target, model override, conventions, pipeline flow. **Fill in the placeholders first; delete the sections that don't apply.**
- `.claude/agents/`: `planner`, `designer`, `coder`, `tester`, `reviewer`, `deployer`
- `.claude/commands/`: `ship`, `fix`
- `.claude/skills/`: 14 bundled skills the agents enforce (matrix below)
- `.pipeline/`: handoff workspace (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`, `deploy-report.md`, `shots/`), with past runs in `.pipeline/archive/`

### Skill matrix

Each agent reads only the skills that belong to its stage: documented here so the mapping is auditable:

| Skill | Used by | Enforces |
|---|---|---|
| `relational-mastery` | planner (data features only) | Normalization (3NF), FK lifecycles with explicit ON DELETE/UPDATE, index-per-query justification, ownership/tenant columns, ORM-ready schema in the spec's Data model section |
| `premium-design` | designer | Palette discipline, typography metrics, intentional motion, marketing-vs-product intensity |
| `ux-research` | designer + tester | Behavioral grounding: user + JTBD first, 3-5 named psychology principles per surface, Nielsen/Krug design-time checklist, dark-pattern hard-lines. Tester runs its 10-question usability diagnostic (severity 0-4) on the driven UI |
| `secure-coding` | coder + reviewer + deployer | One shared security bar (OWASP-derived): injection, authz/ownership, secrets, fail-closed errors, LLM risks. Coder applies it while writing; reviewer audits the same list via its §7 diff-scan; deployer applies §4 to post-SHIP env/infra prep |
| `systematic-debugging` | coder (fix rounds) | Root cause before any fix, one fix per cause, 3-strikes rule: stop and report instead of thrashing |
| `testing-discipline` | tester | Evidence gate (no PASS without fresh output), standard edge-case traps (including numbers and time), silent-failure hunting, regression radar on callers of changed code, async/flaky hygiene, spotting a green suite that lies |
| `api-contract-guardian` | planner + coder + reviewer (data crossing layers) | A Contract table in the spec (database, backend validation, frontend type, nullability), enum and naming sync, ISO dates, no float money, idempotent state-changing calls, backward-compatible changes |
| `performance-profiler` | planner + coder + reviewer (lists, queries, exports) | Expected data scale stated in the spec, no N+1 or unbounded queries, pagination, cleaned-up listeners and bounded caches, streamed large files, virtualized long lists; every finding names the scale where it breaks |
| `architecture-clean-code` | coder + reviewer | Layer separation, no dead code left by the diff, rule of three for duplication, no circular imports, size norms as signals. Judges only what the diff adds; hard constraints in `CLAUDE.md` always win |
| `antislop` | designer + coder + tester (UI features) | Core filter adapted from anti-slop: a purpose test for every visual technique, 38 rules in three tiers, the Design Read with ENERGY/RHYTHM/MOTION dials, and a Delivery Gate report with evidence |
| `antislop-ui` | designer + coder + tester | Generic AI UI patterns with tell, why, and fix, including app screens: default dashboard shell, stat cards with invented numbers, filler activity feeds, charts without a question, placeholder empty states |
| `antislop-layoutmobile` | coder + tester | Reflow from phone to desktop: breakpoints, fluid scale, collapsing grids, overflow, tap targets, mobile navigation |
| `antislop-human` | coder + tester | Contrast computed with `contrast-check.py`, keyboard and focus, color-only feedback, UI states, zoom |
| `antislop-code` | coder + tester | Comment hygiene: no decorative banners, restated code, step narration, vague TODOs, or signature echoes; keeps comments that explain why. Changes comments only, never code |

Worth knowing, deliberately **not** bundled: [`differential-review`](https://github.com/trailofbits/skills) by Trail of Bits (deeper security diff review; CC BY-SA licensed, install separately rather than vendoring into an MIT repo) and [`webapp-testing`](https://github.com/anthropics/skills) by Anthropic (Playwright-driven UI testing; add per-project when browser e2e is actually needed).

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed: `/ship` is a **Claude Code slash command**, typed inside a Claude Code session in the project folder (it's not an npm script).
- Your stack's toolchain (e.g. Node.js + npm) so the tester can actually run build/lint/test.

### Start a new project

1. Copy `_pipeline-template` → rename the folder
2. Fill in `CLAUDE.md` (replace all placeholders: stack, hard constraints, verification commands, deployment target, conventions; set Deployment Target to "none" if the deliverable is handed over manually)
3. Scaffold your stack
4. Fresh git history: open Claude Code in the folder. On the first session it detects the copied template `.git` and resets it automatically (or do it yourself: delete the hidden `.git`, then `git init`). Plain `git init` alone does NOT remove the template's history/remote.
5. Open Claude Code in the folder → run `/ship [feature description]`

### Standard

What sets the output apart from instant generators: **planned, tested, and reviewed**, not just "it runs". Details in `CLAUDE.md` + each agent file.

And **fail loud, not silent**: the coder stops on a missing or ambiguous spec instead of guessing, the tester reports failures verbatim without patching over them, and the verdict is mechanical: any FAIL or NOT TESTED criterion means it cannot be SHIP.

### Credit

**Author**
- Redho Ramadhani: [linkedin.com/in/redhoramadhanihamid](https://id.linkedin.com/in/redhoramadhanihamid) · [github.com/redhoram](https://github.com/redhoram)

**Included skills adapted from others**
- `antislop` and its companion skills: adapted from [anti-slop](https://github.com/miqdadbadjuber/anti-slop) v3.2.23 by miqdadbadjuber, MIT License (`.claude/skills/antislop/LICENSE`). Changes: no install wizard or mode question, a fixed mode per agent, a theme toggle only when asked, and no em dash or emoji characters.

Built with [Claude Code](https://claude.com/claude-code).
