# T-008: Seed 20–30 dokumen sintetis

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

Baca `docs/adr/0003-ai-provider-and-data-classification.md` dan `docs/eval/README.md`.
Ambil gaya penulisan, penomoran, dan struktur dari mockup.

## Lingkup

- 20–30 dokumen Markdown di `db/seed/documents/`, dengan YAML frontmatter lengkap.
- Tersebar di keenam kategori dan keempat level klasifikasi.
- Gaya mengikuti contoh di mockup: `SOP-IT-014 Manajemen Identitas`,
  `Kebijakan Password & Autentikasi Perusahaan (ISMS-POL-07)`, `Matriks SLA Layanan IT 2026`,
  `Runbook VPN`.
- Sertakan struktur yang menantang: tabel, list bertingkat, blok kode, referensi antar
  dokumen, nomor versi.
- Sertakan beberapa dokumen yang **saling mirip** (misal dua kebijakan password dari tahun
  berbeda) untuk menguji ketajaman retrieval.
- Beberapa dokumen sengaja dibuat kadaluarsa untuk menguji peringatan tinjau ulang.
- `scripts/seed.ts` — idempotent: memuat dokumen, kategori, label, pengguna contoh untuk
  setiap kombinasi role/clearance.

## Acceptance criteria

- [ ] `pnpm seed` berjalan dari database kosong maupun database yang sudah terisi
- [ ] Keempat klasifikasi dan keenam kategori terwakili
- [ ] Panjang dokumen bervariasi: ada yang satu halaman, ada yang panjang
- [ ] Minimal 30 pertanyaan eval bisa dijawab dari korpus ini
- [ ] **Nol data Telkom asli.** Semua nama orang, sistem, dan angka bersifat fiktif
- [ ] Frontmatter valid dan lolos validasi zod

## Batasan

- Jangan menyalin dokumen internal asli, bahkan sebagian, bahkan untuk pengembangan lokal.
- Jangan memakai nama karyawan sungguhan.
- Tulis dalam bahasa Indonesia, dengan gaya dokumentasi korporat.

## Cara menguji

Jalankan seed, jelajahi katalog sebagai role berbeda, lalu baca beberapa dokumen. Kalau
terasa seperti dokumentasi sungguhan, task ini berhasil. Kalau terasa seperti data uji,
belum.
