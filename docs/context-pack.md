# Context pack

Ringkasan kanonik proyek. **Tempel file ini** ke model AI mana pun di awal sesi, alih-alih
menjelaskan ulang proyek. Jaga tetap di bawah 200 baris — kalau tumbuh, pindahkan detail
ke dokumen lain dan tinggalkan tautan.

## Apa & kenapa

**IntraDocs** = knowledge hub internal + AI assistant yang jawabannya bisa dipertanggungjawabkan.

Masalah: dokumentasi teknis dan SOP tersebar di banyak tempat (share drive, chat, email).
Orang bertanya berulang ke helpdesk untuk hal yang jawabannya sudah tertulis. Search biasa
gagal karena istilah pencarian tidak sama dengan istilah di dokumen.

Solusi: satu tempat untuk menyimpan dokumen sebagai Markdown, dengan kontrol akses per
dokumen, ditambah AI chat yang menjawab hanya dari dokumen yang boleh dilihat penanya —
lengkap dengan sitasi, tanggal update, dan status verifikasi.

Referensi UX: gitdoc.ai (information architecture + citation UX). Bukan workflow Git-nya.

## Batasan proyek

- **Tim 2 orang, target 4 minggu.** Pemotongan scope adalah keputusan default.
- Dibangun dengan bantuan AI ("vibecoding") — karena itu spec, ADR, dan CI adalah
  gerbang kualitas, bukan formalitas.
- Data internal perusahaan → keamanan dan kepatuhan bukan fitur tambahan.

## Keputusan yang sudah dikunci

| Topik | Keputusan | ADR |
| --- | --- | --- |
| Stack | Next.js 15 + TS strict, Postgres 16 + pgvector, Drizzle, Auth.js, Tailwind + shadcn/ui | 0001 |
| Source of truth konten | Postgres (Markdown + YAML frontmatter), + export nightly ke Git sebagai backup satu arah | 0002 |
| AI provider | Belum diputuskan. Abstraksi provider + guard `AI_MAX_CLASSIFICATION` | 0003 |
| RBAC | 4 role x 4 klasifikasi; filter izin di dalam SQL | 0004 |
| Deploy | Vercel untuk dev/demo, Docker-ready sejak hari pertama, tanpa lock-in | 0005 |
| Embedding | 1024 dimensi, final | 0006 |

Yang belum diputuskan dan siapa penanggung jawabnya: `docs/decisions-open.md`.

## Model data (inti)

```
user            id, email, name, role, clearance, unit, password_hash
invite          id, email, role, clearance, unit, token_hash, expires_at, used_at
category        id, name, slug, parent_id
label           id, name, color
document        id, title, slug, category_id, classification, owner_unit,
                status, review_period_days, current_version_id, created_by
document_version id, document_id, version, markdown, frontmatter, source_file_key,
                content_hash, published_at, published_by
document_label  document_id, label_id
chunk           id, document_version_id, heading_path, content, token_count,
                embedding vector(1024), embedding_model,
                classification, owner_unit          -- didenormalisasi untuk filter
review          id, document_version_id, reviewer_id, decision, note, decided_at
job             id, type, payload, status, attempts, run_after, last_error
audit_log       id, actor_id, action, target_type, target_id, metadata, created_at
document_view   id, document_id, user_id, created_at
ai_query        id, user_id, question, retrieved_chunk_ids, answer, abstained, created_at
```

`classification`: `public` < `internal` < `restricted` < `secret`
`role`: `viewer` | `contributor` | `reviewer` | `admin`
`document.status`: `draft` | `in_review` | `published` | `archived`

## Alur utama

**Ingest.** Upload → berkas asli ke object storage (+ SHA-256) → job konversi ke Markdown →
normalisasi frontmatter → draft → **manusia memeriksa hasil konversi** → review/approval →
published (baris versi baru, immutable) → job chunk + embed + index.

**Search.** Query → Postgres full-text (`tsvector`) + pgvector cosine, keduanya sudah
terfilter izin → gabung dengan Reciprocal Rank Fusion → hasil.

**AI chat.** Pertanyaan → retrieval yang sama (terfilter izin, tersaring
`AI_MAX_CLASSIFICATION`) → kalau skor di bawah threshold: abstain → kalau tidak: LLM
menjawab dengan sitasi wajib → catat ke `ai_query`.

## Aturan yang paling sering dilanggar (jangan)

1. Filter izin **di dalam** query, bukan setelahnya, dan bukan di prompt.
2. Hanya versi `published` yang masuk index retrieval. Draft tidak pernah bisa muncul di AI.
3. Search harus tetap berfungsi saat provider AI mati. AI adalah lapisan di atas search,
   bukan gerbangnya.
4. Render markdown selalu lewat `rehype-sanitize`.
5. Selama provider AI belum disetujui: **hanya dokumen sintetis/publik** yang boleh masuk
   environment yang tersambung API publik.

## Glosarium (ID ↔ EN)

| Indonesia (UI) | English (kode) | Catatan |
| --- | --- | --- |
| Publik / Internal / Terbatas / Rahasia | `public` / `internal` / `restricted` / `secret` | Klasifikasi keamanan dokumen |
| Pembaca | `viewer` | Hanya membaca |
| Kontributor | `contributor` | Boleh membuat draft |
| Peninjau | `reviewer` | Boleh menyetujui & publikasi di kategorinya |
| Admin Knowledge | `admin` | Kelola user, taksonomi, semua dokumen |
| Klasifikasi | `classification` | |
| Kewenangan akses | `clearance` | Klasifikasi tertinggi yang boleh dilihat user |
| Unit / Divisi | `unit` | Dipakai untuk cakupan akses |
| Periode tinjau ulang | `review_period_days` | Dokumen kedaluwarsa → tampilkan peringatan |
| Terverifikasi | `verified` | Sudah ditinjau pemilik dokumen |
| Draft / Menunggu review / Terpublikasi | `draft` / `in_review` / `published` | |
| Potongan dokumen | `chunk` | Unit retrieval |
| Sitasi | `citation` | Menunjuk ke `chunk.heading_path` |

## Environment variables

```
DATABASE_URL=
AUTH_SECRET=
S3_ENDPOINT= S3_BUCKET= S3_ACCESS_KEY_ID= S3_SECRET_ACCESS_KEY=
AI_PROVIDER=            # openai | azure | ollama | none
AI_BASE_URL=
AI_API_KEY=
AI_CHAT_MODEL=
AI_EMBEDDING_MODEL=
AI_EMBEDDING_DIM=1024   # jangan diubah, lihat ADR-0006
AI_MAX_CLASSIFICATION=public   # public | internal | restricted | secret
JOB_TICK_SECRET=
ENABLE_OIDC=false
OIDC_ISSUER= OIDC_CLIENT_ID= OIDC_CLIENT_SECRET=
```

## Status saat ini

Pra-implementasi. Yang ada di repo: mockup HTML dan artifak perancangan.
Pekerjaan berikutnya: `docs/tasks/` (mulai dari T-001).
