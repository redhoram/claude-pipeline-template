---
description: Runs a lighter Coder → Tester → Reviewer pass to revise a feature that already exists, with a short spec written by the main session.
---

User's change request: $ARGUMENTS

Use `/fix` to revise something already built: a rename, a copy change, a layout tweak, a bug fix, a small behavior change. It skips the planner and designer but still leaves a spec, a test report, and a verdict behind, so every revision is auditable.

Use `/ship` instead when the request adds a new screen or feature, changes persisted data, adds a dependency, or changes the visual language. If you discover this mid-way, stop and tell the user to run `/ship`.

Run this sequence IN ORDER:

0. **Archive the workspace** exactly as in step 0 of `.claude/commands/ship.md`, including the **Model override** rule.
1. **Write the spec yourself** to `.pipeline/spec.md`, in the planner's format but kept short. Open the files you name; never guess a path.
   - **Goal**: one sentence.
   - **Context**: the files to change (opened, named), the existing patterns to keep, and the relevant rules from `CLAUDE.md` (Hard constraints, Final files, Content sources).
   - **Scope** and **Out of scope**. Out of scope always includes "every screen and file not named in Scope".
   - **Data model**: exactly "No data changes". If data changes, stop: this is a `/ship`.
   - **Success criteria**: a `- [ ]` checklist of individually verifiable statements. For UI changes, include the visual result at the widths in `CLAUDE.md` Verification, and that old labels or text no longer appear anywhere.
   - **Task breakdown**, **Edge cases**.
   - **Open questions**: "None", or the questions whose answers change what gets built.
2. **Open-questions gate**: if Open questions is not "None", STOP and ask the user. Write the answers back into the spec, then continue.
3. **Mark design as skipped**: write `.pipeline/design.md` with one line: "Design skipped: revision via /fix. The existing UI is the design reference; keep its visual language."
4. Call the **coder** subagent. Wait until `.pipeline/changelog.md` is written.
5. Call the **tester** subagent. Wait until `.pipeline/test-report.md` is written.
6. Call the **reviewer** subagent for the final verdict.
7. Show the verdict and handle NEEDS WORK, BLOCK, and SHIP exactly as in step 7 of `.claude/commands/ship.md` (commit offer included; deployer only if the change is operational and Deployment Target is not "none").

The **Fatal = stop the chain** rules at the end of `.claude/commands/ship.md` apply here too: a missing output file, a `## BLOCKED` section in the changelog, or a reported broken workspace stops the chain, and you show the user what is blocked and wait.
