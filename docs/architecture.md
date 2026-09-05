# Arsitektur IntraDocs

Satu aplikasi Next.js 15 App Router, TypeScript strict, satu Postgres 16 dengan pgvector dan full-text. ADR-0012 menyelaraskan dokumen aktif; ADR lama tetap rekam keputusan historis.

## Runtime dan kepemilikan

- Next.js `output: 'standalone'`, route handler `runtime = 'nodejs'`; tanpa Edge, local filesystem, SDK storage vendor, Docker, atau queue pada MVP.
- Deploy Vercel + Neon pada region yang sama. Tidak ada dokumen perusahaan sungguhan.
- Auth.js kredensial; guard server memuat status aktif, role, dan cakupan terkini. Kontrak login adalah facade tipis Auth.js, bukan sistem session buatan sendiri.
- Rute halaman di `docs/ui-inventory.md`. Rute AI tunggal `POST /api/ai/ask`; kontrak payload/SSE di `docs/api-contract.md`.
- Modul: `lib/documents`, `lib/rbac`, `lib/search`, `lib/ai`, skema input `lib/schemas/`.

## Basis data — sebelas tabel pada akhir T-003

| Tabel | Field/kontrak minimum |
| --- | --- |
| user | id, email unik, password_hash, name, role, unit, category_scope array ID/null, is_active, timestamps; tanpa clearance |
| category | id, name, slug unik, parent_id nullable, sort_order; data MVP satu tingkat |
| label | id, name, slug unik, color |
| document | id, title, slug unik global, category_id, classification, owner_unit, owner_user_id, status, draft_markdown, current_version_id nullable, review_period_days, verified_at, revision, timestamps |
| document_label | document_id + label_id unik |
| document_version | id, document_id, version integer, content, content_hash, created_by, published_at; immutable, unik document_id/version |
| chunk | id, document_id, document_version_id, category_id, classification, owner_unit, heading_path array, anchor, content, token_count, embedding vector(1024), embedding_model, content_hash, search_vector tsvector |
| audit_log | id, actor_id, action, target_type, target_id, metadata tanpa isi, created_at |
| ai_query | id, user_id, conversation_id nullable, request_id nullable unique per user, request_status pending/completed/failed, response_message_id nullable, kind search/ask, question, role snapshot, result_count, cited_chunk_ids, abstained nullable, retrieval_mode, latency_ms, first_token_ms nullable, error_code nullable, feedback nullable, feedback_note nullable, created_at |
| conversation | id, user_id, title, scope JSON, timestamps |
| message | id, conversation_id, role user/assistant, content, cited_chunk_ids, created_at |

FK dan indeks ditulis melalui migrasi Drizzle. Indeks GIN search_vector, HNSW embedding, kategori/klasifikasi pada document/chunk, dan conversation_id/created_at pada message. Tidak ada tabel invite, job, review, document_file, document_grant, document_view.

Draft ada pada `document.draft_markdown`; isi published berasal dari `document_version` yang ditunjuk current_version_id. Versi lama bukan endpoint UI MVP. Chunk menduplikasi kategori/klasifikasi untuk filter cepat, tetapi query juga memeriksa induk published/current_version_id agar draft, versi lama, atau salinan izin basi tidak lolos. Keamanan lebih penting daripada klaim tanpa join.

## Dua jalur data

**Publikasi (T-006):** input `.md`/`.txt` jadi teks → validasi dan can() → scan sensitif → hitung hash/chunk/anchor → guard klasifikasi → embed → transaksi: periksa ulang revision dan izin, tulis versi, chunk, metadata lalu pindah current_version_id. Kegagalan embedding tidak mempublikasikan versi setengah jadi; versi published sebelumnya tetap utuh. Revisi bersaing menghasilkan 409, tidak menimpa diam-diam.

Dokumen/reindex memakai pipeline yang sama. Metadata kategori/klasifikasi juga memperbarui chunk secara atomik; selalu periksa induk di query. T-006 menyediakan `embed()` lewat AiProvider dan `pnpm reindex`; T-009 memakainya, bukan membuat pipeline kedua.

**Pencarian (T-009):** query full-text `simple` dengan ts_rank_cd + embedding query/pgvector → kedua jalur diberi filter izin dan published sebelum LIMIT → RRF k=60 → deduplikasi dokumen. Ini full-text PostgreSQL, bukan BM25 native. Jika embed gagal/timeout, kembalikan hasil full-text dengan mode keyword; tidak memanggil chat().

**Jawaban (T-011):** autentikasi/ownership → validasi izin seluruh sumber riwayat → kondensasi maksimal dua putaran → retrieval yang sama dengan tambahan scope dan AI_MAX_CLASSIFICATION → threshold/no source: abstain; selain itu generasi bersitasi → SSE → simpan pesan/log. Guard sebelum kondensasi maupun generasi.

## Batas keamanan

Semua pembacaan dokumen/sumber/hitungan memakai `visibleDocumentsFilter(user)`; aksi memakai `can(user, action, resource)`. Scope mempersempit izin, tidak menambah izin. Riwayat yang kehilangan izin ditolak utuh dengan 404, bukan menampilkan jawaban lama tanpa sumber. Detail di matriks RBAC.

Audit dokumen sensitif dan log query tidak opsional. Tidak ada isi sensitif di error/log teknis. Respons unauthorized document/conversation sama dengan resource tidak ditemukan.

## Skala dan pengukuran

Skala rancangan 10.000 dokumen/2.000 pengguna, bukan benchmark yang sudah terbukti. Tinjau ulang sekitar 200.000 dokumen atau 50 qps dengan pengukuran nyata. Target mutu/latensi hanya di scope; cara mengukurnya di eval README. Tidak mengklaim layak produksi sampai pengujian dan persyaratan deployment nyata diperiksa.
