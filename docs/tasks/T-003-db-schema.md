# T-003: Skema DB + migrasi

- Pemilik: Orang A · Minggu 0 · Estimasi: 1 hari

## Tujuan

Skema yang mendukung RBAC, versi, dan search — disepakati bersama sebelum kedua orang
bekerja paralel.

> **Ini titik sinkronisasi paling penting di proyek ini.** Bahas berdua sebelum menulis
> migrasi. Perubahan skema setelah minggu pertama jauh lebih mahal.

## Konteks

Baca `docs/architecture.md`, `docs/rbac-matrix.md`, `docs/api-contract.md`,
`docs/adr/0006-embedding-dimension.md`, `docs/adr/0009-rbac-cakupan-kategori.md`,
`docs/adr/0010-chat-multi-turn.md`.

## Lingkup

Sebelas tabel, tidak lebih. Jumlah ini mengikat: ADR-0007 memotong job queue, object
storage, alur invite, dan approval berjenjang dari MVP, sehingga tabel penopangnya tidak
boleh ikut dibuat "selagi lewat".

- `user` — email, `password_hash`, name, `role`, `unit`, `category_scope` (`NULL` berarti
  semua kategori, ADR-0009), `is_active`, timestamps
- `category` — slug, urutan, `parent_id` (kolom sudah ada, tetapi MVP memakai satu tingkat)
- `label` — name, slug, color
- `document` — title, slug, `category_id`, `classification`, `owner_unit`, `owner_user_id`,
  `status`, `current_version_id`, `review_period_days`, `verified_at`, timestamps
- `document_label` — relasi banyak-ke-banyak
- `document_version` — **immutable**: `document_id`, `version`, `content` (Markdown +
  frontmatter), `content_hash`, `created_by`, `published_at`
- `chunk` — `document_version_id`, `heading_path`, `content`, `token_count`,
  `embedding vector(1024)`, `embedding_model`, `content_hash`, ditambah salinan
  `classification` dan `owner_unit`
- `audit_log` — actor, action, `target_type`, `target_id`, metadata, `created_at`
- `ai_query` — user, question, `cited_chunk_ids`, `abstained`, latency, feedback
- `conversation` — user, title, timestamps (ADR-0010)
- `message` — `conversation_id`, role, content, `cited_chunk_ids`, `created_at` (ADR-0010)

Klasifikasi tertinggi yang boleh dibaca **diturunkan dari `user.role`** sesuai tabel di
`docs/rbac-matrix.md`. Tidak ada kolom clearance per pengguna: dua sumber kebenaran untuk
satu aturan visibilitas dilarang oleh ADR-0004.

Tidak dibuat di MVP, dan jangan ditambahkan tanpa ADR baru: `invite`, `document_file`,
`document_grant`, `job`, `review`, `document_view`.

Index:

- GIN `tsvector` pada isi chunk, dengan konfigurasi teks yang sesuai
- HNSW pada `chunk.embedding`
- Index komposit yang mendukung predikat izin: `(classification, owner_unit, status)`
- Index pada `message (conversation_id, created_at)`

## Acceptance criteria

- [ ] Migrasi jalan dari database kosong tanpa error
- [ ] Migrasi bisa dijalankan ulang dengan aman (idempotent)
- [ ] Ekstensi `pgvector` dibuat oleh migrasi, bukan manual
- [ ] Tepat sebelas tabel; tidak ada tabel di luar daftar di atas
- [ ] `user.category_scope` dan `user.is_active` ada sejak migrasi pertama (ADR-0009)
- [ ] `conversation` dan `message` ada sejak migrasi pertama (ADR-0010)
- [ ] `document_version` tidak bisa di-update (dijaga trigger atau konvensi + test)
- [ ] Enum untuk `role`, `classification`, `status`
- [ ] Semua foreign key punya aturan `ON DELETE` yang eksplisit dan dipikirkan
- [ ] Ada `EXPLAIN ANALYZE` di deskripsi PR untuk query search dengan predikat izin

### Fixture yang dibutuhkan test RBAC

Ketujuh test wajib di `docs/rbac-matrix.md` ditulis pada hari 2. Skema dan fixture harus
sudah bisa menopangnya tanpa migrasi tambahan.

- [ ] Fixture menyediakan tepat **enam** pengguna sesuai seluruh tabel `docs/rbac-matrix.md`:
      **lima aktif dan satu nonaktif**, bukan enam pengguna ditambah satu lagi
- [ ] Fajar Nugroho adalah fixture nonaktif: `role = viewer`, unit **Finance (mitra)**,
      cakupan kategori **SOP & Proses Bisnis**, `is_active = false` — prasyarat
      `tests/rbac/inactive-user.spec.ts`, sesuai layar 8 mockup
- [ ] **Viewer Demo** adalah akun sintetis aktif: `role = viewer`, unit **Demo**,
      `category_scope = NULL`, `is_active = true` — kontrol positif login dan akun demo
- [ ] Fixture menyediakan dokumen `restricted` di **dua kategori berbeda** — prasyarat
      `tests/rbac/category-scope.spec.ts`, agar cakupan sempit Dwi Kurniawan bisa dibuktikan
- [ ] Fixture menyediakan dokumen pada keempat klasifikasi dan dua `owner_unit` berbeda
- [ ] `audit_log` bisa ditulis dan dibaca oleh test — prasyarat `tests/rbac/audit-log.spec.ts`

## Batasan

- SQL standar saja — tanpa fitur khusus vendor (ADR-0005).
- Jangan menyimpan isi dokumen di `audit_log`, hanya referensi.
- `chunk` sengaja didenormalisasi; jelaskan alasannya di komentar skema.
- `drizzle-kit generate`, bukan `drizzle-kit push`.

## Cara menguji

Jalankan migrasi pada database kosong, sisipkan data contoh, jalankan query katalog dan
search dengan predikat izin, periksa rencana query-nya menggunakan index.
