# Keadaan Sekarang

Berkas ini berumur pendek dan sering berubah. Fakta proyek yang stabil ada di `docs/context-pack.md`; scope yang mengikat ada di `docs/scope-mvp.md`.

Perbarui di akhir setiap hari sprint. Satu commit, dua menit.

- **Terakhir diperbarui:** 4 September 2026, sore
- **Hari sprint:** 1 dari 6
- **Target MVP:** Jumat, 11 September 2026

## Sudah selesai

- Mockup dibaca utuh, sebelas layar, dipetakan ke task
- Dokumen fondasi: PRD, arsitektur, matriks RBAC, kontrak API, spec task T-001 sampai T-008
- ADR 0001 sampai 0010
- Scope dipotong ke sprint enam hari lewat ADR-0007
- Rencana lingkungan, CI/CD, dan protokol pindah akun AI
- Pemeriksaan kesesuaian terhadap layar 9-11 mockup, lima temuan diperbaiki

## Sedang dikerjakan

PR #1 terbuka dan menunggu review. Belum ada kode aplikasi.

## Berikutnya

PR scaffold: T-001 kerangka Next.js, T-002 design token dan `AppShell` responsif, T-003 skema sebelas tabel dengan seed.

Perubahan yang harus ikut masuk ke PR scaffold:

- `user.category_scope` dan `user.is_active` pada skema (ADR-0009)
- Tabel `conversation` dan `message` (ADR-0010)
- Sidebar menutup di bawah titik potong pada `AppShell`
- `docs/ui-inventory.md`, `docs/context-pack.md`, `docs/prd.md`, dan `docs/architecture.md` masih menyebut scope empat minggu dan delapan layar. Selaraskan di PR ini

## Menunggu atau rusak

- Akun Neon dengan pgvector — belum ada, penghambat T-003
- Kunci API penyedia AI — belum ada, penghambat hari 5
- Badan PR #1 masih menjelaskan empat commit pertama saja

## Catatan

Seluruh konten dokumen bersifat sintetis dan fiktif. Tidak ada dokumen Telkom asli yang boleh masuk ke lingkungan mana pun yang terhubung API publik.
