# T-007: Halaman muka dan katalog

Pemilik B; prasyarat T-002/T-003/T-005. Fixture T-003 cukup untuk pengembangan; demo final memakai T-008.

## Baca tambahan

API katalog; ui-inventory; mockup-alignment layar 1/3 bila perlu.

## Subtask

- T-007a: GET documents/categories/labels dan hitungan terotorisasi.
- T-007b: / dan /katalog, kartu kategori, daftar dokumen terbaru, filter kategori/cursor di URL.

Pakai AppShell, jangan dibuat ulang. Kategori satu tingkat. Halaman muka berisi pencarian, enam kartu kategori, dokumen published terbaru, tautan AI; statistik hanya bila dihitung sesudah filter izin.

Tidak ada topik populer, paling banyak dibaca, favorit, riwayat baca, subkategori, atau filter selain kategori. Cursor/limit mengatur daftar, bukan fitur filter produk tambahan. Tidak membuat tabel document_view.

## Kriteria terima

- [ ] Daftar dan total memakai filter sama, total dihitung sebelum pagination tetapi sesudah izin.
- [ ] Viewer Demo/reviewer sempit/admin mendapat tepat daftar fixture yang diizinkan. Tidak semua pasangan harus berbeda.
- [ ] Ganti kategori/refresh/pagination mempertahankan URL dan tidak mengulang/melewatkan item akibat urutan tidak stabil.
- [ ] Ringkasan public/internal tetap lintas kategori; restricted mengikuti scope.
- [ ] State loading/kosong/error dan keyboard/375 px diperiksa.
- [ ] Tidak ada judul terkunci, hitungan global bocor, atau angka mockup yang menjadi fakta.

Tambahkan integrasi endpoint pada catalog-visibility.spec.ts; jangan hanya menguji komponen dengan array tersaring.
