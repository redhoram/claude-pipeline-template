---
name: systematic-debugging
description: Use when a fix round starts (verdict NEEDS WORK), a test fails, or code behaves unexpectedly. Four-phase root-cause process that forbids guess-fixes, limits thrashing with a 3-strikes rule, and escalates loudly instead of patching symptoms.
---

# Systematic Debugging

The Iron Law: **NO FIX WITHOUT ROOT CAUSE INVESTIGATION FIRST.**

A fix written before the cause is understood is a guess. Guesses stack: each one
"almost works", mutates the code, and buries the real cause deeper. This process
is faster than thrashing — especially under time pressure, especially when the fix
looks "obviously simple", and especially after a previous fix already failed.

## When this applies

- The Reviewer returned **NEEDS WORK** and you are the Coder on a fix round.
- Any test failure, build failure, or behavior that contradicts the spec.
- NOT for writing new features — this is for making wrong things right.

## Phase 1 — Investigate (no code changes allowed yet)

1. **Read the error completely.** Full message, full stack trace, file:line.
   The answer is usually in text you were about to skip.
2. **Reproduce reliably.** Exact command/steps that trigger it every time.
   Can't reproduce → you don't understand it yet; gather more data, don't fix.
3. **Check what changed.** `git diff`, recent commits, new dependencies, config
   or env differences. Most bugs live in the newest change.
4. **Trace the data.** Find where the bad value is BORN, not where it crashes.
   Follow the call chain backward until the first wrong value appears.
5. Multi-layer systems (UI → API → DB): log the value at each boundary and find
   the exact layer where it goes wrong — never assume the layer.

## Phase 2 — Compare

- Find working code in this same codebase that does something similar.
  List EVERY difference against the broken path — no difference is "too minor".
- If a reference implementation or doc exists, read the relevant part fully;
  a half-read pattern applied blind creates the next bug.

## Phase 3 — Hypothesize (single, written, falsifiable)

- Write one sentence: "I believe X causes this because Y."
- Test it with the SMALLEST possible probe — one variable at a time, a log line
  or a minimal change, not a rewrite.
- Hypothesis wrong → discard the probe completely, write a new hypothesis.
  Never leave failed-experiment code in place while trying the next idea.

## Phase 4 — Fix

1. Write (or identify) the failing test/reproduction FIRST — proof you found it.
2. Apply ONE fix at the root cause. No drive-by refactoring in the same change.
3. Re-run: the failing case now passes AND the rest of the suite still passes.
4. State the evidence in one line: command run + result ("`npm test` — 42 pass, 0 fail").

## The 3-strikes rule (anti-thrash, fail loud)

After **3 failed fix attempts** on the same problem: STOP. The problem is almost
certainly architectural, or the spec conflicts with reality. Do not attempt fix #4.
In this pipeline that means: write what you tried and what you learned into
`.pipeline/changelog.md` under **Deviations**, and report back as blocked — the
Reviewer/user decides the next move. A documented stop beats a silent hack.

## Red flags — you are about to violate the Iron Law

- "Quick fix for now, investigate later"
- "It's probably X" without evidence X is reachable in the failing path
- Changing several things, then re-running to "see if it helps"
- Fixing where the error APPEARS instead of where the bad value ORIGINATES
- Deleting/relaxing an assertion to make a test pass
- Fix #2 revealing a new error in a different place — that's a strike, not progress

| Rationalization | Reality |
|---|---|
| "Bug is trivial, process is overkill" | Trivial bugs have root causes too — Phase 1 takes 2 minutes |
| "No time for systematic work" | Thrashing takes longer; guessing under pressure compounds it |
| "One more attempt will do it" | After 3 strikes, attempt #4 is how codebases rot |
| "Test after the fix works" | Untested fixes regress silently the next round |

## Done check

- [ ] Root cause stated in one sentence, with the evidence that proves it
- [ ] Failing reproduction existed BEFORE the fix; passes after
- [ ] Exactly one change, at the origin of the bad value — not at the symptom
- [ ] Full suite still green; evidence line quoted
- [ ] 3 failed attempts → stopped, documented in changelog Deviations, reported blocked

> Credits: adapted for this pipeline from `systematic-debugging` in
> [obra/superpowers](https://github.com/obra/superpowers) (MIT).
