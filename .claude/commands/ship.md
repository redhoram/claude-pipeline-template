---
description: Runs the Planner → Designer → Coder → Tester → Reviewer pipeline for one feature end-to-end.
---

User's feature request: $ARGUMENTS

Run this sequence IN ORDER, don't skip:

0. **Clean the workspace**: create `.pipeline/` if it doesn't exist; delete old files in it (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`) if present, so a previous run isn't read by mistake. Do NOT delete `.gitkeep`.
1. Call the **planner** subagent with the feature request above. Wait until `.pipeline/spec.md` is written.
2. **Open-questions gate**: read the spec's **Open questions** section. If it says anything other than "None", STOP and ask the user every question. Write the answers back into `spec.md` (replace the Open questions section with the decisions made), then continue. Never let the designer or coder inherit an unresolved question.
3. Call the **designer** subagent to produce a design spec from the spec. Wait until `.pipeline/design.md` is written. (If the feature has no UI surface, the designer will mark design as skipped — continue to the next step anyway.)
4. Call the **coder** subagent to read spec + design and write the code. Wait until `.pipeline/changelog.md` is written.
5. Call the **tester** subagent to check the result. Wait until `.pipeline/test-report.md` is written.
6. Call the **reviewer** subagent for the final verdict.
7. Show the final verdict (SHIP / NEEDS WORK / BLOCK) to the user with a summary of what happened at each stage.
   - If **NEEDS WORK**: offer to re-run from **coder** (it reads `.pipeline/verdict.md` as the fix list) → tester → reviewer. Don't loop automatically without user confirmation.
   - If **BLOCK**: stop completely, explain the problem to the user, don't proceed.

If any subagent reports a fatal problem mid-way (missing spec, unresolved questions, broken workspace), stop the pipeline and report to the user — don't force the next stage.
