---
description: Runs the Planner → Drafter → Coder → Tester → Gatekeeper pipeline for one feature end-to-end.
---

User's feature request: $ARGUMENTS

You are the Engineering Manager. Run this sequence IN ORDER, don't skip:

0. **Archive the workspace**: create `.pipeline/` if it doesn't exist. If it holds artifacts from a previous run (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`, `deploy-report.md`, `shots/`), MOVE them into `.pipeline/archive/<YYYYMMDD-HHMM>/` so a previous run is never read by mistake and the audit trail survives. Do NOT touch `.gitkeep` or existing archive folders.
   **Model override**: read the `## Model override` table in `CLAUDE.md`. Whenever you call a subagent listed there, pass that model when spawning it. Agents not listed use the model in their own file.
1. Call the **planner** subagent with the feature request above. Wait until `.pipeline/spec.md` is written.
2. **Open-questions gate**: read the spec's **Open questions** section. If it says anything other than "None", STOP and ask the user every question. Write each answer into `spec.md` Context under **Decisions from the user** (question, answer, date). Then rewrite every part of the spec the planner marked as pending that question (provisional values, success criteria, tasks) so it states the answer, leaving no stale placeholder behind. Then set the Open questions section to exactly "None" so the coder can start. When a question came from a `CLAUDE.md` placeholder, also offer to write the answer into `CLAUDE.md` so the next run doesn't ask again. Continue. Never let the drafter or coder inherit an unresolved question.
3. Call the **drafter** subagent to produce a design spec from the spec. Wait until `.pipeline/design.md` is written. (If the feature has no UI surface, the drafter writes `STATUS: SKIPPED`: continue to the next step anyway. Questions the drafter answered with a default are listed in design.md; show them in the step 7 summary, don't stop for them.)
   If design.md starts with `STATUS: BLOCKED`, ask the user the question, write the answer into spec.md Context under **Decisions from the user**, and re-run the drafter only. Never edit design.md's STATUS line by hand.
4. Call the **coder** subagent to read spec + design and write the code. Wait until `.pipeline/changelog.md` is written.
5. Call the **tester** subagent to check the result. Wait until `.pipeline/test-report.md` is written.
6. Call the **gatekeeper** subagent for the final verdict.
7. Show the final verdict (SHIP / NEEDS WORK / BLOCK) to the user with a summary of what happened at each stage.
   - If **NEEDS WORK**: offer to re-run from **coder** (it reads `.pipeline/verdict.md` as the fix list) → tester → gatekeeper. When every item on the fix list is addressed to the tester (a coverage gap, no code change), offer to re-run tester → gatekeeper only. Before any re-run, COPY (don't move: the coder and tester read it) the current `verdict.md`, `test-report.md`, and `changelog.md` into `.pipeline/archive/<YYYYMMDD-HHMM>-round/`, so the earlier round survives being overwritten. Don't loop automatically without user confirmation.
   - If **BLOCK**: stop completely, explain the problem to the user, don't proceed.
   - If **SHIP**:
     1. Offer to commit. Stage ONLY the files listed in `.pipeline/changelog.md`, by explicit path, never `git add -A`. Use the `Git author` and `Git` lines of `CLAUDE.md` Conventions; when `Git author` is still in [brackets], commit as the repo's configured git user and say so. Never push without the user's explicit confirmation.
     2. If `CLAUDE.md` Deployment Target is filled in and is not "none" (a Target still in [brackets] counts as "none"), offer to call the **deployer** subagent to prepare the change for production (env-var inventory and proposed infra config, all in `.pipeline/deploy-report.md`; it writes no other file). It requires the verdict to be SHIP: it will refuse otherwise. Afterwards run `git status`: show the user any change besides `.pipeline/deploy-report.md` as not reviewed by the gatekeeper, and never stage it. Skip it when the user deploys manually or nothing operational changed.

**Fatal = stop the chain.** After each stage, check its output before calling the next one. Fatal means any of:
- the stage finished without writing its output file (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`);
- the spec's Open questions are still unresolved after the gate in step 2;
- `.pipeline/design.md` starts with `STATUS: BLOCKED`, or its first line is not `STATUS: READY`, `STATUS: SKIPPED`, or `STATUS: BLOCKED`;
- `.pipeline/changelog.md` contains a `## BLOCKED` section (the coder's build gate failed);
- a subagent reports a broken workspace or a missing precondition.

On any of these: stop, show the user exactly what is blocked and why, and wait for their call. Don't improvise around it, and don't re-run the chain hoping it fixes itself.
