---
description: Kerjakan satu task spec IntraDocs dari awal sampai PR
argument-hint: T-005
---

Kerjakan task **$ARGUMENTS**.

Sebelum menulis kode, baca berkas ini dengan urutan berikut:

1. `docs/context-pack.md` — fakta proyek
2. `docs/scope-mvp.md` — batas scope; berkas ini mengikat
3. `AGENTS.md` — aturan menulis kode di repo ini
4. `docs/tasks/$ARGUMENTS-*.md` — spec task ini
5. ADR apa pun yang disebut di dalam spec tersebut

Lalu:

- Buat branch `feat/$ARGUMENTS-<slug-pendek>`. Jangan pernah bekerja di `main`.
- Kerjakan **hanya** yang ada di dalam lingkup spec. Kalau menemukan hal lain yang perlu
  diperbaiki, catat di deskripsi PR, jangan dikerjakan sekalian.
- Kalau spec-nya ambigu atau terasa salah, **tanya dulu sebelum menulis kode.** Menebak lalu
  menulis 300 baris ke arah yang salah jauh lebih mahal daripada satu pertanyaan.
- Jalankan `pnpm typecheck && pnpm lint && pnpm test` sebelum commit.
- Commit dengan Conventional Commits, satu commit logis per perubahan yang berdiri sendiri.
- Buka PR memakai template repo. Sebutkan ID task dan cara mengujinya secara manual.

Batasan yang tidak boleh dilanggar:

- Filter izin selalu lewat `visibleDocumentsFilter` di SQL, tidak pernah di dalam prompt LLM.
- Tanpa warna atau spacing hardcoded; pakai design token.
- Tanpa `any` di TypeScript.
- Setiap tampilan punya state loading, kosong, error, dan tanpa izin.
- Jangan pernah merge PR atau menghapus branch.

Setelah selesai, laporkan: apa yang berubah, apa yang sudah diuji, dan apa yang sengaja tidak
dikerjakan.
