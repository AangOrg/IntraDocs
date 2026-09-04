# PRD — IntraDocs

Berkas ini menjelaskan **kenapa**. Daftar **apa saja** yang masuk MVP ada di `docs/scope-mvp.md` dan itu yang mengikat.

## Masalah

Dokumentasi internal divisi IT tersebar di banyak tempat. Orang bertanya ke Service Desk untuk hal yang sebenarnya sudah ada dokumennya, karena mencari lebih mahal daripada bertanya. Dokumen yang ketemu pun sering tidak jelas masih berlaku atau tidak, dan tidak semua orang boleh membaca semuanya.

## Tiga hasil yang dikejar

1. Pertanyaan umum terjawab tanpa manusia, dengan jawaban yang **bisa ditelusuri ke dokumen sumber**.
2. Jawaban **tidak pernah** menampilkan dokumen di luar hak akses penanya.
3. Pengetahuan yang belum terdokumentasi bisa **terlihat sebagai data**, bukan dugaan.

Poin 1 dan 2 dibuktikan minggu ini. Poin 3 dipersiapkan datanya minggu ini (log kueri) dan ditampilkan setelah MVP.

## Pengguna

| Role | Kebutuhan utama | Bukti di MVP |
|---|---|---|
| Viewer | Menemukan jawaban cepat, tahu mana yang masih berlaku | Cari, baca, tanya AI |
| Contributor | Menaruh pengetahuan tanpa ritual berat | Dokumen tersedia via seed; form sunting bila waktu cukup |
| Reviewer | Menjaga kualitas di kategori tanggung jawabnya | Cakupan kategori aktif di semua jalur baca |
| Admin | Melihat kondisi knowledge base dan mengatur akses | Akses penuh + audit log tercatat |

## Cerita pengguna yang harus jalan di demo

1. Sebagai **Viewer**, saya mengetik pertanyaan bahasa Indonesia dan mendapat jawaban dengan sitasi ke dokumen yang boleh saya baca.
2. Sebagai **Viewer**, ketika jawabannya tidak ada di korpus, sistem mengatakan tidak tahu — bukan mengarang.
3. Sebagai **Reviewer dengan cakupan sempit**, saya melihat dokumen terbatas kategori saya, dan tidak melihat dokumen terbatas kategori lain — di katalog, di pencarian, maupun di jawaban AI.
4. Sebagai **Admin**, saya melihat dokumen terbatas milik semua kategori, dan pembacaan itu tercatat di audit log.
5. Sebagai siapa pun, saya bisa bertanya lanjutan ("kalau perangkatnya hilang bagaimana?") dan sistem memahami konteks percakapan.

Cerita 1–4 adalah definisi lulus. Cerita 5 kuat tapi boleh turun kualitasnya kalau hari 5 tersendat.

## Bukan tujuan MVP

Alur approval, versioning berjenjang, konsol admin, SSO, unggah multi-format, OCR, dasbor analitik, notifikasi, integrasi ticketing. Semuanya ada di roadmap dan sebagian besar memang fase 2 menurut roadmap mockup sendiri.

## Kriteria terima MVP

MVP dinyatakan tercapai bila di lingkungan yang bisa diakses lewat URL:

- Empat role bisa login dan melihat isi yang berbeda sesuai izin.
- Katalog, pencarian, dan AI memakai **satu** filter izin yang sama.
- `tests/rbac/ai-retrieval-leak.spec.ts` hijau.
- Jawaban AI selalu punya sitasi, atau menyatakan tidak menemukan.
- 10 pertanyaan evaluasi lulus ambang di `docs/eval/README.md`.

## Hubungan dengan roadmap mockup

Mockup mengusulkan 5–6 bulan dengan AI di fase 3. Sprint ini bukan versi dipercepat dari fase 1 — ia **irisan vertikal** yang menembus fase 1 dan fase 3 sekaligus dan sengaja melewati fase 2. Alasannya: fase 1 hampir tanpa risiko teknis, sedangkan seluruh risiko produk ada di fase 3. Rencana mockup mengetahui jawabannya di bulan kelima; rencana ini mengetahuinya di hari keenam.
