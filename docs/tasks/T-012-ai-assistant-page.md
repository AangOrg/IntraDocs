# T-012: Halaman AI Assistant

Pemilik B; prasyarat T-002. Paralel T-011 dengan kontrak SSE tetap; integrasi harus selesai sebelum ditutup.

## Baca tambahan

API AI; ui-inventory dan mockup-alignment layar 9.

## Subtask

- T-012a: layout, percakapan/sidebar, pemilih scope, state/loading/abstain/error.
- T-012b: konsumsi SSE, sitasi, feedback dan integrasi riwayat.

/ai-assistant memakai q untuk prefilling dan scope dokumen/kategori sesuai kontrak. Pil Jawaban dibatasi hak akses Anda dan tombol Percakapan baru memakai istilah mockup. Scope/riwayat bukan opsional karena tanggal.

## Kriteria terima

- [ ] meta menyimpan conversationId; sources menampilkan dasar sebelum token; done menyimpan aiQueryId untuk feedback.
- [ ] Giliran lanjutan benar-benar terkirim pada percakapan sama; reload memuat riwayat milik sendiri.
- [ ] Scope all/category/document dikirim dan UI tidak menampilkan pilihan yang tak berfungsi.
- [ ] Kartu sumber menampilkan judul, versi, heading, verified; tautan menuju anchor sumber. Versi usang ditandai sesuai kontrak, tidak pura-pura versi terbaru.
- [ ] 401/404 riwayat membersihkan konten sensitif dari tampilan dan menawarkan percakapan baru.
- [ ] Stream terputus menampilkan jawaban belum selesai, bukan keberhasilan. Feedback menggunakan ID milik pengguna.
- [ ] Test komponen SSE/state/sitasi, integrasi endpoint, keyboard dan viewport 375 px lulus.

Tidak ada berbagi percakapan, ekspor, lampiran, chip saran atau salin tautan publik. Boleh salin teks jawaban lokal; tautan percakapan tetap perlu login pemilik.
