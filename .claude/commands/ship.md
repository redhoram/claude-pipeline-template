---
description: Runs the Planner → Designer → Coder → Tester → Reviewer pipeline for one feature end-to-end.
---

User's feature request: $ARGUMENTS

Run this sequence IN ORDER, don't skip:

0. **Archive the workspace**: create `.pipeline/` if it doesn't exist. If it holds artifacts from a previous run (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`, `deploy-report.md`), MOVE them into `.pipeline/archive/<YYYYMMDD-HHMM>/` so a previous run is never read by mistake and the audit trail survives. Do NOT touch `.gitkeep` or existing archive folders.
   **Model override**: read the `## Model override` table in `CLAUDE.md`. Whenever you call a subagent listed there, pass that model when spawning it. Agents not listed use the model in their own file.
1. Call the **planner** subagent with the feature request above. Wait until `.pipeline/spec.md` is written.
2. **Open-questions gate**: read the spec's **Open questions** section. If it says anything other than "None", STOP and ask the user every question. Write the answers back into `spec.md` (replace the Open questions section with the decisions made), then continue. Never let the designer or coder inherit an unresolved question.
3. Call the **designer** subagent to produce a design spec from the spec. Wait until `.pipeline/design.md` is written. (If the feature has no UI surface, the designer will mark design as skipped — continue to the next step anyway.)
4. Call the **coder** subagent to read spec + design and write the code. Wait until `.pipeline/changelog.md` is written.
5. Call the **tester** subagent to check the result. Wait until `.pipeline/test-report.md` is written.
6. Call the **reviewer** subagent for the final verdict.
7. Show the final verdict (SHIP / NEEDS WORK / BLOCK) to the user with a summary of what happened at each stage.
   - If **NEEDS WORK**: offer to re-run from **coder** (it reads `.pipeline/verdict.md` as the fix list) → tester → reviewer. Don't loop automatically without user confirmation.
   - If **BLOCK**: stop completely, explain the problem to the user, don't proceed.
   - If **SHIP**:
     1. Offer to commit. Stage ONLY the files listed in `.pipeline/changelog.md`, by explicit path, never `git add -A`. Use the author and rules in the `Git` line of `CLAUDE.md` Conventions. Never push without the user's explicit confirmation.
     2. If `CLAUDE.md` Deployment Target is anything other than "none", offer to call the **deployer** subagent to prepare the change for production (env-var inventory, infra config, `.pipeline/deploy-report.md`). It requires the verdict to be SHIP — it will refuse otherwise. Skip it when the user deploys manually or nothing operational changed.

If any subagent reports a fatal problem mid-way (missing spec, unresolved questions, broken workspace), stop the pipeline and report to the user — don't force the next stage.
