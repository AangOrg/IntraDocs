# Arsitektur IntraDocs

Satu aplikasi Next.js, satu basis data Postgres. Tidak ada komponen lain di MVP.

## Peta layar 11 mockup ke implementasi kita

| Komponen di mockup | Implementasi kita | Catatan |
|---|---|---|
| Portal Web (Help Center) | Rute publik-internal Next.js | Dibangun |
| Konsol Admin | Ditunda | Fase 2 |
| AI Assistant | `/ai-assistant` + `/api/chat` | Dibangun |
| Mobile Web / PWA | Tata letak responsif saja | Service worker & luring tidak dibangun |
| API Gateway + SSO/OIDC | Route handler Next.js + Auth.js kredensial | OIDC di balik flag, ADR-0003 |
| Layanan Dokumen | `lib/documents` | Dibangun |
| Mesin Approval | Ditunda | Fase 2, sesuai roadmap mockup |
| Layanan RBAC | `lib/rbac` | Dibangun, jadi satu titik |
| Layanan Pencarian | Postgres `tsvector` | Digabung ke Postgres |
| Layanan RAG | `lib/ai` + `/api/chat` | Dibangun |
| Pemroses Berkas & OCR | Ditunda | MVP hanya `.md` dan `.txt` |
| Notifikasi | Ditunda | Fase 2 |
| PostgreSQL | Postgres 16 (Neon) | Sama |
| Object Storage | Ditunda | Isi dokumen disimpan sebagai teks di DB |
| Vector DB | `pgvector` di Postgres yang sama | Penyederhanaan sadar |
| Search Index | `tsvector` di Postgres yang sama | Penyederhanaan sadar |
| Audit Log | Tabel `audit_log` | Dibangun |

Dua penyederhanaan terakhir adalah keputusan, bukan kelalaian: pada 1.284 dokumen (angka mockup sendiri) maupun pada target rancangan 10.000 dokumen, memisahkan indeks menambah dua sistem untuk dirawat tanpa menambah kemampuan. Ambang untuk meninjau ulang ada di bagian Skala.

## Bentuk runtime

- Next.js 15 App Router, TypeScript strict, `output: 'standalone'`.
- Semua route handler `export const runtime = 'nodejs'`. Tidak ada Edge runtime.
- Tidak ada penulisan ke filesystem lokal. Tidak ada SDK spesifik vendor.
- Jawaban AI dialirkan lewat SSE: `sources` → `token` → `done` → `error`.

Aturan portabilitas ini yang membuat aplikasi bisa pindah dari Vercel ke server perusahaan tanpa menulis ulang. Lihat ADR-0005.

## Basis data — 11 tabel

| Tabel | Peran |
|---|---|
| `user` | Identitas, `role`, `clearance`, `unit`, `category_scope`, `is_active` |
| `category` | Kategori, satu tingkat di MVP |
| `label` | Label bebas |
| `document` | Metadata + isi Markdown, `classification`, `status`, `owner_unit`, `review_period_days` |
| `document_label` | Relasi banyak-ke-banyak |
| `document_version` | Riwayat isi, tanpa UI di MVP |
| `chunk` | Potongan + `embedding vector(1024)` + `embedding_model` + `content_hash` + `heading_path` |
| `audit_log` | Unggah, setujui, baca dokumen terbatas, unduh |
| `ai_query` | Satu baris per pencarian dan per pertanyaan AI |
| `conversation` | Sesi percakapan AI |
| `message` | Giliran percakapan |

`chunk` menyimpan salinan `classification` dan `owner_unit` dari dokumen induknya. Denormalisasi ini disengaja: filter izin bisa dijalankan di satu tabel saat pencarian vektor, tanpa join yang mematikan indeks.

## Dua jalur data

**Publikasi.** Simpan dokumen → hitung `content_hash` → kalau berubah, potong per heading → embed serentak → tulis `chunk`. Hanya dokumen `published` yang diindeks. Embedding dilakukan langsung saat publikasi, bukan lewat antrean — pada volume ini antrean hanya menambah bagian yang bisa rusak.

**Menjawab.** Pertanyaan → (kalau lanjutan) tulis ulang jadi mandiri → jalankan pencarian kata kunci dan pencarian vektor secara paralel, keduanya sudah dibatasi `visibleDocumentsFilter` → gabungkan dengan RRF → ambil potongan teratas → susun prompt → alirkan jawaban dengan sitasi → catat ke `ai_query`.

Satu retrieval per pertanyaan. Tidak ada tool calling, tidak ada agent. Alasannya di ADR-0010: RAG satu langkah bisa dievaluasi secara deterministik, agent tidak.

## Titik izin tunggal

Semua jalur baca — katalog, pencarian kata kunci, pencarian vektor, retrieval RAG — memanggil `visibleDocumentsFilter(user)` yang mengembalikan fragmen SQL. Tidak ada jalur kedua. Kalau ada kode yang mengambil dokumen tanpa memanggilnya, itu bug keamanan, bukan pilihan gaya.

Dokumen yang tidak boleh dilihat menghasilkan **404, bukan 403**. 403 membocorkan keberadaan dokumen.

## Skala dan kapan berubah

Rancangan ini nyaman sampai sekitar 10.000 dokumen dan 2.000 pengguna. Tinjau ulang bila melewati ~200.000 dokumen atau ~50 kueri per detik. Yang pertama pecah kemungkinan besar pencarian vektor; jalur keluarnya adalah indeks HNSW terpisah atau vector DB khusus — dan karena semua akses lewat satu fungsi filter, perpindahan itu terlokalisasi.
