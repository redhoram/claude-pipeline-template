---
name: api-contract-guardian
description: Use when a feature changes a database schema, an API request or response shape, an enum, nullability, or a form that posts to an endpoint. Keeps database, backend, and frontend in sync, makes state-changing calls idempotent, and keeps changes backward compatible. The Planner writes the contract into the spec; the Reviewer verifies it against the diff.
---

# API Contract Guardian

Core belief: most full-stack bugs are not wrong logic but **two layers that disagree**:
a column the form never fills, an enum value the dropdown doesn't know, a field the
backend renamed and the frontend still reads. The contract is written once, in the spec,
and every layer is checked against it.

## 1. Contract table (Planner, in `spec.md`)

When the feature touches data that crosses layers, add a **Contract** subsection under
the spec's Data model with one row per field or endpoint:

| Field / endpoint | Database | Backend validation | Frontend type | Nullable? |
|---|---|---|---|---|
| `order_id` | `uuid not null` | `z.string().uuid()` | `orderId: string` | No |
| `status` | `text check (status in (...))` | `OrderStatus` enum | `OrderStatus` union | No |

`relational-mastery` owns the database column design; this table owns how that column
travels to the screen and back.

## 2. Sync rules

- **Names**: one documented mapping between casings (`snake_case` in the database,
  `camelCase` in the frontend, or whatever `CLAUDE.md` sets). No field silently dropped
  by the mapping.
- **Nullability both ways**: a `not null` column can't be left empty by a form; a
  nullable column is rendered safely when empty.
- **Enums from one source**: database check, backend validation, and frontend options
  derive from the same definition, or the spec lists all three as identical.
- **Dates**: ISO 8601 in UTC on the wire; converted to local time only for display.
- **Money and exact decimals**: integer minor units or a decimal type, never float.

## 3. Idempotency

- Every state-changing endpoint that a retry could repeat (create order, pay, change
  status, adjust stock) is protected by a unique constraint or an idempotency key.
- GET, PUT, and DELETE stay idempotent. POST that creates a resource guards against
  rapid double submission on the server, not only by disabling the button.

## 4. Compatibility

- Additions are optional for existing consumers.
- Renames and removals either update every consumer in the same diff, or follow a
  deprecation step (old and new both work, old is removed later). The spec says which.
- Status codes mean what they say: 200/201 success, 400 validation, 401 not logged in,
  403 not allowed, 404 not found, 409 conflict (unique violation or concurrent edit).

## 5. Reviewer check

Rebuild the contract table from the actual diff (migration, validation schema, frontend
types) and compare it with the spec. Severity for the verdict:

- Field missing in one layer, enum mismatch, nullability mismatch, float money =
  **NEEDS WORK**.
- Breaking change to an endpoint other code consumes, with no consumer update and no
  deprecation step = **NEEDS WORK**, or **BLOCK** if it corrupts or loses data.
- Non-idempotent payment or financial write = **BLOCK**.

## Done check

- [ ] Spec has a Contract table for every field that crosses layers
- [ ] Names, nullability, enums, dates, and money follow §2
- [ ] Retried state-changing calls can't run twice
- [ ] Renames and removals update all consumers or have a deprecation step
- [ ] Reviewer rebuilt the table from the diff, not from the changelog

> Credits: distilled for this pipeline from a custom API-contract checklist by the
> template author.
