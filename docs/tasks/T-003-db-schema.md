# T-003: Skema DB + migrasi

- Pemilik: Orang A · Minggu 0 · Estimasi: 1 hari

## Tujuan

Skema yang mendukung RBAC, versi, dan search — disepakati bersama sebelum kedua orang
bekerja paralel.

> **Ini titik sinkronisasi paling penting di proyek ini.** Bahas berdua sebelum menulis
> migrasi. Perubahan skema setelah minggu pertama jauh lebih mahal.

## Konteks

Baca `docs/architecture.md`, `docs/rbac-matrix.md`, `docs/api-contract.md`,
`docs/adr/0006-embedding-dimension.md`.

## Lingkup

Tabel:

- `user` — email, `password_hash`, name, `role`, `clearance`, `unit`, timestamps
- `invite` — token, email, role, clearance, unit, `expires_at`, `accepted_at`
- `category` — hierarkis, `parent_id`, slug, urutan
- `label` — name, slug, color
- `document` — title, slug, `category_id`, `classification`, `owner_unit`, `owner_user_id`,
  `status`, `current_version_id`, `review_period_days`, `verified_at`, timestamps
- `document_label` — relasi banyak-ke-banyak
- `document_version` — **immutable**: `document_id`, `version`, `content` (Markdown +
  frontmatter), `content_hash`, `created_by`, `published_at`
- `document_file` — berkas asli: `file_key`, `mime`, `size`, `sha256`
- `document_grant` — akses eksplisit per pengguna atau per unit
- `chunk` — `document_version_id`, `heading_path`, `content`, `token_count`,
  `embedding vector(1024)`, `embedding_model`, `content_hash`, ditambah salinan
  `classification` dan `owner_unit`
- `job` — type, payload, status, attempts, `run_after`, `last_error`
- `review` — `document_id`, `reviewer_id`, decision, note, timestamps
- `audit_log` — actor, action, `target_type`, `target_id`, metadata, `created_at`
- `ai_query` — user, question, `cited_chunk_ids`, `abstained`, latency, feedback
- `document_view` — untuk peringkat "paling banyak dibaca"

Index:

- GIN `tsvector` pada isi chunk, dengan konfigurasi teks yang sesuai
- HNSW pada `chunk.embedding`
- Index komposit yang mendukung predikat izin: `(classification, owner_unit, status)`
- Index pada `job (status, run_after)`

## Acceptance criteria

- [ ] Migrasi jalan dari database kosong tanpa error
- [ ] Migrasi bisa dijalankan ulang dengan aman (idempotent)
- [ ] Ekstensi `pgvector` dibuat oleh migrasi, bukan manual
- [ ] `document_version` tidak bisa di-update (dijaga trigger atau konvensi + test)
- [ ] Enum untuk `role`, `classification`, `status`, `job.status`
- [ ] Semua foreign key punya aturan `ON DELETE` yang eksplisit dan dipikirkan
- [ ] Ada `EXPLAIN ANALYZE` di deskripsi PR untuk query search dengan predikat izin

## Batasan

- SQL standar saja — tanpa fitur khusus vendor (ADR-0005).
- Jangan menyimpan isi dokumen di `audit_log`, hanya referensi.
- `chunk` sengaja didenormalisasi; jelaskan alasannya di komentar skema.

## Cara menguji

Jalankan migrasi pada database kosong, sisipkan data contoh, jalankan query katalog dan
search dengan predikat izin, periksa rencana query-nya menggunakan index.
