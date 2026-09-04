# STATUS

Diperbarui: hari 1 sprint, sore. Fondasi perencanaan sudah masuk `main`.

## Keadaan

**Perencanaan selesai dan dibekukan.** PR #1 sudah di-merge, jadi seluruh dokumen fondasi ada di `main` dan bisa dibaca langsung. Semua dokumen sudah konsisten satu sama lain setelah pemeriksaan menyeluruh terhadap 11 layar mockup, dan spec keempat belas task sudah lengkap — setiap sesi eksekusi bisa berdiri sendiri tanpa konteks percakapan sebelumnya.

**Belum ada kode aplikasi sama sekali.** Repo saat ini berisi dokumen perencanaan dan berkas mockup.

## Task berikutnya — hari 1

Tiga task, boleh dibagi dua orang. Kerjakan satu task per sesi.

- **T-001** scaffold Next.js + CI + deploy Vercel — orang A
- **T-002** token desain + AppShell responsif — orang B, tidak bergantung T-001 selesai
- **T-003** skema 11 tabel + migrasi + seed — orang A, setelah T-001

Urutan yang tidak boleh dibalik ada di `docs/eksekusi.md`.

## Dua penyesuaian jadwal yang sudah diputuskan

1. **T-008 (korpus sintetis) boleh dimulai kapan saja, termasuk sekarang.** Menulis 20–25 dokumen Markdown fiktif tidak bergantung pada satu baris kode pun, dan realistisnya butuh 4–6 jam menulis — ini item yang paling sering diremehkan di seluruh sprint. Mengerjakannya lebih awal melepas setengah hari di hari 3 dan menghapus risiko terbesar terhadap hari 4. Tulis sekalian 10 pertanyaan evaluasinya, selagi isi dokumennya masih segar di kepala.
2. **Pengukuran mutu retrieval pindah ke hari 4**, di dalam T-009, memakai `scripts/eval-retrieval.ts` yang tidak memanggil LLM generasi. Gerbangnya: hit@5 di bawah 0,7 berarti berhenti dan perbaiki retrieval sebelum menyentuh hari 5.

## Yang harus ikut sejak hari 1, jangan sampai terlewat

- `user.category_scope` dan `user.is_active` di skema (ADR-0009)
- Tabel `conversation` dan `message` (ADR-0010) — total 11 tabel, bukan 9
- Sidebar responsif di `AppShell` sejak awal, bukan ditambal belakangan
- Seed berisi satu reviewer bercakupan sempit dan satu pengguna nonaktif
- `AI_MAX_CLASSIFICATION=secret` di `.env.example` — nilai `public` akan mematikan demo tanpa terlihat

## Butuh dari pemilik repo

Panduan langkah demi langkah ada di **`docs/setup-infra.md`**. Ringkasnya:

1. **Akun Neon**, region Singapore, ekstensi `vector` aktif. **Memblokir T-003.**
2. **API key penyedia AI**. Memblokir T-009 dan seterusnya, belum dibutuhkan hari 1.
3. **Project Vercel** dengan region fungsi Singapore dan env var terisi untuk Production dan Preview.

Region Neon dan region fungsi Vercel harus sama. Kalau berbeda, target p95 tidak akan tercapai dan memperbaikinya butuh membuat project Neon baru.

## Catatan

Hari 5 (T-011 + T-012) adalah hari terberat dan keduanya ada di jalur paling berisiko. Kalau tersendat, yang dipotong pertama adalah penyimpanan riwayat percakapan — chat tetap berfungsi, riwayatnya saja yang tidak tersimpan. Urutan potong lengkap ada di `docs/scope-mvp.md`.

Batas durasi fungsi Vercel hanya terlihat setelah dideploy, tidak di localhost. Uji satu jawaban panjang di preview pada hari 5 — lihat `docs/setup-infra.md`.

## Cara memperbarui berkas ini

Setiap PR memperbarui bagian "Keadaan" dan "Task berikutnya". Maksimal 15 baris perubahan. Kalau sebuah informasi hanya ada di dalam chat, informasi itu belum ada.
