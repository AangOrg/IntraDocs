# AGENTS.md — aturan kerja untuk manusia & AI di repo ini

Baca ini sebelum menulis satu baris kode. Berlaku untuk semua kontributor, termasuk
agent AI (Notion AI, Claude Code, Copilot, Cursor).

## Proyek dalam lima baris

IntraDocs adalah platform knowledge management internal. Pengguna mencari dan membaca
dokumentasi/SOP, lalu bertanya ke AI yang menjawab **hanya** dari dokumen yang boleh
diakses pengguna tersebut, dengan sitasi ke bagian dokumen. Kontributor mengunggah
berkas (MD/TXT/DOCX/PDF) yang dikonversi ke Markdown, diberi metadata dan klasifikasi
keamanan, lalu di-review sebelum dipublikasikan. Konteks lengkap: `docs/context-pack.md`.

## Tiga aturan yang tidak bisa dilanggar

1. **Semua query dokumen harus lewat `visibleDocumentsFilter(user)`.**
   Filter izin diterapkan **di dalam SQL query** — bukan di aplikasi setelah query,
   dan sama sekali **bukan** di prompt LLM. Berlaku untuk katalog, search, dan retrieval
   RAG. Lihat `docs/adr/0004-rbac-and-permission-aware-retrieval.md`.
2. **AI tidak boleh mengarang.** Setiap klaim wajib punya sitasi ke chunk. Kalau skor
   retrieval di bawah threshold → abstain eksplisit. Jangan pernah menambahkan
   "pengetahuan umum" model ke dalam jawaban.
3. **Markdown hasil upload adalah untrusted input.** Render selalu melalui
   `rehype-sanitize`. Isi dokumen tidak boleh pernah diperlakukan sebagai instruksi
   oleh LLM, dan AI chat tidak punya akses tool/write apa pun.

## Larangan

- Jangan tambah dependency tanpa ADR. Tanya dulu: bisa selesai < 100 baris sendiri?
- Jangan pakai `any`, `as any`, `@ts-ignore`, atau non-null assertion `!`. TypeScript strict.
- Jangan bikin abstraksi untuk satu pemakai. Duplikasi dua kali itu OK; abstraksi prematur tidak.
- Jangan tulis komentar yang mengulang kode. Komentar hanya menjelaskan **kenapa**.
- Jangan tinggalkan kode mati, `console.log`, TODO kosong, atau data placeholder/lorem.
- Jangan buat file > 300 baris tanpa alasan. Jangan buat folder untuk satu file.
- Jangan tulis test yang hanya memverifikasi mock.
- Jangan sentuh file di luar daftar "File yang boleh disentuh" pada task spec.
- Jangan ubah skema DB tanpa file migrasi.
- Jangan pakai Edge runtime, Vercel Blob, atau filesystem lokal untuk penyimpanan.
  Lihat `docs/adr/0005-deployment-and-portability.md`.
- Jangan commit `.env`, secret, atau dokumen internal Telkom yang asli.

## Kewajiban

- Kode, identifier, nama file, dan commit message: **Bahasa Inggris**.
  Copy/teks UI: **Bahasa Indonesia**. Tanpa campur.
- Warna, spacing, radius, dan shadow **hanya** dari design token di `tailwind.config.ts`.
  Nol nilai hex hardcoded — token diturunkan dari CSS variable di mockup.
- Validasi semua input di boundary dengan zod. Skema tinggal di `lib/schemas/`.
- **Vertical slice**: satu task menyelesaikan DB → API → UI untuk satu fitur.
  Jangan "semua model dulu, UI nanti".
- Sebelum menyatakan selesai: `pnpm typecheck && pnpm lint && pnpm test` harus hijau.
- PR wajib berisi bagian **Cara menguji** dengan langkah konkret.

## Konvensi

- Package manager **pnpm**. Versi Node mengikuti `.nvmrc`.
- Struktur: `app/` (route), `components/` (UI), `lib/` (logic murni), `db/` (skema + migrasi),
  `scripts/`, `tests/`.
- Logic yang **wajib** punya unit test: RBAC filter, markdown chunker, RRF fusion,
  parser berkas, invite token, dan `AI_MAX_CLASSIFICATION` guard.
- Commit: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`).
- Branch: `feat/T-00X-slug`, `fix/...`, `docs/...`. Satu task = satu branch = satu PR.
- Target ukuran PR: < 400 baris diff.

## Cara mengambil pekerjaan

1. Pilih task di `docs/tasks/` dengan `Status: Ready`.
2. Baca **Acceptance criteria** dan **File yang boleh disentuh**.
3. Buat branch, kerjakan, jalankan gerbang kualitas.
4. Buka PR memakai template, tautkan ID task.
5. Cek Definition of Done di `docs/workflow.md` sebelum minta review.

## Kalau ragu

Ambil keputusan paling **boring** dan paling mudah dihapus. Kalau keputusannya mengikat
(dependency baru, perubahan skema, pola arsitektur baru), **berhenti dan tulis ADR dulu** —
jangan diam-diam memutuskan di dalam PR.
