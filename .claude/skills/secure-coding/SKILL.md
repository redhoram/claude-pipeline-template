---
name: secure-coding
description: Use when writing or reviewing any code that touches user input, auth, data access, secrets, or AI/LLM calls. Security checklist for the Coder (while writing) and the Reviewer (while auditing the diff) — injection, authz, secrets, fail-closed errors, and LLM-specific risks.
---

# Secure Coding

Core belief: security is a **default posture, not a feature ticket**. The cheapest
vulnerability is the one never written. The Coder applies this while writing;
the Reviewer audits the diff against the same list — one source of truth, no drift.

## 0. Non-negotiables (apply to every change)

- No secret ever enters the repo: no API keys, tokens, passwords, or connection
  strings in code, comments, tests, fixtures, or `.env*` committed to git.
- Every external input is untrusted: request bodies, query params, headers,
  cookies, file uploads, webhook payloads, LLM output.
- Validate on the **server**. Client-side validation is UX, not security.
- Deny by default: no route, query, or tool call is public unless the spec says so.
- Fail **closed**: an error in an auth/permission check means "no", never "yes".

## 1. Injection — parameterize everything

- SQL: parameterized queries / ORM bindings only. String-built SQL is a finding,
  even in a one-off script.
- XSS: escape at render; never write user input into `innerHTML`,
  `dangerouslySetInnerHTML`, or template strings that become HTML. If rich text is
  required, sanitize with an allowlist library — not a hand-rolled regex.
- Command/path: never interpolate user input into shell commands or file paths.
  Use argument arrays; resolve paths and verify they stay inside the intended root.
- Uploads: validate type and size server-side; store outside the web root or in
  object storage; never execute or `include` an uploaded file.

## 2. Authentication

- Passwords: hash with Argon2 or bcrypt — never MD5/SHA-x, never reversible, never plain.
- Sessions/tokens: framework- or platform-issued (e.g. Supabase Auth, NextAuth) —
  don't hand-roll session IDs or JWT signing unless the spec demands it.
- Cookies carrying sessions: `HttpOnly`, `Secure`, `SameSite` set deliberately.
- Rate-limit login, signup, password-reset, and OTP endpoints.

## 3. Authorization — the #1 real-world hole

- **Ownership check on every row**: `WHERE id = ?` is broken until it is
  `WHERE id = ? AND user_id = ?` (or the RLS policy proves the same thing).
  An attacker incrementing IDs must get 404/403, not data (IDOR).
- Multi-tenant projects: every table access is scoped by the tenant key
  (e.g. `organization_id`) — via RLS policy or query filter, per `CLAUDE.md` rules.
  A missing tenant filter is a BLOCK, not a nit.
- Authorize **server-side per request** — hiding a button is not authorization.
- Check the verb too: a user allowed to read a resource is not thereby allowed
  to update or delete it.

## 4. Secrets & data protection

- Secrets live in env vars; ship a `.env.example` with placeholder values only.
- Log lines never contain passwords, tokens, full PII, or raw request bodies from
  auth endpoints.
- Collect the minimum personal data the feature needs; the spec's fields are the cap.
- TLS/HTTPS assumptions: never disable certificate verification "temporarily".

## 5. Errors — fail closed, leak nothing

- User-facing errors are generic ("Something went wrong"); details go to server logs.
- No stack traces, SQL fragments, or internal paths in HTTP responses.
- A `try/catch` around an auth or payment check must re-throw or deny — swallowing
  the error and continuing is a vulnerability, not resilience.

## 6. AI / LLM features (when the project calls a model)

- **Prompt injection**: keep a hard boundary between instructions and untrusted
  content — user text, fetched pages, and file contents are data, never merged into
  the system prompt as instructions.
- **LLM output is untrusted input**: validate/escape it before rendering it as HTML,
  executing it, or passing it to tools/SQL.
- **Least-privilege tools**: an agent/tool gets only the scopes the feature needs;
  destructive actions require an explicit user approval step.
- **Cost guards**: cap tokens, add timeouts, rate-limit model-calling endpoints —
  an unmetered LLM endpoint is a wallet-drain vulnerability.

## 7. Reviewer diff-scan (run these, in order)

1. `git diff` for secrets: grep the diff for `api[_-]?key`, `secret`, `token`,
   `password`, `Bearer `, `sk-`, long base64/hex literals. Check `git status` for
   `.env*` files staged.
2. Grep new/changed queries for string concatenation or template literals feeding
   SQL; confirm parameterization.
3. For every new route/handler/server action in the diff, name the line where
   authn is checked and the line where **ownership/tenant** is enforced. Can't
   point to it → NEEDS WORK.
4. Grep for `dangerouslySetInnerHTML`, `innerHTML`, `eval(`, `exec(`,
   `child_process`, `verify: false`, `rejectUnauthorized: false`.
5. Errors in new `catch` blocks: confirm nothing sensitive is returned to the
   client and auth failures deny.
6. If the diff touches LLM calls: check §6 items one by one.

Severity mapping for the verdict: exposed secret, missing authz/tenant check,
or injection path = **BLOCK**. Missing rate limit, verbose errors, over-broad
logging = **NEEDS WORK**.

## Done check

- [ ] Zero secrets in code or git; `.env.example` only
- [ ] All external input validated server-side; queries parameterized
- [ ] Every new endpoint: authn + ownership/tenant check you can point to
- [ ] Errors fail closed and leak no internals
- [ ] LLM calls (if any): injection boundary, output treated as untrusted, tools least-privilege, cost caps
- [ ] Reviewer ran §7 greps on the actual diff — not from memory

> Credits: adapted for this pipeline from [agamm/claude-code-owasp](https://github.com/agamm/claude-code-owasp)
> (MIT) and the OWASP Top 10:2025 / LLM Top 10 guidance it distills.
