# T-006: Dokumen, publikasi, viewer

Pemilik B untuk UI, koordinasi A untuk pipeline; satu pemilik per subtask. Prasyarat T-002/T-003/T-004/T-005. API key embedding dibutuhkan pada subtask publikasi, tidak baru pada T-009.

## Baca tambahan

API katalog/dokumen; arsitektur publikasi; ui-inventory layar 4/5; matriks aksi.

## Subtask berurutan

- T-006a (B): CRUD draft terotorisasi, form/pratinjau, masukan md/txt jadi teks.
- T-006b (A): AiProvider.embed, parser heading/anchor/chunker, scan sensitif, pipeline publish/reindex sinkron dan test transaksi.
- T-006c (B): viewer server tersanitasi, DocNav/metadata/expired state dan integrasi sitasi.

Tidak ada task T-015 terpisah: scan sudah termasuk T-006b. Modul lib/ai/provider.ts menyediakan antarmuka embed/chat; implementasi chat milik T-011. Koordinasikan perubahan berkas, bukan dua implementasi provider.

## Kontrak publikasi

POST/PATCH dokumen dan POST /api/documents/:id/publish mengikuti API. Konten draft berbeda dari versi published. Validasi klasifikasi/cakupan target, optimistic revision, scan sebelum embed. Konfirmasi warning hanya untuk false-positive sintetis dan terikat revision; data asli tetap dilarang.

Ikuti transaksi pada arsitektur: embedding gagal tidak menerbitkan versi setengah jadi; perubahan metadata dan chunk konsisten. Gunakan hash/model untuk reindex idempotent; panggil guard AI sebelum konten keluar.

## Kriteria terima

- [ ] Viewer gagal menulis; pemilik/reviewer/admin tunduk matriks aksi termasuk perubahan kategori/klasifikasi.
- [ ] md/txt sederhana terbaca dan pratinjau aman; PDF/format lain ditolak tanpa parser baru.
- [ ] Publish sukses menghasilkan versi immutable/current pointer/chunk lengkap; retry dan konflik revision tidak menggandakan versi atau menimpa perubahan lain.
- [ ] Test scan: pola kredensial/NIK sintetis, konfirmasi revisi, konten berubah membatalkan konfirmasi.
- [ ] Test kegagalan embed/DB menjaga versi published lama dan tidak mencampur chunk versi.
- [ ] Anchor heading bertingkat/duplikat stabil, memakai aturan sama di chunker dan viewer; sitasi dapat menuju bagian.
- [ ] Payload script/HTML berbahaya tersanitasi, termasuk snippet; server tidak mengirim parser ke browser.
- [ ] Detail tak berizin 404 generik; audit restricted/secret dicatat.
- [ ] Metadata/kedaluwarsa/state/375 px diuji; tidak ada daftar versi, rollback, feedback dokumen, approval atau object storage.

Bukti: unit/integrasi pipeline, test izin, screenshot viewer dan form.
