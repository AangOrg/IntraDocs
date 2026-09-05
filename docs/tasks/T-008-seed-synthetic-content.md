# T-008: Seed 20–25 dokumen sintetis

- Pemilik: Orang B · Minggu 1 · Estimasi: 1 hari (bisa dicicil)

## Tujuan

Konten yang realistis untuk pengembangan, demo, dan eval — **tanpa pernah menyentuh dokumen
internal Telkom yang asli.**

## Kenapa task ini setara pentingnya dengan task kode

Tiga alasan. Pertama, demo dengan data placeholder "Lorem ipsum" tidak akan meyakinkan
siapa pun untuk memakai platform ini. Kedua, kualitas retrieval tidak bisa dinilai tanpa
korpus yang menyerupai korpus nyata — termasuk kemiripan antar dokumen yang membingungkan
search. Ketiga, dokumen ini sekaligus menjadi eval set (`docs/eval/`).

## Konteks

Baca `docs/adr/0003-ai-provider-and-data-classification.md`, `docs/eval/README.md`, dan tabel
pengguna seed di `docs/rbac-matrix.md` sebagai acuan tunggal identitas, role, cakupan, dan status.
Ambil gaya penulisan, penomoran, dan struktur dari mockup.

## Lingkup

- 20–25 dokumen Markdown di `db/seed/documents/`, dengan YAML frontmatter lengkap. Angka ini
  ditetapkan di `docs/scope-mvp.md`; lebih banyak bukan lebih baik, karena setiap dokumen
  harus tetap layak dibaca.
- Tersebar di keenam kategori dan keempat level klasifikasi.
- Gaya mengikuti contoh di mockup: `SOP-IT-014 Manajemen Identitas`,
  `Kebijakan Password & Autentikasi Perusahaan (ISMS-POL-07)`, `Matriks SLA Layanan IT 2026`,
  `Runbook VPN`.
- Sertakan struktur yang menantang: tabel, list bertingkat, blok kode, referensi antar
  dokumen, nomor versi.
- Sertakan beberapa dokumen yang **saling mirip** (misal dua kebijakan password dari tahun
  berbeda) untuk menguji ketajaman retrieval.
- Beberapa dokumen sengaja dibuat kadaluarsa untuk menguji peringatan tinjau ulang.
- Heading bertingkat sampai tiga tingkat pada minimal tiga dokumen, agar `chunk.heading_path`
  dan sitasi "Bagian 2.1.2" punya bahan uji yang sungguhan.
- Minimal dua dokumen `restricted` pada dua kategori berbeda, agar cakupan kategori bisa diuji.
- `scripts/seed.ts` — idempotent: memuat dokumen, kategori, label, dan tepat **enam** pengguna
  sesuai seluruh tabel `docs/rbac-matrix.md`: lima aktif dan **Fajar Nugroho nonaktif**
  (`viewer`, unit Finance (mitra), cakupan SOP & Proses Bisnis, `is_active = false`).
  **Viewer Demo** adalah akun sintetis aktif pengganti untuk demo dan kontrol positif login,
  bukan pengguna ketujuh. Tidak ada pengguna nonaktif tambahan di luar tabel tersebut.

## Acceptance criteria

- [ ] `pnpm seed` berjalan dari database kosong maupun database yang sudah terisi
- [ ] Keempat klasifikasi dan keenam kategori terwakili
- [ ] Panjang dokumen bervariasi: ada yang satu halaman, ada yang panjang
- [ ] Sepuluh pertanyaan eval di `docs/eval/questions.jsonl` bisa dijawab dari korpus ini,
      termasuk satu pertanyaan yang sengaja **di luar** korpus untuk menguji jalur abstain
- [ ] Tepat enam pengguna seed sesuai `docs/rbac-matrix.md`: lima aktif dan satu nonaktif,
      dengan Dwi Kurniawan tetap reviewer aktif bercakupan Keamanan Informasi saja
- [ ] Fajar Nugroho tetap `viewer` nonaktif dengan unit Finance (mitra) dan cakupan SOP & Proses
      Bisnis setelah seed dijalankan ulang; tidak diaktifkan kembali sebagai akun demo
- [ ] **Viewer Demo** tetap `viewer` aktif dengan unit Demo dan `category_scope = NULL` setelah
      seed dijalankan ulang; akun sintetis ini adalah pengganti viewer aktif untuk demo
- [ ] **Nol data Telkom asli.** Semua nama orang, sistem, dan angka bersifat fiktif
- [ ] Frontmatter valid dan lolos validasi zod
- [ ] Tidak ada angka dari mockup yang ikut tersalin sebagai fakta (1.284 dokumen, 318
      pengguna, 96% akurasi, 42 kategori, 68 label — semuanya pengisi tempat)

## Batasan

- Jangan menyalin dokumen internal asli, bahkan sebagian, bahkan untuk pengembangan lokal.
- Jangan memakai nama karyawan sungguhan.
- Tulis dalam bahasa Indonesia, dengan gaya dokumentasi korporat.

## Cara menguji

Jalankan seed, jelajahi katalog sebagai role berbeda, lalu baca beberapa dokumen. Kalau
terasa seperti dokumentasi sungguhan, task ini berhasil. Kalau terasa seperti data uji,
belum.
