# _pipeline-template

> 🇬🇧 [English version](./README.md)

Template framework untuk **aplikasi dengan CRUD / logika kompleks** — fitur berlapis, backend + frontend, butuh quality gate sebelum rilis. Saudara dari `_landing-template` (yang buat landing page & app sederhana), tapi dioptimalkan untuk kerja fitur yang terstruktur.

## Filosofi: pipeline batch dengan quality gate

Fitur kompleks butuh perencanaan, implementasi, pengujian, dan review yang terpisah jelas — biar tiap tahap bisa diaudit dan nggak ada yang kelewat. Alurnya doc-driven: tiap subagent nulis artefak ke `.pipeline/`, lalu di-handoff ke tahap berikutnya.

- **`/ship [deskripsi fitur]`**: jalanin pipeline 5 subagent end-to-end — planner → designer → coder → tester → reviewer. Reviewer kasih verdict akhir (SHIP / NEEDS WORK / BLOCK).
- Untuk fix kecil/tweak: nggak wajib `/ship` — langsung minta Claude Code biasa.

## Beda dari `_landing-template`

| | `_pipeline-template` (CRUD kompleks) | `_landing-template` (landing & app sederhana) |
|---|---|---|
| Mode | Batch, doc-driven, fire-and-forget | Loop visual + interaktif |
| Agent | planner → designer → coder → tester → reviewer | designer → builder → qa |
| Gate akhir | Reviewer (SHIP/BLOCK, fokus keamanan) | QA (punch-list, fokus UI/UX) |
| Command | `/ship` | `/craft` (0→1) + `/polish` (1→bagus) |
| Verifikasi | build/lint/test + skenario | live preview: render, responsif, console |

## Isi

- `CLAUDE.md` — instruksi project: stack, konvensi, alur pipeline. **Isi placeholder-nya dulu.**
- `.claude/agents/` — `planner`, `designer`, `coder`, `tester`, `reviewer`
- `.claude/commands/` — `ship`
- `.pipeline/` — workspace handoff (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`)

## Bikin project baru

1. Copy `_pipeline-template` → rename folder
2. Isi `CLAUDE.md` (ganti semua placeholder: stack, konvensi, aturan project)
3. Scaffold stack-mu
4. `git init`

## Standar

Yang bikin output beda dari generator instan: **terencana, teruji, dan di-review** — bukan sekadar jalan. Detail di `CLAUDE.md` + tiap file agent.
