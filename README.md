# IntraDocs

Platform knowledge management internal: cari, baca, dan tanya AI tentang dokumentasi, SOP, dan panduan teknis internal.

Referensi UX: [gitdoc.ai](https://gitdoc.ai). Konten disimpan sebagai Markdown, dengan kontrol akses per dokumen (RBAC + klasifikasi keamanan) dan AI chat yang **wajib** mensitasi sumber.

> **Status: perencanaan (pra-implementasi).** Belum ada kode aplikasi. Seluruh isi repo ini masih artifak perancangan.

## Mulai dari mana

| Kalau kamu... | Baca |
| --- | --- |
| Baru bergabung, atau membuka sesi baru dengan model AI | [`docs/context-pack.md`](docs/context-pack.md) — ringkasan proyek, tempel ini ke model mana pun |
| Mau tahu apa yang dibangun dan apa yang **tidak** | [`docs/prd.md`](docs/prd.md) |
| Mau tahu cara sistemnya bekerja | [`docs/architecture.md`](docs/architecture.md) |
| Akan menulis kode (manusia atau AI) | [`AGENTS.md`](AGENTS.md) — **wajib, baca dulu** |
| Mau tahu cara kerja tim, PR, dan review | [`docs/workflow.md`](docs/workflow.md) |
| Mau mengambil pekerjaan | [`docs/tasks/`](docs/tasks/) |
| Mau tahu kenapa keputusan X diambil | [`docs/adr/`](docs/adr/) |
| Mau tahu apa yang belum diputuskan | [`docs/decisions-open.md`](docs/decisions-open.md) |

## Ringkasan teknis

- **Next.js 15** (App Router) + TypeScript strict — satu deployable
- **PostgreSQL 16 + pgvector** — relational, full-text search, dan vector dalam satu datastore
- **Drizzle ORM**, **Auth.js**, **Tailwind + shadcn/ui**
- Konten: **Markdown + YAML frontmatter**, source of truth di Postgres — [ADR-0002](docs/adr/0002-content-source-of-truth.md)
- AI: **hybrid retrieval** (full-text + vector, digabung dengan RRF) → jawaban bersitasi, dengan abstain eksplisit
- Embedding: **1024 dimensi**, provider-agnostic — [ADR-0006](docs/adr/0006-embedding-dimension.md)

## Mockup

`intradocs-mockup_1.html` — mockup HTML statis multi-screen; acuan visual dan information architecture. Buka langsung di browser.
Pemetaan screen ke komponen: [`docs/ui-inventory.md`](docs/ui-inventory.md).

## Tim & timeline

2 orang, target 4 minggu. Pembagian peran dan roadmap: [`docs/workflow.md`](docs/workflow.md) dan [`docs/roadmap.md`](docs/roadmap.md).
