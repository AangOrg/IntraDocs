# T-010 — Halaman hasil pencarian

Hari 4 · orang B · perkiraan setengah hari

## Tujuan

Layar 2 mockup, memakai endpoint dari T-009.

## Prasyarat

T-002 (AppShell) sudah ada. T-009 boleh berjalan paralel — pakai data tiruan dulu bila endpoint belum siap.

## Baca dulu

`docs/ui-inventory.md` · `docs/api-contract.md`

## Berkas yang disentuh

`app/cari/page.tsx` · `components/search/*` · `components/ui/*` (pakai ulang, jangan bikin baru kalau sudah ada)

## Langkah

1. Kotak pencarian dengan kueri di URL, supaya hasil bisa dibagikan lewat tautan.
2. Baris hasil: judul, cuplikan, kategori, label, lencana klasifikasi, lencana Terverifikasi, tanggal diperbarui.
3. Filter kategori di samping. **Hanya kategori** — filter lain ada di urutan potong.
4. Baris ringkasan di atas hasil: jumlah hasil dan durasi, dihitung sungguhan dari respons, bukan angka mockup.
5. Tombol "Lanjutkan di chat" yang membawa kueri ke `/ai-assistant`.
6. Tiga keadaan wajib: tidak ada hasil, sedang memuat, gagal memuat. Keadaan kosong menyebutkan bahwa hasil dibatasi hak akses.

## Kriteria terima

- Mencari lalu memuat ulang halaman menghasilkan tampilan yang sama.
- Filter kategori mengubah hasil tanpa memuat ulang seluruh halaman.
- Terlihat wajar di lebar 375 px: sidebar filter menutup, hasil tetap terbaca.
- Tidak ada angka pengisi dari mockup yang tersisa di kode.

## Test

Satu test komponen untuk keadaan kosong dan keadaan gagal. Sisanya diperiksa manual.

## Di luar ruang lingkup

Blok jawaban AI di halaman pencarian — itu ada di layar 2 mockup, tapi milik T-011/T-012 dan hanya dikerjakan kalau hari 5 selesai lebih cepat. Pengurutan, penomoran halaman, riwayat pencarian.
