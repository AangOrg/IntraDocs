# Kontrak API — baseline MVP

Acuan bentuk request/response untuk kerja paralel. Scope menang atas berkas ini. ADR-0012 menetapkan route tunggal dan aturan riwayat. Fitur Fase 2 di akhir adalah daftar penundaan, bukan kontrak implementasi beku.

## Aturan umum

- Node.js runtime, zod di boundary (`lib/schemas/`), Auth.js dan guard pengguna aktif terkini. Semua endpoint produk membutuhkan login.
- Error `{ error: { code, message, details? } }`; jangan memasukkan secret, judul tersembunyi atau stack.
- 400 input invalid; 401 belum login/nonaktif; 403 aksi terlarang pada resource yang boleh diketahui; 404 resource tidak ada/tak terlihat/bukan milik pengguna; 409 konflik revisi/versi/scope.
- Dokumen/chunk/judul/snippet/total/sitasi memakai visibleDocumentsFilter(user) sebelum ranking/agregasi/LIMIT. can() memeriksa perubahan metadata dan aksi.
- Input role/userId/clearance dari klien tidak menjadi identitas. Scope hanya mempersempit izin server. Respons teks/Markdown disanitasi saat dirender.

## Tipe bersama

| Nama | Field |
| --- | --- |
| Classification | public / internal / restricted / secret |
| Role | viewer / contributor / reviewer / admin |
| DocStatus | draft / in_review / published / archived; MVP hanya menghasilkan draft/published |
| Scope | `{ type: all }` atau `{ type: category, categoryId }` atau `{ type: document, documentId }` |
| DocumentSummary | id, title, slug, category {id,name,slug,path[]}, labels [{id,name,color}], classification, ownerUnit, status, version (integer/null untuk draft baru), updatedAt, verified, reviewOverdue |
| Citation | chunkId, documentId, documentTitle, version (integer), headingPath[], anchor, href, classification, updatedAt, verified |

Versi berasal dari document_version, bukan heading_path. href dibangun server dari kategori/slug dan `?version=N#anchor`, bukan URL karangan model. Anchor memakai algoritme chunker/viewer yang sama, termasuk heading duplikat. Kedaluwarsa adalah turunan tanggal/periode, bukan status.

## Autentikasi — MVP

| Method/path | Input | Hasil |
| --- | --- | --- |
| POST /api/auth/login | email, password; mekanisme CSRF Auth.js | cookie Auth.js; user ringkas |
| POST /api/auth/logout | mekanisme CSRF Auth.js | 204; sesi keluar |

Facade memanggil Auth.js, tidak menciptakan format session sendiri. Password salah, email tak ada, dan is_active=false memakai 401 generik tanpa sesi. Role/scope/aktif dimuat kembali untuk permintaan terlindungi; session stale tidak mempertahankan izin.

## Dokumen dan katalog — MVP

| Method/path | Input | Hasil |
| --- | --- | --- |
| GET /api/documents | categoryId?, cursor?, limit? | items: DocumentSummary[], nextCursor?, total |
| GET /api/documents/:slug | version? | DocumentSummary + markdown, owner, revision; tanpa sourceFileUrl |
| POST /api/documents | title, categoryId, classification, ownerUnit, markdown, labelIds?, reviewPeriodDays | id, slug, revision; membuat draft |
| PATCH /api/documents/:id | revision dan field editable parsial yang sama | id, revision; menyimpan draft |
| POST /api/documents/:id/publish | revision, acknowledgeSensitiveWarning? | id, version, revision |
| GET /api/categories | — | enam kategori satu tingkat + documentCount sesudah izin |
| GET /api/labels | — | daftar label; hitungan penggunaan jika ada wajib sesudah izin |

Limit default 10, maksimum 50; cursor opaque, urutan stabil. Total katalog dihitung setelah izin/filter sebelum pagination. Slug unik global. Bila version diminta berbeda dari versi current: periksa izin dahulu, lalu 409 SOURCE_VERSION_CHANGED; jangan menampilkan versi baru seolah sumber lama. UI riwayat/rollback tetap tidak dibangun.

Form /unggah menerima md/txt melalui pembacaan teks di browser, lalu JSON markdown yang sama; tanpa file upload/object storage. Batas teks MVP 256 KiB UTF-8, divalidasi server; ini pembatas input, bukan dukungan unggah 50 MB mockup. Format lain ditolak.

Status/verifikasi/pemilik user tidak bebas diubah melalui field parsial. Publish mengikuti transaksi arsitektur, memindai konten sebelum embedding. Warning mengembalikan 409 SENSITIVE_CONTENT_WARNING tanpa isi sensitif; konfirmasi false-positive harus membawa revision yang sama. Versi berubah memerlukan scan ulang. Pipeline gagal tidak memindah current version.

Pembacaan restricted/secret, termasuk sumber AI, menulis audit_log referensi. Dokumen tersembunyi selalu 404 generik. UI boleh menyediakan editor untuk dokumen milik sendiri; draft tidak masuk search/AI.

## Search — MVP

GET /api/search menerima q (nonkosong, maksimal 2000 karakter), categoryId?, cursor?, limit?. Respons: items berisi DocumentSummary + snippet teks + score; took dalam ms; total; nextCursor?; mode hybrid/keyword. Total adalah jumlah dokumen dalam kumpulan kandidat setelah izin dan deduplikasi, sebelum pagination, bukan klaim ukuran seluruh korpus.

Full-text simple/ts_rank_cd + query embedding, digabung RRF. Boleh embed(), dilarang chat(). Embedding gagal/timeout menghasilkan keyword fallback yang tetap berizin; mode dicatat dan ditampilkan. Tidak ada generasi jawaban di /cari. Tepat satu log ai_query kind=search per request, termasuk error/nol hasil.

## AI dan riwayat — MVP

| Method/path | Input | Hasil |
| --- | --- | --- |
| POST /api/ai/ask | question, requestId, conversationId?, scope? (default all) | SSE |
| GET /api/ai/conversations | cursor?, limit? | items [{id,title,scope,updatedAt}], nextCursor? |
| GET /api/ai/conversations/:id | — | id,title,scope,messages [{role,content,citations[],createdAt}] |
| POST /api/ai/feedback | aiQueryId, vote up/down, note? | 204 |

question maksimal 4000 karakter, note maksimal 2000. requestId UUID dari klien adalah kunci idempotensi per user, bukan otorisasi; simpan pada ai_query, unique(user_id,request_id). Retry selesai mengembalikan pesan tersimpan, bukan mengirim ulang generasi; request masih berjalan mengembalikan 409 REQUEST_IN_PROGRESS.

conversationId, aiQueryId dan riwayat hanya milik user aktif. Periksa ownership dan izin semua sumber sebelum mengembalikan judul/teks, kondensasi, atau generasi. Sumber tak lagi current/diizinkan/di bawah batas AI menyebabkan percakapan 404; daftar menghilangkan percakapan itu. Filter sumber yang terotorisasi dilakukan lewat SQL, bukan mengirim teks dulu lalu menyaring.

Scope disimpan di conversation. Mengganti scope memulai percakapan baru; scope tidak cocok dengan conversationId menghasilkan 409 SCOPE_CHANGED. Dokumen scope tak terlihat 404. Tidak ada fallback dari scope sempit ke seluruh KB.

| Event SSE | Data |
| --- | --- |
| meta | conversationId, requestId |
| sources | citations: Citation[] |
| token | text |
| done | conversationId, aiQueryId, abstained |
| error | code, message |

Urutan sukses meta → sources → token (0 atau lebih) → done. sources selalu sebelum teks, termasuk daftar kosong untuk abstain. Error setelah stream dimulai mengirim error tanpa done sukses. Error sebelum stream memakai status HTTP. Simpan pesan/log sebelum done; jawaban parsial tidak ditandai selesai.

Riwayat maksimal dua putaran. Kondensasi dan generasi memakai identitas asli/guard provider; retrieval reusable tidak mencatat search terpisah. Tanpa dasar cukup, lewati generasi dan kirim abstain. Saat sumber berubah, UI mengosongkan riwayat yang ditolak dan menawarkan percakapan baru.

## Operasional MVP

GET /api/health hanya status ringkas kesehatan aplikasi/DB/vector untuk pemantauan, tanpa secret/detail koneksi. pnpm reindex memakai pipeline T-006; tidak ada endpoint admin reindex MVP. Target angka ada di scope, bukan disalin di sini.

## Fase 2 — jangan dibuat oleh task MVP

Invite/accept, submit-review/review, daftar versi/rollback, presign/complete/job upload, admin users/kategori/audit/dashboard/reindex, jobs/tick, rate limiter aplikasi, SSO, favorit dan feedback dokumen. Sebagian memerlukan tabel tambahan, sebagian memakai tabel yang sudah ada; tidak benar bahwa seluruh endpoint Fase 2 tidak punya tabel.

PDF, DOC/DOCX, XLSX, PPTX, HTML, OCR, ZIP dan batas 50 MB adalah referensi mockup untuk rencana ingest berikutnya, bukan kontrak MVP. Kontrak Fase 2 baru dibekukan bersama spec/ADR terkait.
