---
name: ux-research
description: Use when writing a design spec or verifying a built UI. Behavioral grounding for the Designer — who the user is, which psychology principles apply to this surface, ethics hard-lines — and a 10-question usability diagnostic with severity ratings for the Tester.
---

# UX Research

Core belief: `premium-design` decides whether the UI **looks** right; this skill decides
whether it **works for a human**. Every flow, hierarchy, and copy decision must trace to
a reason about the user — "looks clean" is not a reason. A beautiful page that confuses
users is a failed page.

## 1. Ground the design first (Designer, before any visual decision)

Answer in 2-3 sentences at the top of `design.md`, derived from `spec.md` + `CLAUDE.md` —
NOT from an invented persona document:

- **Who** uses this surface, and **what job** are they hiring it to do (JTBD)?
- **Context of use**: device, frequency (daily tool vs one-time visit), urgency,
  and what the user was doing right before arriving here.
- If the spec doesn't say who the user is and it changes the design → that's an
  open question for the pipeline gate, not a guess.

## 2. Principle picker — 3-5 per surface, named in the spec

Pick ONLY what fits this surface type; list them in `design.md` under
**Principles used**, each with WHERE it's applied. Unnamed psychology = decoration.

| Surface | Reach for these first |
|---|---|
| Landing / marketing | Visual hierarchy (one focal point per viewport), Hick's law (ONE primary CTA), social proof near the ask, loss-framing in CTA copy only when true, Von Restorff (the single accent earns attention) |
| Pricing | Anchoring (order & reference price), chunking of plan differences, cognitive-load cut (compare ≤4 things), transparency beats decoy tricks |
| Onboarding / forms | Progressive disclosure (ask only what this step needs), chunking (groups of 3-5 fields), goal gradient (show progress), immediate inline validation, error prevention over error messages |
| Dashboard / product | Recognition over recall (visible options, not memorized commands), consistency with existing patterns, status visibility (every async action shows state), Fitts's law (frequent targets big & close), calm defaults — no persuasion theater in a work tool |
| Checkout / critical flows | Cognitive load minimum (guest path, autofill), error prevention + forgiveness (review step, undo, edit-in-place), trust signals at the moment of commitment, no surprise costs late in the flow |

## 3. Usability heuristics — design-time checklist (Designer)

Condensed from Nielsen's 10 + Krug; check while writing the section breakdown:

1. **Visibility of status** — every action has visible feedback within 100ms (even "saving…").
2. **Match the user's language** — labels use the user's words, not the schema's (`"Team"` not `"organization_id"`).
3. **User control** — back/cancel/undo exists; destructive actions are reversible or confirmed.
4. **Consistency** — same action, same place, same name everywhere; follow platform conventions.
5. **Error prevention** — constrain inputs (pickers, formats shown), disable invalid submits, confirm irreversibles.
6. **Recognition over recall** — options visible; nothing requires remembering a previous screen.
7. **Self-evident navigation** — on any screen the user can answer: where am I, what can I do here, where's the primary action? (Krug's trunk test)
8. **Minimalism with a job** — every element competes for attention; cut what doesn't serve the JTBD.
9. **Errors that help** — plain language, say what happened, offer the fix ("Email already registered — log in instead?").
10. **No manual needed** — if it needs explanation, redesign before documenting.

## 4. Ethics hard-lines (violating any = design.md is rejected)

- No fake scarcity or countdown timers that reset.
- No confirmshaming ("No thanks, I hate saving money").
- No dark defaults: opt-ins are unchecked; the cheaper/lighter choice is never hidden.
- No roach motel: leaving (cancel, unsubscribe, delete account) is as easy as entering.
- Persuasion serves the USER's goal; if a principle only serves conversion at the
  user's expense, don't use it. HR/people-facing products doubly so.

## 5. Tester quick diagnostic (run while driving the built UI)

Answer these 10 while walking the primary flow (see `testing-discipline` §5).
Each "no" becomes a finding: **question + severity + one-line evidence** in `test-report.md`.

1. Within 5 seconds on each screen: is it obvious what this page is for?
2. Is the primary action visually the #1 thing on the page?
3. Does every click/submit give feedback (loading, success, error)?
4. Can you tell where you are in the app / in a multi-step flow?
5. Can you back out of every step without losing work?
6. Do form errors appear inline, near the field, in plain language?
7. Is anything essential hidden behind hover-only or memory of a previous screen?
8. Are destructive actions guarded (confirm/undo) and non-destructive ones NOT over-guarded?
9. Empty states: does a first-time user see guidance, not a blank void?
10. Any §4 ethics violation on screen?

Severity scale (report it): **0** cosmetic · **1** minor annoyance · **2** hurts task completion ·
**3** user likely fails the task · **4** catastrophic/ethics — a 3+ finding should make the
Reviewer think NEEDS WORK regardless of green tests.

## Done check

**Designer:**
- [ ] design.md opens with who + JTBD + context of use (from spec, not invented)
- [ ] **Principles used** section: 3-5 principles, each mapped to a concrete section
- [ ] §3 checklist passed on the section breakdown; flows need zero explanation
- [ ] Zero §4 violations

**Tester:**
- [ ] 10-question diagnostic run on the driven UI, findings with severity in test-report
- [ ] Severity 3+ findings explicitly flagged for the Reviewer

> Credits: adapted for this pipeline from [Nuclear-Marmalade/ux-psychology-skill](https://github.com/Nuclear-Marmalade/ux-psychology-skill)
> (MIT). Heuristics condensed in our own words from the public methodology of
> Nielsen Norman Group (10 Usability Heuristics) and Steve Krug's *Don't Make Me Think*.
