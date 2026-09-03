# Keputusan yang belum selesai

Daftar hidup. Setiap baris punya penanggung jawab dan tenggat. Kalau sebuah baris sudah
dijawab, pindahkan ke ADR dan hapus dari sini.

> Baris bertanda BLOKIR memblokir pekerjaan di minggu-minggu berikutnya. Kejar jawabannya
> lebih dahulu, dan jangan menunggu kode siap untuk mulai mengurus izinnya.

| # | Pertanyaan | Kenapa penting | PIC | Tenggat | Status |
| --- | --- | --- | --- | --- | --- |
| 1 **BLOKIR** | Bolehkah dokumentasi internal diproses oleh LLM cloud (dengan kontrak/DPA)? Kalau tidak, apakah tersedia server GPU on-prem? | Menentukan `AI_MAX_CLASSIFICATION` dan apakah produk bisa dipakai dengan dokumen nyata | — | Minggu 1 | Terbuka |
| 2 **BLOKIR** | Di mana produksi akan berjalan: server/cloud internal atau cloud publik? | Menentukan target deploy dan batasan runtime | — | Minggu 2 | Terbuka |
| 3 **BLOKIR** | Dokumen nyata mana yang boleh dipakai untuk pilot, dan siapa pemilik/approver-nya? | Tanpa konten nyata, platform tidak akan dipakai | — | Minggu 2 | Terbuka |
| 4 | Siapa 5–10 orang peserta UAT di Minggu 4? | Gate Minggu 4 bergantung pada ini | — | Minggu 3 | Terbuka |
| 5 | Apakah SSO/OIDC internal tersedia, dan apa issuer serta klaim yang dikirim? | MVP memakai akun lokal; OIDC sudah ter-wire di balik flag | — | Minggu 3 | Terbuka |
| 6 | Berapa baseline pertanyaan berulang per bulan untuk 5 topik teratas? | Tanpa baseline, dampak proyek tidak bisa dibuktikan | Orang B | Minggu 2 | Terbuka |
| 7 | Daftar unit/divisi resmi dan pemetaan clearance awal per unit | Dibutuhkan untuk seed data RBAC yang realistis | Orang A | Minggu 1 | Terbuka |
| 8 | Berapa lama log pertanyaan AI (`ai_query`) boleh disimpan? | Kebijakan retensi data pribadi | — | Minggu 3 | Terbuka |
| 9 | Apakah butuh ekspor PDF dengan watermark klasifikasi? | Sering diminta saat review keamanan | — | Minggu 4 | Terbuka |
| 10 | Siapa pemilik operasional setelah handover (backup, reindex, kelola user)? | Menentukan isi runbook | — | Minggu 4 | Terbuka |

## Template pertanyaan untuk stakeholder

Salin dan kirim untuk mengejar baris 1–4:

> 1. Untuk fitur AI assistant, isi dokumen perlu diproses oleh model bahasa. Apakah kami
>    diizinkan memakai layanan cloud yang terikat kontrak pemrosesan data, atau harus
>    seluruhnya di dalam infrastruktur internal? Kalau harus internal, apakah tersedia
>    server ber-GPU?
> 2. Aplikasi ini nantinya akan di-hosting di mana?
> 3. Untuk pilot, dokumen apa yang boleh kami gunakan, dan siapa pemilik yang berwenang
>    menyetujui publikasinya?
> 4. Siapa 5–10 rekan yang bisa kami libatkan untuk uji coba pengguna di minggu terakhir?
>
> Selama poin 1 belum ada jawaban, kami mengembangkan dengan dokumen contoh yang dibuat
> sendiri, bukan dokumen internal asli.
