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

## Autentikasi

| Method | Path | Body / Query | Respons |
| --- | --- | --- | --- |
| `POST` | `/api/auth/login` | `{ email, password }` | set cookie session |
| `POST` | `/api/auth/logout` | — | `204` |
| `POST` | `/api/invites` (admin) | `{ email, role, clearance, unit }` | `{ inviteUrl, expiresAt }` |
| `POST` | `/api/invites/accept` | `{ token, name, password }` | set cookie session |

## Katalog & dokumen

| Method | Path | Query / Body | Respons |
| --- | --- | --- | --- |
| `GET` | `/api/documents` | `categoryId?`, `labelIds?`, `classification?`, `status?`, `updatedWithinDays?`, `cursor?`, `limit?` | `{ items: DocumentSummary[], nextCursor?, total }` |
| `GET` | `/api/documents/:slug` | — | `{ ...DocumentSummary, markdown, frontmatter, owner, sourceFileUrl? }` |
| `POST` | `/api/documents` | `{ title, categoryId, classification, ownerUnit, markdown?, labelIds? }` | `{ id, slug }` |
| `PATCH` | `/api/documents/:id` | field parsial | `{ id }` |
| `POST` | `/api/documents/:id/submit-review` | `{ reviewerId, note? }` | `{ reviewId }` |
| `POST` | `/api/documents/:id/review` (reviewer/admin) | `{ decision, note? }` | `{ status }` |
| `GET` | `/api/documents/:id/versions` | — | `{ versions: Array<{ version, publishedAt, publishedBy }> }` |
| `GET` | `/api/categories` | — | pohon kategori |
| `GET` | `/api/labels` | — | daftar label |

## Upload & ingest

| Method | Path | Body | Respons |
| --- | --- | --- | --- |
| `POST` | `/api/uploads/presign` | `{ fileName, contentType, size }` | `{ uploadUrl, fileKey }` |
| `POST` | `/api/uploads/complete` | `{ fileKey, fileName }` | `{ jobId, documentId }` |
| `GET` | `/api/uploads/:jobId` | — | `{ status, error?, documentId? }` |

Batas: 50 MB per berkas. Format didukung di MVP: `.md`, `.txt`, `.docx`, `.pdf`
(hanya PDF yang punya text layer). Format lain ditolak dengan pesan yang jelas.

## Search

| Method | Path | Query | Respons |
| --- | --- | --- | --- |
| `GET` | `/api/search` | `q`, filter sama seperti `/api/documents`, `limit?` | `{ items: Array<DocumentSummary & { snippet: string; score: number }>, took: number, total }` |

Snippet menyorot istilah yang cocok. Search **tidak boleh** memanggil provider AI — harus
tetap berfungsi ketika AI mati.

## AI chat

| Method | Path | Body | Respons |
| --- | --- | --- | --- |
| `POST` | `/api/ai/ask` | `{ question, conversationId? }` | SSE stream |
| `POST` | `/api/ai/feedback` | `{ aiQueryId, vote, note? }` | `204` |

Event SSE:

```
event: sources     data: { citations: Citation[] }        // dikirim SEBELUM token pertama
event: token       data: { text: string }
event: done        data: { aiQueryId, abstained: boolean }
event: error       data: { code, message }
```

Sumber dikirim lebih dahulu supaya UI bisa menampilkan dasar jawaban bahkan ketika sistem
memilih abstain atau ketika generasi gagal di tengah jalan.

Rate limit: per pengguna, dikembalikan sebagai `429` dengan `Retry-After`.

## Admin & operasional

| Method | Path | Keterangan |
| --- | --- | --- |
| `GET` | `/api/admin/users` | daftar pengguna dan role |
| `PATCH` | `/api/admin/users/:id` | ubah role, clearance, unit |
| `POST` | `/api/admin/reindex` | jadwalkan reindex penuh atau per dokumen |
| `GET` | `/api/admin/audit` | audit log dengan filter |
| `POST` | `/api/jobs/tick` | dilindungi `JOB_TICK_SECRET`; dipanggil cron atau worker |
| `GET` | `/api/health` | memeriksa DB dan object storage |
