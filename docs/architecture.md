# Arsitektur

## Prinsip

1. **Satu runtime, satu database, satu deployable.** Setiap komponen tambahan harus
   membayar biayanya sendiri dalam bentuk nilai yang jelas.
2. **Portabilitas dari format, bukan dari storage.** Konten adalah Markdown + YAML
   frontmatter, jadi kita tidak terikat pada database maupun vendor mana pun.
3. **Keamanan sebagai kode, bukan sebagai janji.** Aturan akses dan batas klasifikasi
   ditegakkan oleh query dan config, lalu diuji otomatis.
4. **Degradasi berjenjang.** AI adalah lapisan di atas search, bukan gerbangnya.

## Gambaran

```
                    Browser (RSC, JS client minimal)
                              |
              +---------------+---------------+
              |     Next.js 15 (App Router)   |
              |  route handler + server action|
              +---------------+---------------+
                  |           |            |
           +------+---+  +----+-----+  +---+-----------+
           | Postgres |  | Object   |  | AI provider   |
           | 16 +     |  | storage  |  | (abstraksi)   |
           | pgvector |  | S3-compat|  | chat + embed  |
           +----------+  +----------+  +---------------+
                  ^
                  |
           +------+-------------------------+
           | Job worker                     |
           | Vercel Cron -> /api/jobs/tick  |
           |   atau loop proses di Docker   |
           +--------------------------------+
```

Postgres memikul tiga peran sekaligus: relational, full-text search (`tsvector`), dan
vector search (`pgvector`). Itu sebabnya tidak ada Elasticsearch dan tidak ada vector
database terpisah — pada skala 10.000 dokumen, dua komponen itu hanya menambah biaya ops.

## Lapisan

| Lapisan | Isi | Aturan |
| --- | --- | --- |
| `app/` | Route, layout, server action | Tipis. Tanpa business logic. |
| `components/` | Komponen UI | Client component hanya bila perlu interaktivitas. |
| `lib/` | Logic murni: rbac, chunker, rrf, parser, ai provider, schemas | Bisa diuji tanpa DB. Di sinilah unit test hidup. |
| `db/` | Skema Drizzle + migrasi | Perubahan skema selalu lewat file migrasi. |
| `scripts/` | seed, export-to-git, reindex, eval | Bisa dijalankan sendiri. |

## Alur ingest

```
upload
  -> simpan berkas asli ke object storage + SHA-256
  -> job convert       : DOCX (mammoth) | PDF text-layer (unpdf) | MD/TXT (passthrough)
  -> job normalize     : susun YAML frontmatter, bersihkan header/footer/nomor halaman
  -> draft             : MANUSIA memeriksa & memperbaiki hasil konversi   <- wajib
  -> in_review         : reviewer kategori atau admin
  -> published         : baris document_version baru, immutable
  -> job index         : chunk -> embed -> simpan vector
```

Kenapa pemeriksaan manusia wajib: konversi PDF ke Markdown tidak pernah sempurna,
terutama tabel dan list bertingkat. Publikasi otomatis akan mencemari knowledge base,
dan AI yang bersumber dari konten rusak akan terdengar meyakinkan sambil salah.

### Chunking

- Heading-aware: potong pada batas heading Markdown, gabungkan bagian yang terlalu kecil.
- Target 500–800 token, overlap sekitar 15%.
- Simpan `heading_path`, misal `SOP-IT-014 > Reset via Service Desk > Verifikasi identitas`.
  Ini yang membuat sitasi menunjuk ke **bagian**, bukan hanya ke dokumen.
- Simpan `classification` dan `owner_unit` di baris chunk (denormalisasi) supaya filter izin
  bisa dijalankan di dalam index scan tanpa join.

### Pola job

Satu tabel `job` di Postgres. Tanpa Redis, tanpa BullMQ.

- `POST /api/jobs/tick` (dilindungi `JOB_TICK_SECRET`) memproses maksimal N job lalu selesai.
- Di Vercel: dipanggil Vercel Cron setiap menit. Di Docker: loop proses memanggil fungsi
  yang sama. **Satu implementasi, dua environment.**
- Satu tick mengerjakan satu langkah kecil (konversi satu dokumen, embed satu batch) agar
  aman dari batas waktu serverless.
- Job harus idempotent dengan kunci `document_version_id`. Ada retry dengan backoff dan
  status `dead` setelah batas percobaan, plus tombol reindex manual di halaman admin.

## Alur retrieval

```
pertanyaan
  -> visibleDocumentsFilter(user)        // menghasilkan predikat SQL
  -> jalur A: full-text  (tsvector, ts_rank)      keduanya SUDAH terfilter
  -> jalur B: vector     (pgvector, cosine)
  -> Reciprocal Rank Fusion  => top-k
  -> filter AI_MAX_CLASSIFICATION         // batas kepatuhan provider
  -> skor tertinggi < threshold ? abstain : kirim ke LLM
  -> jawaban wajib bersitasi -> catat ke ai_query
```

**Kenapa hybrid, bukan vector saja.** Pertanyaan internal penuh identifier dan akronim:
`SOP-IT-014`, `ISMS-POL-07`, `SSPR`, `ADUC`. Vector search sering gagal pada pencocokan
persis semacam itu, sementara full-text unggul. Sebaliknya full-text gagal pada parafrase
("cara ganti sandi domain" vs "reset password Active Directory"). Keduanya dibutuhkan;
RRF menggabungkan peringkat tanpa perlu menyetel bobot skor yang tidak sebanding.

**Kontrak jawaban.** Jawab hanya dari konteks. Setiap klaim bersitasi. Kalau tidak ada
dasar, abstain dan tetap tampilkan dokumen yang mungkin relevan. Setiap sitasi menampilkan
klasifikasi, tanggal update, dan status verifikasi — dan memperingatkan bila dokumen sudah
melewati periode tinjau ulang.

**Prompt injection.** Isi dokumen adalah input tidak terpercaya. Konteks dibungkus
delimiter yang jelas dan diperlakukan sebagai data, bukan instruksi. AI chat tidak diberi
akses tool atau write apa pun — read-only.

## Keamanan

- Satu fungsi `visibleDocumentsFilter(user)` dipakai oleh katalog, search, dan RAG.
  Satu sumber kebenaran, tiga konsumen. Detail: `docs/rbac-matrix.md`.
- Berkas asli diakses lewat signed URL berumur pendek, bukan path publik.
- Render Markdown selalu melalui `rehype-sanitize`.
- Rate limit pada login dan endpoint AI, per pengguna.
- Audit log untuk akses dokumen `restricted`/`secret` dan untuk semua aksi admin.
- `ai_query` menyimpan pertanyaan untuk keperluan eval dan audit, dengan retensi terbatas.
- Dependency dipin, secret scanning aktif, CSP, HTTPS-only.

## Budget performa (dicek di CI, bukan diharapkan)

| Metrik | Target |
| --- | --- |
| JS terkirim per rute | < 150 KB gzip |
| LCP halaman dokumen | < 2,5 s |
| TTFB | < 500 ms |
| p95 search | < 500 ms pada 10.000 dokumen |
| p95 token pertama AI | < 3 s |

Cara mencapainya: server-first (RSC + server action), Markdown dirender ke HTML di server
dan di-cache per versi dokumen, ikon sebagai inline SVG sprite seperti pada mockup, tanpa
state manager global, tanpa icon library besar.

## Skala

Didesain untuk **10.000 dokumen dan 2.000 pengguna**. Pada skala itu, pgvector dengan index
HNSW plus full-text Postgres sangat memadai.

Jalur peningkatan bila memang dibutuhkan: aplikasi stateless → scale horizontal; tambah
worker job; read replica untuk beban baca; tambah reranker bila hit rate belum memadai.
Bila melewati ~200.000 dokumen atau ~50 QPS search, baru pertimbangkan partisi index dan
reranker terdedikasi. **Sebelum ambang itu, jangan dipikirkan.**

Skalabilitas yang sesungguhnya relevan di sini bukan RPS, melainkan skalabilitas **konten
dan kontributor**: taksonomi, kepemilikan dokumen, dan periode tinjau ulang adalah fitur
skalabilitas.

## Reliabilitas

- Search tidak bergantung pada LLM. Provider AI mati → produk masih berguna.
- Job idempotent, retry, dead-letter, dan reindex manual.
- `/api/health` memeriksa DB dan object storage.
- Structured logging (`pino`) + error tracking.
- Backup harian **dan latihan restore yang benar-benar dijalankan**. Backup yang belum
  pernah di-restore bukan backup.
- Gagal cepat saat startup bila env wajib tidak ada, dengan pesan yang jelas.
- Setiap tampilan punya state eksplisit: loading, kosong, error, tanpa izin, dan abstain.
