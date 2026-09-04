# STATUS

Diperbarui: hari 1 sprint, siang.

## Keadaan

**Perencanaan selesai dan dibekukan.** Semua dokumen di repo sudah konsisten satu sama lain setelah pemeriksaan menyeluruh terhadap 11 layar mockup. Tidak ada lagi dokumen yang menjelaskan ruang lingkup lama. Spec untuk keempat belas task sudah lengkap, jadi setiap sesi eksekusi bisa berdiri sendiri.

Belum ada kode aplikasi sama sekali. PR #1 berisi seluruh fondasi perencanaan dan masih menunggu review.

## Task berikutnya

Hari 1 — tiga task, boleh dibagi dua orang:

- **T-001** scaffold Next.js + CI + deploy Vercel (orang A)
- **T-002** token desain + AppShell responsif (orang B)
- **T-003** skema 11 tabel + migrasi + seed (orang A, setelah T-001)

Urutan yang tidak boleh dibalik ada di `docs/eksekusi.md`.

## Dua penyesuaian jadwal yang sudah diputuskan

1. **T-008 (korpus sintetis) boleh dimulai kapan saja, termasuk sekarang.** Menulis 20–25 dokumen Markdown fiktif tidak bergantung pada satu baris kode pun, dan realistisnya butuh 4–6 jam menulis — ini item yang paling sering diremehkan di seluruh sprint. Mengerjakannya lebih awal, di sela waktu atau di malam hari, melepas setengah hari di hari 3 dan menghilangkan risiko terbesar terhadap jadwal hari 4. Tulis sekalian 10 pertanyaan evaluasinya bersamaan, selagi isi dokumennya masih segar di kepala.
2. **Pengukuran mutu retrieval pindah ke hari 4**, di dalam T-009, memakai `scripts/eval-retrieval.ts` yang tidak memanggil LLM generasi. Alasannya ada di spec T-009. Singkatnya: menemukan retrieval yang buruk di hari 6 berarti tidak ada waktu memperbaikinya.

## Yang harus ikut sejak hari 1, jangan sampai terlewat

- `user.category_scope` dan `user.is_active` di skema (ADR-0009)
- Tabel `conversation` dan `message` (ADR-0010) — total 11 tabel, bukan 9
- Sidebar responsif di `AppShell` sejak awal, bukan ditambal belakangan
- Seed berisi satu reviewer bercakupan sempit dan satu pengguna nonaktif
- `AI_MAX_CLASSIFICATION=secret` di `.env.example` — nilai `public` akan mematikan demo tanpa terlihat

## Butuh dari pemilik repo

1. **Akun Neon** dengan `pgvector` aktif, lalu `DATABASE_URL` ditaruh di `.env.local` dan di Vercel. ~5 menit. Memblokir T-003.
2. **API key penyedia AI** untuk embedding dan generasi. ~5 menit. Memblokir T-009 dan seterusnya, tapi belum dibutuhkan hari 1.
3. Review dan merge PR #1, supaya dokumen perencanaan ada di `main` dan bisa dibaca sesi eksekusi berikutnya.

## Catatan

Hari 5 (T-011 + T-012) adalah hari terberat. Kalau tersendat, yang dipotong pertama adalah penyimpanan riwayat percakapan — chat tetap berfungsi, riwayatnya saja yang tidak tersimpan. Urutan potong lengkap ada di `docs/scope-mvp.md`.
