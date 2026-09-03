# Task specs

Satu task per branch. Satu branch per PR.

## Kenapa task ditulis sebagai spec, bukan sebagai judul

Kualitas kode yang dihasilkan agent AI hampir seluruhnya ditentukan oleh kualitas
instruksinya. "Buatkan halaman upload" menghasilkan kode karangan. Spec dengan file yang
disebutkan, acceptance criteria yang bisa diuji, dan batasan yang eksplisit menghasilkan
kode yang bisa langsung di-review.

Menulis spec juga memaksa kita memikirkan masalahnya lebih dahulu — yang jauh lebih murah
daripada menemukannya saat review.

## Format

Setiap task punya bagian: **Tujuan**, **Konteks** (file yang perlu dibaca), **Lingkup**
(dan yang di luar lingkup), **Acceptance criteria** (bisa dicentang), **Batasan**, dan
**Cara menguji**.

## Cara memakai bersama agent

1. Tempel `docs/context-pack.md` + `docs/scope-mvp.md` + file task ke agent.
2. Minta agent membaca file yang disebut di **Konteks** sebelum menulis kode.
3. Minta agent bekerja di branch `feat/T-00X-slug` lalu membuka PR.
4. Review diff dengan model kelas atas, bukan dengan mata saja.

## Papan task — sprint 6 hari

Disesuaikan dengan `docs/scope-mvp.md` dan ADR-0007.

| ID | Task | Hari | Pemilik | Status spec |
| --- | --- | --- | --- | --- |
| T-001 | Scaffold + Neon + deploy Vercel | 1 | A | ada — abaikan bagian Docker & CI berat |
| T-002 | Design token + sprite ikon + `AppShell` | 1 | B | ada |
| T-003 | Skema DB + migrasi + seed dasar | 1 | A | ada — **kurangi jadi 9 tabel**, lihat catatan |
| T-004 | Auth akun lokal (seed user) | 2 | A | ada — **abaikan bagian invite** |
| T-005 | RBAC + `visibleDocumentsFilter` + test kebocoran | 2 | A | ada — tanpa perubahan |
| T-007 | Katalog + filter + sidebar (mockup s3) | 2 | B | ada |
| T-006 | Viewer + TOC + editor Markdown (mockup s4) | 3 | B | ada |
| T-008 | 20–25 dokumen sintetis | 3 | B | ada |
| T-009 | Hybrid search FTS + pgvector + RRF | 4 | A | ditulis hari 3 |
| T-010 | Halaman hasil pencarian (mockup s2) | 4 | B | ditulis hari 3 |
| T-011 | Endpoint RAG + sitasi + abstain | 5 | A | ditulis hari 4 |
| T-012 | UI chat + chip sitasi + state | 5 | B | ditulis hari 4 |
| T-013 | Eval 10 pertanyaan + tuning | 6 | A | ditulis hari 5 |
| T-014 | Polish, aksesibilitas, README, skrip demo | 6 | B | ditulis hari 5 |

T-009 sampai T-014 ditulis satu hari sebelum dikerjakan, ketika bentuk kodenya sudah
diketahui. Menulis semuanya sekarang menghasilkan spec yang harus dibuang.

## Catatan penyesuaian scope

Spec T-001 sampai T-008 ditulis untuk rencana empat minggu. Yang berubah:

- **T-001** — lewati `Dockerfile`, `docker-compose`, dan job CI mingguan. Cukup
  `typecheck → lint → test → build` di GitHub Actions. Tetap tegakkan aturan portabilitas
  (tanpa Vercel Blob, tanpa Edge, tanpa filesystem lokal).
- **T-003** — kurangi menjadi 9 tabel: `user`, `category`, `label`, `document`,
  `document_label`, `document_version`, `chunk`, `audit_log`, `ai_query`.
  **Dilewati:** `invite`, `job`, `review`, `document_grant`, `document_file`,
  `document_view`. Semua kolom pada tabel yang dibuat tetap lengkap — termasuk
  `embedding_model`, `content_hash`, `review_period_days`.
- **T-004** — hanya credentials + `scripts/seed-admin.ts`. Alur invite dilewati; buat 5 user
  lewat seed yang mencakup kombinasi role dan clearance.
- **T-005** — tanpa perubahan. **Ini task yang paling tidak boleh dipercepat.**
- **T-006** — tambahkan TOC hierarkis bernomor sesuai mockup s4. Lewati UI riwayat versi
  (datanya tetap disimpan).
- **T-008** — 20–25 dokumen cukup, tidak perlu 30. Semua `.md`. Tetap sertakan dokumen yang
  saling mirip dan dokumen kedaluwarsa.
