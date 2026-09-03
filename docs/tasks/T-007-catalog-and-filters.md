# T-007: Katalog, kategori, label, Help Center

- Pemilik: Orang B · Minggu 1 · Estimasi: 1 hari

## Tujuan

Pengguna bisa menemukan dokumen dengan menjelajah, bukan hanya dengan mencari — dan hanya
melihat yang boleh dilihatnya.

## Konteks

Baca `docs/ui-inventory.md`, `docs/api-contract.md`, `docs/rbac-matrix.md`. Lihat screen s1
pada mockup.

## Lingkup

- Halaman Help Center: hero dengan search, enam kartu kategori beserta jumlah dokumen,
  chip topik populer, bagian "Paling banyak dibaca bulan ini", banner AI.
- Halaman kategori dengan sub-kategori dan daftar dokumen.
- Filter: kategori, label, klasifikasi, terakhir diperbarui, status.
- Filter tersimpan di URL query string agar bisa dibagikan dan di-bookmark.
- Pagination berbasis cursor.
- Sidebar: Navigasi, Kontribusi, Bantuan — sesuai mockup.
- Peringkat "paling banyak dibaca" dari `document_view`.

## Acceptance criteria

- [ ] Jumlah dokumen per kategori **menghormati izin pengguna** — dua pengguna berbeda dapat
      angka berbeda
- [ ] Filter bisa digabungkan dan tercermin di URL
- [ ] Refresh halaman mempertahankan filter
- [ ] State kosong menjelaskan langkah berikutnya, bukan hanya "tidak ada data"
- [ ] Angka statistik pada hero berasal dari data nyata, atau tidak ditampilkan
- [ ] Bisa dinavigasi sepenuhnya dengan keyboard
- [ ] Rendering di server, tanpa state manager global

## Batasan

- **Jangan menampilkan angka "96% akurasi AI" dari mockup.** Angka yang tidak diukur adalah
  angka yang menyesatkan (lihat `docs/eval/README.md`).
- Jumlah dokumen tidak boleh dihitung dengan cara yang membocorkan keberadaan dokumen
  terlarang.

## Cara menguji

Login sebagai `viewer` dengan clearance `internal` lalu sebagai `admin`. Bandingkan jumlah
per kategori dan daftar dokumennya — keduanya harus berbeda dan konsisten dengan matriks RBAC.
