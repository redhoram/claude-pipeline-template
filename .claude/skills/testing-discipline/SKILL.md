---
name: testing-discipline
description: Use when verifying a feature against its success criteria — the Tester's method. Evidence-gated claims (no PASS without fresh command output), edge-case hunting, async/flaky test hygiene, and how to spot a lying green test suite.
---

# Testing Discipline

The Iron Law: **NO CLAIM WITHOUT FRESH EVIDENCE.** "Should work", "looks correct",
"probably passes" do not exist in a test report. Either you ran it and read the
output, or the criterion is NOT TESTED — and saying so honestly is a passing grade
for the Tester, while a bluffed PASS is the one unforgivable failure.

## 1. The evidence gate (before writing PASS on anything)

1. **Identify** the exact command or action that proves the claim.
2. **Run** it fresh — not "it passed earlier", not a cached result.
3. **Read** the full output and exit code, not just the last line.
4. **Compare**: does the output actually prove THIS criterion, or something adjacent?
5. Only then write the verdict, quoting the decisive line.

| Claim | Required evidence |
|---|---|
| Tests pass | Test-runner output: N passed, 0 failed |
| Build succeeds | Build command exit 0, no error lines |
| Lint clean | Linter output: 0 errors |
| Endpoint works | Actual request sent + status/body observed (curl, test, or driven UI) |
| Bug fixed | The ORIGINAL failing case, re-run, now passing |
| UI renders | Page actually loaded/driven — reading the JSX is NOT evidence |

## 2. Edge-case hunt (run after the happy path)

The spec lists edge cases — verify those first, then probe these standard traps
for every input surface the feature added:

- **Empty & missing**: `""`, `null`/`undefined`, empty array, missing field, no rows.
- **Boundaries**: 0, -1, 1, exactly-at-limit, one-over-limit, page 2 of 1-page data.
- **Oversized**: very long strings, huge numbers, big file, 1000-row list.
- **Wrong type/shape**: string where number expected, malformed JSON, invalid enum.
- **Hostile**: `'; DROP TABLE--`, `<script>alert(1)</script>`, `../../etc/passwd`
  in any text field — expect escaping/rejection, not execution (see `secure-coding`).
- **Duplicate & repeat**: same request twice (double-submit), unique-constraint hits.
- **Unauthorized**: the same actions logged out, and as a DIFFERENT user — foreign
  IDs must yield 403/404, never data.
- **Real-world text**: unicode, emoji, `O'Brien`, leading/trailing whitespace.

Report each probe with input → observed behavior, one line each.

## 3. Async & flaky hygiene

- A test that needs `sleep(2000)` to pass is guessing. Wait for the CONDITION,
  not the clock:

```ts
async function waitFor<T>(cond: () => T | false | null | undefined,
                          what: string, timeoutMs = 5000): Promise<T> {
  const t0 = Date.now();
  for (;;) {
    const r = cond();
    if (r) return r;
    if (Date.now() - t0 > timeoutMs) throw new Error(`Timeout waiting for ${what}`);
    await new Promise(res => setTimeout(res, 10));
  }
}
```

- Every wait gets a timeout AND a named description, so a hang fails with a reason.
- A test that passes on retry is a FAIL with extra steps — report the flake, don't
  re-run until green and call it PASS.
- Fixed delays are legitimate ONLY when timing IS the feature (debounce, throttle) —
  and then the test says so.

## 4. Reading the Coder's tests — green can lie

Before trusting a passing suite, spot-check for these anti-patterns:

- **Mock-assertions**: the test verifies a mock exists/was called, but never the
  real behavior (`expect(mockFn).toHaveBeenCalled()` as the ONLY assertion).
- **Hollow mocks**: mocked responses missing fields the real API returns — the
  code under test never sees realistic data.
- **Tautologies**: asserting a constant equals itself, snapshot-everything tests,
  `expect(true).toBe(true)` padding.
- **Test-only production code**: methods added to production classes purely so
  tests can call them.
- **Deleted teeth**: assertions commented out or loosened (`toBeGreaterThan(0)`
  where an exact value was checkable).

A green suite with these patterns proves nothing — note it in the report for the
Reviewer; do NOT count it as evidence.

## 5. UI verification (features with a UI surface)

- If the project has a browser-driving tool available (Playwright, the Claude Code
  browser, `npm run dev` + curl for API-backed pages), DRIVE the primary flow:
  load the page, perform the user action, observe the result.
- No driving tool available → the flow goes under **Not tested** with the reason.
  Reading component code and declaring it works is forbidden (§1 table, last row).
- While driving, also watch the console for errors — a rendered page with a red
  console is a finding, not a pass.

## Done check

- [ ] Every criterion verdict has a quoted evidence line (command + decisive output)
- [ ] Spec edge cases + §2 standard traps probed on new input surfaces
- [ ] Zero PASS verdicts based on reading code or prior runs
- [ ] Flakes reported as failures, not retried into PASS
- [ ] Coder's test suite spot-checked for §4 anti-patterns
- [ ] UI flow driven for real, or honestly listed under Not tested

> Credits: adapted for this pipeline from `verification-before-completion`,
> `testing-anti-patterns`, and `condition-based-waiting` in
> [obra/superpowers](https://github.com/obra/superpowers) (MIT); UI-driving guidance
> inspired by `webapp-testing` in [anthropics/skills](https://github.com/anthropics/skills) (Apache 2.0).
