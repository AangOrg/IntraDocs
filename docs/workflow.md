# Workflow tim & vibecoding

## Pembagian peran

Dibagi berdasarkan batas teknis, bukan per-screen, supaya dua orang jarang menyentuh file
yang sama.

| | **Orang A — Platform & AI** | **Orang B — Product & Experience** |
| --- | --- | --- |
| Milik | Skema DB, migrasi, auth, penegakan RBAC, job queue, parser, search, RAG, CI, deploy | Seluruh UI dari mockup, design token, viewer, wizard upload, konsol admin, aksesibilitas & performa frontend |
| Minggu 1 | Auth + RBAC + filter + test | Layout shell, katalog, viewer, token |
| Minggu 2 | Pipeline ingest + hybrid search | Wizard upload 4 langkah + halaman search & filter |
| Minggu 3 | Endpoint RAG + eval harness | UI chat + sitasi + state loading/error/abstain |
| Minggu 4 | Performa DB, backup/restore, runbook | UAT, aksesibilitas, polish, materi demo |
| Juga | Security review | Content seeding, 30 pertanyaan eval, koordinasi UAT |

**Hari 1–2: sepakati bersama skema DB dan `docs/api-contract.md`.** Setelah itu keduanya
bisa jalan paralel tanpa saling menunggu. Ini satu-satunya cara agar dua orang tidak
berubah menjadi satu orang yang saling memblokir.

## Loop vibecoding

```
1. Ambil satu task dari docs/tasks/ (sudah ada spec + acceptance criteria)
2. Agent murah: baca kode terkait -> tulis di branch feat/T-00X-slug -> buka PR + cara menguji
3. Manusia: pull -> jalankan -> review diff dengan model kelas atas
4. Perbaikan kecil di-commit langsung; perubahan besar direvisi lewat komentar PR
5. CI hijau -> merge -> tandai task selesai
```

### Ekonomi penggunaan model

Prinsip: **model mahal untuk keputusan, model murah untuk produksi kode.** Jangan dibalik.

| Pekerjaan | Model |
| --- | --- |
| Brainstorm, PRD, ADR, task spec | Model murah/menengah dengan konteks panjang |
| Scaffolding, CRUD, migrasi, komponen dari mockup, boilerplate test | Model murah, dipandu spec |
| Review diff, debug sulit, tuning kualitas RAG, security review, keputusan refactor | Model kelas atas |
| Verifikasi akhir | CI + manusia |

Aturan yang paling berdampak:

1. **Tempel `docs/context-pack.md`, jangan menjelaskan ulang proyek.** Penghemat token
   terbesar yang tersedia.
2. **Beri model kelas atas sebuah diff, spec, dan test yang gagal** — bukan permintaan
   "tolong bikin fitur X". Konteks sempit menghasilkan jawaban lebih baik dan lebih murah.
3. **Batch review** satu sampai dua kali sehari, bukan per commit.
4. **PR kecil.** PR besar berarti review dangkal, bug lolos, lalu token terbuang untuk debugging.
5. **Satu task = satu branch = satu himpunan file.** Jangan pernah dua agent menyentuh file
   yang sama bersamaan; konflik merge adalah pemborosan terbesar dalam vibecoding.
6. **CI adalah reviewer pertama.** `tsc` dan linter menangkap sebagian besar kesalahan agent
   secara gratis. Jangan pakai model untuk pekerjaan yang bisa dilakukan compiler.

## Aturan branch & PR

- `main` dilindungi: wajib PR, wajib CI hijau, tanpa force-push.
- Branch: `feat/T-00X-slug`, `fix/...`, `docs/...`, `chore/...`.
- Conventional Commits.
- Target diff < 400 baris. Kalau lebih besar, pecah task-nya.
- Semua PR memakai template dan menautkan ID task.
- Agent AI tidak pernah merge, tidak pernah menghapus branch, dan tidak pernah menulis ke
  `main` tanpa instruksi eksplisit dari manusia.

## Review

- Review silang wajib. SLA 24 jam.
- PR < 100 baris boleh disetujui cepat.
- PR yang menyentuh **RBAC, retrieval, atau autentikasi wajib dibaca serius** oleh yang lain,
  berapa pun ukurannya.
- Reviewer memeriksa tiga hal, dalam urutan ini: benar secara izin, benar secara fungsi,
  lalu baru gaya kode.

## Definition of Done

Sebuah task selesai ketika:

- [ ] Semua acceptance criteria pada task spec terpenuhi
- [ ] `pnpm typecheck && pnpm lint && pnpm test` hijau
- [ ] Ada test untuk logic yang berisiko (izin, parsing, scoring)
- [ ] Sudah diuji manual mengikuti bagian "Cara menguji" pada PR
- [ ] Tidak ada kode mati, `console.log`, atau data placeholder yang tertinggal
- [ ] Dokumen terkait diperbarui (ADR bila keputusan berubah, context-pack bila skema berubah)
- [ ] Tidak ada regresi budget performa

## Ritme

- Standup async 10 menit sekali sehari di satu kanal: selesai / hari ini / blocker.
- "Integrator minggu ini" bergilir: bertanggung jawab menjaga `main` hijau dan deploy jalan.
- Jumat: demo internal 15 menit atas gate minggu tersebut (lihat `docs/roadmap.md`).
- **Bus factor:** keduanya harus bisa menjalankan proyek dari nol. Uji ini di Minggu 2,
  bukan di Minggu 4.
