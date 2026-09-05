# PRD — IntraDocs

Berkas ini menjelaskan tujuan. Scope-mvp menentukan cakupan/gerbang; ADR-0012 mengganti asumsi tenggat tetap.

## Masalah dan hasil

Dokumentasi internal sulit ditemukan, mutu/kemutakhirannya tidak jelas, dan akses tiap orang berbeda. Hasil yang dikejar: jawaban dapat ditelusuri ke sumber, AI tidak melewati izin, dan kueri tak terjawab tercatat sebagai bahan perbaikan pengetahuan.

## Pengguna dan cerita utama

- Viewer aktif mencari, membaca, dan bertanya dengan sitasi; tanpa bukti, menerima abstain.
- Contributor membuat/sunting/mempublikasikan dokumen milik sendiri sesuai cakupan dan klasifikasi.
- Reviewer melihat/mengelola dokumen pada cakupan yang diizinkan, tidak mendapat sumber sensitif kategori lain.
- Admin mempunyai akses global sesuai model MVP; pembacaan sensitif tercatat.
- Pengguna aktif dapat bertanya lanjutan dan membatasi scope kategori/dokumen. Riwayat tetap pribadi dan mengikuti izin terkini.
- Pengguna nonaktif ditolak, termasuk sesi yang dibuat sebelum dinonaktifkan.

Keempat role aktif harus dapat diuji, tetapi tidak semua pasangan wajib memiliki hasil berbeda bila izin efektifnya sama. Multi-turn/scope/form adalah requirement, bukan bonus jika waktu tersisa.

## Batas dan bukti

Semua akun/konten sintetis; ini demo MVP, bukan persetujuan memasukkan dokumen perusahaan asli. Form md/txt, publikasi sinkron, viewer/katalog/search/AI membentuk irisan vertikal.

Approval, konsol admin, SSO, multi-format/OCR, favorit, riwayat baca dan dashboard ditunda. Log query/audit tetap dibangun; dashboard nanti memakai data tersebut.

Kelulusan mengikuti definisi selesai/angka di docs/scope-mvp.md serta metode docs/eval/README.md. Mockup adalah sumber tata letak/istilah; deviasi eksplisit ada di mockup-alignment, bukan dianggap requirement mentor sudah terpenuhi seluruhnya.
