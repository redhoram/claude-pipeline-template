<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-Template-D97757?logo=anthropic&logoColor=white" alt="Claude Code Template">
  <img src="https://img.shields.io/badge/Pipeline-6%20Agents-informational" alt="6-Agent Pipeline">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License">
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.id.md">Bahasa Indonesia</a>
</p>

---

# _pipeline-template

Template framework untuk **aplikasi dengan CRUD / logika kompleks**: fitur berlapis, backend + frontend, butuh quality gate sebelum rilis. Saudara dari [`_landing-template`](https://github.com/redhoram/claude-landing-template) (yang buat landing page & app sederhana), tapi dioptimalkan untuk kerja fitur yang terstruktur.

### Filosofi: pipeline batch dengan quality gate

Fitur kompleks butuh perencanaan, implementasi, pengujian, dan review yang terpisah jelas, biar tiap tahap bisa diaudit dan nggak ada yang kelewat. Alurnya doc-driven: tiap subagent nulis artefak ke `.pipeline/`, lalu di-handoff ke tahap berikutnya.

- **`/ship [deskripsi fitur]`**: jalanin pipeline end-to-end (planner → designer → coder → tester → reviewer). Reviewer kasih verdict akhir (SHIP / NEEDS WORK / BLOCK); kalau SHIP, tahap **deployer** opsional nyiapin perubahan buat produksi (inventaris env var, infra config, deploy report).
- **Open-questions gate**: kalau spec dari planner masih ada open question, pipeline berhenti dan nanya kamu dulu sebelum designer/coder jalan, jadi nggak ada tahap yang mewarisi tebakan.
- **`/fix [permintaan perubahan]`**: jalur ringan buat merevisi fitur yang udah ada (ganti nama, ubah teks, rapikan layout, perbaiki bug). Sesi utama nulis spec singkat, lalu coder → tester → reviewer jalan. Revisi tetap ninggalin spec, test report, dan verdict, bukan lolos sebagai edit tanpa jejak.
- **Model tiering**: planner, designer & reviewer jalan di `opus` (leverage-nya ada di perencanaan dan judgment), coder, tester & deployer jalan di `sonnet` (eksekusi murah terhadap spec yang udah nggak ambigu). Model tiap agent bisa diganti per project lewat tabel `Model override` di `CLAUDE.md`, tanpa ngedit file agent.
- **Cek visual**: tiap perubahan UI, tester merender layar yang kena di lebar yang tertulis di `CLAUDE.md`, ngeliat screenshot-nya, dan ngelaporin yang rusak walaupun nggak ada di kriteria. Severity 3 ke atas menahan SHIP.
- **Jejak audit**: tiap run mindahin artefak `.pipeline/` sebelumnya ke `.pipeline/archive/<timestamp>/`, bukan ngehapus.
- **Commit saat SHIP**: pipeline nawarin commit cuma untuk file di changelog, di-stage per path. Nggak pernah push tanpa konfirmasimu.
- Untuk tweak satu baris: nggak wajib pakai command apa pun; langsung minta Claude Code biasa.

### Beda dari `_landing-template`

| | `_pipeline-template` (CRUD kompleks) | `_landing-template` (landing & app sederhana) |
|---|---|---|
| Mode | Batch, doc-driven, fire-and-forget | Loop visual + interaktif |
| Agent | planner → designer → coder → tester → reviewer (→ deployer setelah SHIP) | writer → designer → builder → qa |
| Gate akhir | Reviewer (SHIP/BLOCK, fokus keamanan) | QA (punch-list, fokus UI/UX) |
| Command | `/ship` (fitur baru) + `/fix` (revisi) | `/craft` (0→1) + `/polish` (1→bagus) |
| Verifikasi | build/lint/test + skenario | live preview: render, responsif, console |

### Isi

- `CLAUDE.md`: instruksi project berisi stack, batasan keras, keputusan final, file final, sumber konten, perintah verifikasi, target deployment, model override, konvensi, alur pipeline. **Isi placeholder-nya dulu; hapus bagian yang nggak relevan.**
- `.claude/agents/`: `planner`, `designer`, `coder`, `tester`, `reviewer`, `deployer`
- `.claude/commands/`: `ship`, `fix`
- `.claude/skills/`: 9 skill bawaan yang ditegakkan para agent (matriks di bawah)
- `.pipeline/`: workspace handoff (`spec.md`, `design.md`, `changelog.md`, `test-report.md`, `verdict.md`, `deploy-report.md`), dengan run lama di `.pipeline/archive/`

### Matriks skill

Tiap agent cuma baca skill milik tahapnya: didokumentasikan di sini biar pemetaannya bisa diaudit:

| Skill | Dipakai oleh | Menegakkan |
|---|---|---|
| `relational-mastery` | planner (fitur berdata aja) | Normalisasi (3NF), lifecycle FK dengan ON DELETE/UPDATE eksplisit, justifikasi index per query, kolom ownership/tenant, skema siap-ORM di section Data model spec |
| `premium-design` | designer | Disiplin palet, metrik tipografi, motion yang disengaja, intensitas marketing-vs-product |
| `ux-research` | designer + tester | Landasan perilaku: user + JTBD dulu, 3-5 prinsip psikologi bernama per surface, checklist Nielsen/Krug saat mendesain, garis keras anti dark-pattern. Tester menjalankan diagnostik usability 10 pertanyaan (severity 0-4) pada UI yang di-drive |
| `secure-coding` | coder + reviewer + deployer | Satu bar keamanan bersama (turunan OWASP): injection, authz/ownership, secrets, error fail-closed, risiko LLM. Coder menerapkannya saat menulis; reviewer mengaudit daftar yang sama lewat diff-scan §7-nya; deployer menerapkan §4 ke persiapan env/infra pasca-SHIP |
| `systematic-debugging` | coder (ronde fix) | Root cause dulu sebelum fix apa pun, satu fix per penyebab, aturan 3-strikes: berhenti dan lapor, bukan terus menebak |
| `testing-discipline` | tester | Gerbang bukti (nggak ada PASS tanpa output segar), jebakan edge-case standar (termasuk angka dan waktu), berburu error yang ditelan diam-diam, regression radar ke pemakai kode yang diubah, higiene async/flaky, mendeteksi suite hijau yang bohong |
| `api-contract-guardian` | planner + coder + reviewer (data lintas lapisan) | Tabel Contract di spec (database, validasi backend, tipe frontend, nullability), sinkron enum dan penamaan, tanggal ISO, uang bukan float, panggilan pengubah data yang idempotent, perubahan yang backward-compatible |
| `performance-profiler` | planner + coder + reviewer (list, query, ekspor) | Skala data tertulis di spec, nggak ada N+1 atau query tanpa batas, paginasi, listener dibersihkan dan cache dibatasi, file besar di-stream, list panjang di-virtualisasi; tiap temuan nyebut skala tempat dia rusak |
| `architecture-clean-code` | coder + reviewer | Pemisahan lapisan, nggak ada dead code sisa diff, aturan tiga kali untuk duplikasi, nggak ada import melingkar, ukuran file sebagai sinyal. Cuma menilai yang ditambah diff; batasan keras di `CLAUDE.md` selalu menang |

Patut tahu, sengaja **nggak** di-bundle: [`differential-review`](https://github.com/trailofbits/skills) dari Trail of Bits (security diff review lebih dalam; lisensi CC BY-SA, install terpisah, jangan di-vendor ke repo MIT) dan [`webapp-testing`](https://github.com/anthropics/skills) dari Anthropic (UI testing berbasis Playwright; tambahkan per-project saat browser e2e beneran dibutuhkan).

### Prasyarat

- [Claude Code](https://claude.com/claude-code) ter-install: `/ship` itu **slash command Claude Code**, diketik di dalam sesi Claude Code di folder project (bukan script npm).
- Toolchain stack-mu (mis. Node.js + npm) biar tester beneran bisa jalanin build/lint/test.

### Bikin project baru

1. Copy `_pipeline-template` → rename folder
2. Isi `CLAUDE.md` (ganti semua placeholder: stack, batasan keras, perintah verifikasi, target deployment, konvensi; isi Deployment Target dengan "none" kalau hasil akhirnya diserahkan manual)
3. Scaffold stack-mu
4. Git history baru: buka Claude Code di folder itu. Di sesi pertama dia deteksi `.git` bawaan template dan reset otomatis (atau manual: hapus folder tersembunyi `.git`, lalu `git init`). `git init` doang TIDAK menghapus history/remote template.
5. Buka Claude Code di folder itu → jalankan `/ship [deskripsi fitur]`

### Standar

Yang bikin output beda dari generator instan: **terencana, teruji, dan di-review**, bukan sekadar jalan. Detail di `CLAUDE.md` + tiap file agent.

Dan **gagal keras, bukan diam-diam**: coder berhenti kalau spec hilang/ambigu daripada nebak, tester lapor kegagalan apa adanya tanpa nambal, dan verdict-nya mekanis: ada satu aja kriteria FAIL atau NOT TESTED berarti nggak mungkin SHIP.

### Kredit

**Author**
- Redho Ramadhani: [linkedin.com/in/redhoramadhanihamid](https://id.linkedin.com/in/redhoramadhanihamid) · [github.com/redhoram](https://github.com/redhoram)

Dibangun dengan [Claude Code](https://claude.com/claude-code).
