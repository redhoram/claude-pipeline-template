---
name: performance-profiler
description: Use when a feature adds or changes database queries, list or table views, exports, batch jobs, file handling, or long-lived listeners and timers. Finds the problems that only appear at real data volume: N+1 queries, unbounded queries, missing pagination, memory leaks, and heavy rendering. The Planner states the expected scale; the Reviewer checks the diff against it.
---

# Performance Profiler

Core belief: code that is fast on ten test rows can fall over at ten thousand. Every
finding names the **scale at which it breaks**, so the fix is sized to the real risk,
not to taste.

## 1. Expected scale (Planner, in `spec.md` Context)

For any feature that reads lists or grows with usage, write the expected size: rows per
table after a year, items per page, concurrent users, largest file. If nobody knows and
it changes the design (pagination vs not), it is an Open question; otherwise state the
assumption.

## 2. Queries

- **N+1**: a loop that runs one query per parent row. Load related rows in one query
  (join, `include`, `select_related`, batch `IN (...)`).
- **Unbounded reads**: every list endpoint has a default and a maximum page size. No
  `SELECT *` without a limit on a table that grows with users.
- **Uses an index**: columns in `WHERE`, `JOIN`, `ORDER BY` have one. `relational-mastery`
  designs the index; this skill checks the query actually hits it.
- **Projection**: select the columns the screen needs, not large text or JSON blobs it
  never shows.

## 3. Memory and resources

- Event listeners, subscriptions, sockets, and timers are removed when the component
  unmounts or the session ends.
- In-memory caches have a size limit and an expiry.
- Large uploads, exports, and file transfers stream instead of loading the whole payload
  into memory.

## 4. Frontend

- Long lists are paginated or virtualized. The threshold comes from `CLAUDE.md` if set;
  otherwise treat about 100 rendered rows as the point to act.
- No re-render storms from new object or function props on every render.
- No whole-library imports for one helper.

## 5. Reviewer check

Each finding states: what is slow, at what scale it fails (from §1), and the fix.
Severity for the verdict:

- N+1 or unbounded query on a table that grows with users, or a leak in a long-lived
  screen = **NEEDS WORK**.
- Problems that only appear far beyond the scale in the spec = note for the user, not a
  blocker.

## Done check

- [ ] Spec states the expected data scale
- [ ] No N+1 and no unbounded list query on growing tables
- [ ] Queries filter and sort on indexed columns and select only needed columns
- [ ] Listeners and timers cleaned up; caches bounded; large files streamed
- [ ] Long lists paginated or virtualized
- [ ] Every finding names the scale at which it breaks

> Credits: distilled for this pipeline from a custom performance checklist by the
> template author.
