---
name: relational-mastery
description: Use when a feature adds or changes persisted data — the Planner's method for the spec's Data model section. Normalization, constraints with explicit lifecycles, indexing strategy, and ORM-ready schema output.
---

# Relational Mastery

You are designing the data layer. Schema designs must prioritize data integrity, query performance, and strict typing — an ambiguity in the Data model section becomes a bug in the Coder's migration.

## Core Directives

1. **Normalization (3NF)**: normalize to at least Third Normal Form unless deliberate denormalization is required for a specific, justified performance reason — and then write the justification into the spec.
2. **Data Integrity (Constraints)**:
   - Always define Foreign Key constraints.
   - Define strict `ON DELETE` and `ON UPDATE` behaviors (`CASCADE`, `SET NULL`, `RESTRICT`) — an unspecified lifecycle is a decision the Coder shouldn't be making.
   - `NOT NULL` by default; nullable columns must be logically justified.
   - Apply `UNIQUE` constraints where appropriate to prevent duplicate entries.
3. **Indexing Strategy**:
   - Primary Keys are non-negotiable. Prefer UUIDs or strongly-typed sequences.
   - Add indexes on foreign keys and columns the feature's queries will filter or join on.
   - Avoid over-indexing (it slows down writes) — each index names the query it serves.
4. **Auditability**: include `created_at` and `updated_at` timestamps on mutation-heavy tables.
5. **Ownership/tenancy**: every table carrying user data names its ownership column (`user_id`, `organization_id`, …) per `CLAUDE.md` rules — this is what the `secure-coding` authz checks verify against later.

## Execution Checklist

- [ ] All tables have a clearly defined Primary Key
- [ ] Foreign Key relations map parent-child lifecycles with explicit ON DELETE/UPDATE
- [ ] Indexes proposed for the feature's actual query patterns, each justified
- [ ] Naming conventions consistent (e.g. snake_case for tables/columns)
- [ ] Schema written in the exact format the project's ORM needs (e.g. Prisma, Drizzle) so the Coder implements it verbatim
- [ ] Ownership/tenant column named on every user-data table

> Credits: adapted from the `relational-mastery` skill in
> [antigravity-pipeline-template](https://github.com/redhoram/antigravity-pipeline-template) (same author, MIT).
