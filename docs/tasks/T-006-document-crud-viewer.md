# T-006: CRUD dokumen + viewer Markdown

- Pemilik: Orang B · Minggu 1 · Estimasi: 1,5 hari

## Tujuan

Dokumen bisa dibuat, diedit, dan dibaca dengan tampilan yang membangun kepercayaan.

## Konteks

Baca `docs/api-contract.md`, `docs/ui-inventory.md`,
`docs/adr/0002-content-source-of-truth.md`. Lihat screen s3 pada mockup dan perbarui
`docs/ui-inventory.md` bila isinya berbeda dari perkiraan.

## Lingkup

- Komponen bersama lebih dahulu: `AppShell`, `ClassificationBadge`, `MarkdownViewer`,
  `EmptyState`, `ErrorState`, `PermissionDeniedState`.
- Editor Markdown sederhana dengan pratinjau terpisah. **Bukan WYSIWYG.**
- Render `remark` → `rehype` → HTML **di server**, dengan `rehype-sanitize`, di-cache per
  `document_version`.
- Daftar isi otomatis dari heading, dengan penanda posisi saat menggulir.
- Panel metadata: pemilik, unit, kategori, label, klasifikasi, versi, tanggal update,
  badge terverifikasi.
- Peringatan bila dokumen melewati `review_period_days`.
- Daftar versi dengan tautan ke versi terdahulu (read-only).
- Buat dan edit draft, simpan versi baru saat dipublikasikan.

## Di luar lingkup

Upload berkas (T-009/T-010), alur review (T-012), UI diff antar versi (v1.1).

## Acceptance criteria

- [ ] Membuat draft, mengedit, dan mempublikasikan menghasilkan baris `document_version` baru
- [ ] Versi terdahulu tetap bisa dibaca dan tidak berubah
- [ ] HTML berbahaya di dalam Markdown ter-sanitasi (ada test dengan payload `<script>`)
- [ ] Blok kode punya penyorotan sintaks dan tombol salin
- [ ] Semua metadata dari mockup tampil
- [ ] Halaman dokumen mencapai LCP < 2,5 s dan JS < 150 KB gzip
- [ ] Kelima state UI ada: loading, kosong, error, tanpa izin, tidak ditemukan
- [ ] Setiap jalur baca melewati `visibleDocumentsFilter`

## Batasan

- Render di server, bukan di klien. Jangan mengirim parser Markdown ke browser.
- Client component hanya untuk bagian yang benar-benar interaktif.
- Pakai token dari T-002. Nol warna hardcoded.

## Cara menguji

Buat dokumen berisi Markdown yang rumit: tabel, list bertingkat, blok kode, dan satu tag
`<script>`. Publikasikan, edit, publikasikan lagi, lalu buka versi lama. Ukur bundle-nya.
