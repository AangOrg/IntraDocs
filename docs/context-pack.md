# Context Pack — baseline aktif

Urutan wewenang: scope-mvp > ADR bernomor > berkas ini > dokumen lain. ADR-0012 menyelaraskan baseline; ADR-0011 mengunci judul tersembunyi/404. ADR lama adalah rekam keputusan, bukan daftar task untuk dibangun ulang.

## Proyek dan batas

IntraDocs: dokumentasi internal, RBAC dan AI bersitasi; tugas magang dua orang. Kualitas menentukan rilis, bukan kalender enam hari. Mockup root intradocs-mockup_1.html berisi 11 layar; baca rentang yang perlu saja.

Seluruh konten/akun sintetis. Tidak ada dokumen Telkom asli, kredensial layanan, atau chat helpdesk asli. Empat role viewer/contributor/reviewer/admin; classification public/internal/restricted/secret; status draft/in_review/published/archived tetapi alur MVP hanya draft/published.

Satu kategori per dokumen, banyak label. Role menentukan maksimum klasifikasi tanpa user.clearance. category_scope NULL=semua, kosong=tidak ada kategori sensitif; public/internal published lintas kategori, admin global. Unit hanya metadata. Detail draft/tulis/riwayat di matriks RBAC.

## Identifier tetap

| Hal | Nama |
| --- | --- |
| Filter SQL | lib/rbac/visible-documents.ts → visibleDocumentsFilter(user): SQL |
| Aksi | lib/rbac/can.ts → can(user, action, resource) |
| Provider | lib/ai/provider.ts → embed(), chat(), AiProvider |
| AI HTTP | POST /api/ai/ask |
| Keamanan | tests/rbac/ai-retrieval-leak.spec.ts dan enam suite lain di matriks |
| Eval | docs/eval/questions.jsonl; pnpm eval --retrieval; pnpm eval |
| Perintah | pnpm typecheck, pnpm lint, pnpm test, pnpm build, pnpm seed, pnpm reindex |

Runtime Next.js 15 Node.js/standalone, Postgres16 pgvector/tsvector, Auth.js credentials, Drizzle. Sebelas tabel domain, embedding 1024, chunk 500–800 token overlap sekitar 15%. Korpus 20–25 md; sepuluh kasus eval sintetis. Angka gerbang hanya di scope; metode hanya di eval README.

Search boleh embed(), tidak chat(); full-text simple/ts_rank_cd + RRF, fallback keyword eksplisit. Publikasi/embedding/reindex T-006; retrieval T-009; generasi/riwayat T-011. Riwayat tidak boleh melewati izin terkini atau batas klasifikasi provider.

## Tidak dipakai di MVP

Microservices, Kubernetes, Kafka, Elasticsearch, vector DB terpisah, GraphQL, monorepo, Redis, LangChain/LlamaIndex/framework agent, fine-tuning, i18n framework, state manager global, auth buatan sendiri, realtime, OCR, aplikasi mobile, headless CMS, Docusaurus/MkDocs, Postgres RLS, Dockerfile/compose. Rujuk ADR-0001/0007/0012 sebelum membuka ulang.

Angka mockup (1.284 dokumen, 318 pengguna, 96% akurasi, 41 hasil) bukan fakta. Statistik selalu dari data/eval dan sesudah izin.

## Cara kerja

Satu subtask terkecil per sesi/branch/PR, <=8 berkas kode dan <=400 baris. Tidak push main, merge, tag, atau hapus branch tanpa diminta. Identifier/commit Inggris; teks pengguna Indonesia.

Baca STATUS, AGENTS, berkas ini, spec aktif; tambahan hanya bagian kontrak/aturan yang dirujuk spec. Scope tetap sumber cakupan; jika ada konflik atau persetujuan potong yang tidak tertulis, berhenti. Tidak membaca semua docs untuk perubahan kecil. QC meninjau diff pada commit tertentu dan bukti, bukan mengulang audit repo.
