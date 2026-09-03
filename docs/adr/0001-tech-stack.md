# ADR-0001: Tech stack

- Status: Diterima
- Tanggal: 2026-09-03

## Konteks

Dua orang, satu bulan, wajib menghasilkan produk yang benar-benar dipakai. Batasan yang
mengikat bukan kemampuan teknologi, melainkan **jumlah hal yang bisa dipelajari, di-debug,
dan dioperasikan oleh dua orang dalam empat minggu** — dan diserahkan kepada orang lain
setelahnya.

Karena pengembangan dilakukan dengan bantuan agent AI, ada batasan tambahan yang sering
dilupakan: stack harus punya **banyak contoh publik dan pola yang stabil**. Framework yang
niche menghasilkan kode karangan yang tidak bisa dikompilasi.

## Keputusan

| Lapisan | Pilihan | Alasan singkat |
| --- | --- | --- |
| Framework | Next.js 15 App Router, TypeScript strict | Satu bahasa, satu deployable, RSC menekan JS klien |
| UI | Tailwind + shadcn/ui | Token dari mockup dipetakan langsung; komponen ada di repo, bukan dependensi |
| Database | PostgreSQL 16 + `pgvector` + `tsvector` | Satu database untuk relational + FTS + vector |
| ORM | Drizzle | SQL-first, migrasi eksplisit, tipe aman |
| Auth | Auth.js (credentials + invite; OIDC di balik flag) | Tidak menulis auth sendiri; SSO menyusul tanpa refactor |
| Storage | S3-compatible via `@aws-sdk/client-s3` | Sama API-nya untuk R2 dan MinIO on-prem |
| Markdown | `remark`/`rehype` + **`rehype-sanitize`** | Ekosistem matang; sanitasi wajib |
| Parser | `mammoth` (DOCX), `unpdf` (PDF text-layer), `turndown` (HTML) | Ringan, tanpa layanan eksternal |
| Job | Tabel `job` + endpoint tick | Tanpa Redis, tanpa broker |
| Observability | `pino` + error tracking + `/api/health` | Cukup untuk skala ini |
| Test | Vitest + Playwright | Unit untuk logic, E2E untuk alur izin |
| CI | GitHub Actions | typecheck, lint, test, build, cek migrasi |
| Hash password | `@node-rs/argon2` | Argon2id tanpa native build yang rapuh |

Runtime selalu `nodejs`. Tidak ada Edge runtime.

## Yang sengaja TIDAK dipakai

Microservices · Kubernetes · Kafka · Elasticsearch · vector database terpisah · GraphQL ·
monorepo/Turborepo · Redis (fase 1) · LangChain/LlamaIndex/framework agent · fine-tuning ·
framework i18n · state manager global · auth buatan sendiri · realtime/websocket · OCR ·
aplikasi mobile · headless CMS · Vercel Blob · Edge runtime · filesystem lokal untuk berkas.

Daftar ini bagian dari keputusan, bukan catatan tambahan. Setiap item di atas pernah
terlihat masuk akal dan akan diusulkan lagi oleh agent AI; menuliskannya sebagai penolakan
eksplisit mencegah scope creep yang tidak sengaja.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| Docusaurus / MkDocs / Docsify | Tidak punya RBAC per dokumen. RBAC adalah persyaratan inti, bukan tambahan |
| Django/Laravel + frontend terpisah | Dua bahasa, dua deployable, dua pipeline. Terlalu mahal untuk dua orang |
| Elasticsearch untuk search | Satu komponen ops tambahan; Postgres FTS cukup pada 10.000 dokumen |
| Pinecone/Qdrant/Weaviate | Data konten keluar dari Postgres, sinkronisasi izin menjadi masalah baru |
| Supabase penuh (Auth + Storage SDK) | Mengikat ke satu vendor; on-prem menjadi sulit |
| LangChain | Abstraksi berat untuk pipeline yang di bawahnya hanya ~200 baris SQL + fetch |

## Konsekuensi

**Lebih mudah:** satu perintah untuk menjalankan seluruh stack; agent AI menghasilkan kode
yang benar karena polanya umum; deploy ke mana pun (Vercel maupun Docker); handover realistis.

**Lebih sulit:** Postgres menjadi titik pusat, jadi index dan backup harus benar; tanpa
reranker, kualitas retrieval bergantung pada chunking dan hybrid search yang baik.

**Risiko yang diterima:** pgvector akan melambat jauh di atas ~200.000 dokumen. Itu jauh di
luar target 10.000 dokumen, dan bila terjadi berarti proyek ini sukses — masalah yang layak
dimiliki.

## Cara membatalkan

Setiap komponen bisa ditukar secara terisolasi karena logic tinggal di `lib/`, bukan di
route: search dapat dipindahkan ke Elasticsearch dengan menulis ulang satu modul retrieval;
storage dapat ditukar karena hanya API S3 yang dipakai; provider AI ditukar lewat env
(ADR-0003). Yang benar-benar mahal untuk dibatalkan hanyalah pilihan Postgres, dan itulah
satu-satunya keputusan berat di ADR ini.
