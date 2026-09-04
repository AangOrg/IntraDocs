# Kontrak API

Tujuan dokumen ini: memungkinkan **Orang A dan Orang B bekerja paralel** tanpa saling
menunggu. Sepakati file ini di hari 1–2; setelah itu perubahan yang memutus kontrak harus
dikomunikasikan di PR terpisah.

Aturan umum:

- Semua handler memakai `runtime = 'nodejs'`. Tanpa Edge.
- Semua input divalidasi zod di boundary; skema tinggal di `lib/schemas/`.
- Semua respons memakai bentuk error yang sama.
- Semua endpoint yang mengembalikan dokumen atau chunk **wajib** melewati
  `visibleDocumentsFilter(user)`.
- Kode status: `401` belum login, `403` tidak berwenang, `404` tidak ada **atau tidak
  terlihat oleh pengguna** (jangan bedakan keduanya — membedakannya membocorkan keberadaan
  dokumen).
- Setiap **jumlah** yang dikembalikan (`total`, jumlah hasil pencarian, jumlah dokumen per
  kategori) dihitung **setelah** filter izin. Menjawab `total: 41` lalu hanya mengirim 12
  item juga kebocoran, karena selisihnya bisa dibaca (ADR-0011).

## Penanda fase

Setiap endpoint di bawah punya penanda. Penanda ini mengikat sekuat tabelnya sendiri.

- **MVP** — dibangun minggu ini.
- **MVP (potongan #N)** — masuk MVP, tetapi ada di urutan potong `docs/scope-mvp.md` pada
  nomor N. Kalau waktu habis, yang bernomor kecil dilepas lebih dulu.
- **Fase 2** — kontraknya ditulis di sini supaya bentuknya tidak berubah saat dibangun
  nanti, tetapi **tidak dibangun minggu ini**. Tabel penopangnya sengaja **tidak ada** di
  skema sebelas tabel T-003 (`invite`, `document_file`, `document_grant`, `job`, `review`,
  `document_view`). Membuat route handler-nya berarti menambah tabel tanpa ADR baru.

Endpoint Fase 2 yang muncul di kode MVP adalah cacat, bukan bonus: ia tidak punya tabel,
tidak punya test izin, dan tidak masuk daftar "Definisi selesai" di `docs/scope-mvp.md`.

## Satu nama untuk endpoint AI

`docs/mockup-alignment.md` menyebut layanan RAG sebagai `app/api/chat`, berkas ini menyebut
`POST /api/ai/ask`. Dua nama untuk satu rute berarti dua sesi eksekusi membuat dua handler.

**Nama resmi: `POST /api/ai/ask`.** Alasannya: seluruh endpoint AI berbagi awalan `/api/ai/*`
(`ask`, `feedback`, `conversations`), dan `chat` sudah dipakai sebagai nama fungsi pada
`lib/ai/provider.ts` — `chat()` adalah pemanggilan provider, bukan rute HTTP. `app/api/chat`
tidak dipakai. `docs/mockup-alignment.md` sudah diperbaiki di PR yang sama.

## Bentuk error

```ts
type ApiError = {
  error: { code: string; message: string; details?: unknown }
}
```

## Tipe bersama

```ts
type Classification = 'public' | 'internal' | 'restricted' | 'secret'
type Role = 'viewer' | 'contributor' | 'reviewer' | 'admin'
type DocStatus = 'draft' | 'in_review' | 'published' | 'archived'

type DocumentSummary = {
  id: string
  title: string
  slug: string
  category: { id: string; name: string; path: string[] }
  labels: Array<{ id: string; name: string; color: string }>
  classification: Classification
  ownerUnit: string
  status: DocStatus
  version: number
  updatedAt: string
  verified: boolean
  reviewOverdue: boolean
}

type Citation = {
  chunkId: string
  documentId: string
  documentTitle: string
  headingPath: string[]
  classification: Classification
  updatedAt: string
  verified: boolean
}
```

`category.path` dan `Citation.headingPath` tetap array penuh walaupun UI hanya menampilkan
satu tingkat (potongan #5 `docs/scope-mvp.md`). Yang dipotong tampilannya, bukan datanya.

Enum `DocStatus` punya empat nilai, tetapi alur MVP hanya `draft` → `published`. `in_review`
dan `archived` ada supaya tidak perlu migrasi saat approval dibangun di Fase 2.

## Autentikasi

| Method | Path | Body / Query | Respons | Status |
| --- | --- | --- | --- | --- |
| `POST` | `/api/auth/login` | `{ email, password }` | set cookie session | **MVP** |
| `POST` | `/api/auth/logout` | — | `204` | **MVP** |
| `POST` | `/api/invites` (admin) | `{ email, role, categoryScope, unit }` | `{ inviteUrl, expiresAt }` | Fase 2 — tabel `invite` tidak dibuat |
| `POST` | `/api/invites/accept` | `{ token, name, password }` | set cookie session | Fase 2 |

`login` **wajib** menolak pengguna dengan `is_active = false` dengan `401` dan pesan yang
sama seperti kata sandi salah. Ini yang dibuktikan `tests/rbac/inactive-user.spec.ts`, dan
pengguna keenam pada tabel seed `docs/rbac-matrix.md` ada khusus untuk test ini.

Body invite tidak lagi punya `clearance`: klasifikasi tertinggi diturunkan dari `role`
(`docs/rbac-matrix.md`), dan kolom clearance per pengguna dilarang ADR-0004.

## Katalog & dokumen

| Method | Path | Query / Body | Respons | Status |
| --- | --- | --- | --- | --- |
| `GET` | `/api/documents` | `categoryId?`, `labelIds?`, `classification?`, `status?`, `updatedWithinDays?`, `cursor?`, `limit?` | `{ items: DocumentSummary[], nextCursor?, total }` | **MVP** — filter selain `categoryId` adalah potongan #4 |
| `GET` | `/api/documents/:slug` | — | `{ ...DocumentSummary, markdown, frontmatter, owner }` | **MVP** |
| `POST` | `/api/documents` | `{ title, categoryId, classification, ownerUnit, markdown?, labelIds? }` | `{ id, slug }` | **MVP (potongan #2)** |
| `PATCH` | `/api/documents/:id` | field parsial | `{ id }` | **MVP (potongan #2)** |
| `POST` | `/api/documents/:id/submit-review` | `{ reviewerId, note? }` | `{ reviewId }` | Fase 2 — tabel `review` tidak dibuat |
| `POST` | `/api/documents/:id/review` (reviewer/admin) | `{ decision, note? }` | `{ status }` | Fase 2 |
| `GET` | `/api/documents/:id/versions` | — | `{ versions: Array<{ version, publishedAt, publishedBy }> }` | Fase 2 — tabel `document_version` sudah ada, UI riwayat versinya yang ditunda |
| `GET` | `/api/categories` | — | pohon kategori | **MVP** — satu tingkat, `parent_id` tetap ada di skema |
| `GET` | `/api/labels` | — | daftar label | **MVP** |

`sourceFileUrl` dihapus dari respons `/api/documents/:slug`: tanpa object storage di MVP,
tidak ada berkas asli untuk ditautkan. Ia kembali bersama Fase 2 unggah multi-format.

Membaca dokumen `restricted` atau `secret` **menulis satu baris `audit_log`**, walaupun
halaman audit log sendiri Fase 2. Ini persyaratan kepatuhan dan ada di daftar "tidak boleh
dipotong" `docs/scope-mvp.md`.

Dokumen yang tidak lolos `visibleDocumentsFilter(user)` menghasilkan `404` polos: tanpa
judul, pemilik, atau kategori di badan respons, dan tanpa `PermissionDeniedState` di UI
(ADR-0011).

## Upload & ingest — seluruhnya Fase 2

Di MVP tidak ada jalur unggah berkas. Dokumen masuk lewat `pnpm seed` sebagai Markdown
(`db/seed/documents/`, lihat T-008). Tabel `document_file` dan `job` sengaja tidak dibuat.

| Method | Path | Body | Respons | Status |
| --- | --- | --- | --- | --- |
| `POST` | `/api/uploads/presign` | `{ fileName, contentType, size }` | `{ uploadUrl, fileKey }` | Fase 2 |
| `POST` | `/api/uploads/complete` | `{ fileKey, fileName }` | `{ jobId, documentId }` | Fase 2 |
| `GET` | `/api/uploads/:jobId` | — | `{ status, error?, documentId? }` | Fase 2 |

Rencana Fase 2: batas 50 MB per berkas; format `.md`, `.txt`, `.docx`, `.pdf` (hanya PDF
yang punya text layer); format lain ditolak dengan pesan yang jelas. Angka dan daftar format
ini **bukan** cakupan MVP — MVP hanya `.md` dan `.txt`.

## Search

| Method | Path | Query | Respons | Status |
| --- | --- | --- | --- | --- |
| `GET` | `/api/search` | `q`, filter sama seperti `/api/documents`, `limit?` | `{ items: Array<DocumentSummary & { snippet: string; score: number }>, took: number, total }` | **MVP** |

Snippet menyorot istilah yang cocok. Search **tidak boleh** memanggil provider AI — harus
tetap berfungsi ketika AI mati. Karena itu halaman hasil pencarian tidak punya kotak jawaban
AI seperti layar 2 mockup; jawaban AI hidup di layar 9. Lihat `docs/mockup-alignment.md`.

`total` dan `items` memakai filter izin yang sama, dihitung dalam satu query (ADR-0011).
Setiap baris pencarian menulis satu baris log kueri berisi kueri, jumlah hasil, dan role
penanya — sumber data dashboard Fase 2, dan ada di daftar "tidak boleh dipotong".

Target: p95 < 500 ms.

## AI chat

| Method | Path | Body | Respons | Status |
| --- | --- | --- | --- | --- |
| `POST` | `/api/ai/ask` | `{ question, conversationId? }` | SSE stream | **MVP** |
| `POST` | `/api/ai/feedback` | `{ aiQueryId, vote, note? }` | `204` | **MVP** |
| `GET` | `/api/ai/conversations` | `cursor?`, `limit?` | `{ items: Array<{ id, title, updatedAt }>, nextCursor? }` | **MVP (potongan #1)** |
| `GET` | `/api/ai/conversations/:id` | — | `{ id, title, messages: Array<{ role, content, citations: Citation[], createdAt }> }` | **MVP (potongan #1)** |

Dua endpoint percakapan menopang riwayat di sisi kiri layar 9 (ADR-0010). Keduanya adalah
potongan #1 pada urutan potong: kalau hari 5 tersendat, penyimpanan riwayat yang dilepas
lebih dulu — chat tetap berfungsi, riwayatnya saja tidak tersimpan.

Body `ask` **tidak** punya parameter ruang lingkup jawaban. Selektor ruang lingkup adalah
potongan #3, dan tombol "Tanya AI tentang halaman ini" di layar 4 cukup mengisi pertanyaan
awal. Menambahkannya nanti adalah satu field opsional, bukan perubahan bentuk.

Event SSE:

```
event: sources     data: { citations: Citation[] }        // dikirim SEBELUM token pertama
event: token       data: { text: string }
event: done        data: { aiQueryId, abstained: boolean }
event: error       data: { code, message }
```

Sumber dikirim lebih dahulu supaya UI bisa menampilkan dasar jawaban bahkan ketika sistem
memilih abstain atau ketika generasi gagal di tengah jalan.

Retrieval memakai `visibleDocumentsFilter(user)` dengan identitas **penanya asli**;
penulisan ulang pertanyaan lanjutan tidak boleh mengubah identitas itu. Guard
`AI_MAX_CLASSIFICATION` diperiksa terpisah, sebelum potongan masuk konteks LLM.

Rate limit per pengguna dengan `429` dan `Retry-After`: **Fase 2** — rate limit ada di
daftar "tidak masuk MVP" `docs/scope-mvp.md`.

Target: p95 token pertama < 3 detik.

## Admin & operasional

| Method | Path | Keterangan | Status |
| --- | --- | --- | --- |
| `GET` | `/api/admin/users` | daftar pengguna dan role | Fase 2 — konsol pengguna ditunda, modelnya tetap dibangun penuh |
| `PATCH` | `/api/admin/users/:id` | ubah `role`, `category_scope`, `unit`, `is_active` | Fase 2 — tanpa `clearance`, lihat `docs/rbac-matrix.md` |
| `POST` | `/api/admin/reindex` | jadwalkan reindex penuh atau per dokumen | Fase 2 sebagai endpoint; di MVP reindex dijalankan lewat `pnpm reindex` |
| `GET` | `/api/admin/audit` | audit log dengan filter | Fase 2 untuk UI; **penulisan** `audit_log` masuk MVP |
| `POST` | `/api/jobs/tick` | dilindungi `JOB_TICK_SECRET`; dipanggil cron atau worker | Fase 2 — tabel `job` tidak dibuat |
| `GET` | `/api/health` | memeriksa koneksi basis data dan ekstensi `vector` | **MVP** — tanpa pemeriksaan object storage |
