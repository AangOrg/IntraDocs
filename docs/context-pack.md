# Context Pack — fakta stabil IntraDocs

Berkas ini sengaja pendek. Isinya hanya fakta yang tidak berubah selama sprint.
Kalau ada yang bertentangan, urutan kekuatannya: `docs/scope-mvp.md` > ADR > berkas ini > sisanya.

## Proyek

IntraDocs — web dokumentasi internal dengan RBAC dan AI chat bersitasi.
Konteks: tugas magang, 2 orang, dikerjakan dengan bantuan agent AI. Referensi produk: gitdoc.ai.
Mockup: `intradocs-mockup_1.html` — 11 layar, bahasa Indonesia. Sumber kebenaran untuk tata letak dan istilah UI.

## Lima batasan keras

1. Seluruh isi dokumen bersifat sintetis dan fiktif. Tidak pernah ada dokumen Telkom asli di repo ini.
2. Filter izin selalu dijalankan di SQL lewat `visibleDocumentsFilter()`. Tidak pernah di prompt LLM.
3. Jangan pernah push ke `main`. Satu task satu branch `feat/T-0XX-slug`, lalu buka PR.
4. Jangan merge PR atau menghapus branch tanpa permintaan eksplisit dari pemilik repo.
5. Bahasa Indonesia untuk teks UI dan dokumen. Bahasa Inggris untuk identifier kode dan pesan commit.

## Kosakata domain (jangan dicampur)

- **Klasifikasi keamanan bukan status alur kerja.** Dua kolom terpisah.
  - `classification`: `public < internal < restricted < secret` (UI: Publik, Internal, Terbatas, Rahasia)
  - `status`: `draft | in_review | published | archived`
- Satu dokumen punya **tepat satu kategori** dan **banyak label**.
- Role: `viewer | contributor | reviewer | admin`.
- **Izin punya dua dimensi.** `role` menentukan *apa* yang boleh dilakukan; `user.category_scope` menentukan *di kategori mana*. Cakupan hanya berlaku untuk dokumen `restricted` dan `secret` — `public` dan `internal` tetap terlihat lintas kategori. Lihat ADR-0009.

## Identifier yang harus dipakai persis

| Hal | Nama |
|---|---|
| Filter izin | `lib/rbac/visible-documents.ts` → `visibleDocumentsFilter(user): SQL` |
| Cek aksi | `lib/rbac/can.ts` → `can(user, action, resource)` |
| Abstraksi AI | `lib/ai/provider.ts` → `embed()`, `chat()`, tipe `AiProvider` |
| Test kebocoran | `tests/rbac/ai-retrieval-leak.spec.ts` |
| Set evaluasi | `docs/eval/questions.jsonl` |
| Perintah | `pnpm typecheck` · `pnpm lint` · `pnpm test` · `pnpm build` · `pnpm eval` · `pnpm reindex` |

## Angka yang mengikat

- Skema MVP: **11 tabel**. Embedding **1024 dimensi**. Potongan ~500–800 token, tumpang tindih ~15%.
- Korpus sintetis: 20–25 dokumen `.md`, termasuk beberapa nyaris duplikat dan satu kedaluwarsa.
- Set evaluasi: 10 pertanyaan.
- Target: hit@5 ≥ 0,85 · kebocoran izin = 0 · 0 klaim tanpa sitasi · abstain ≥ 90% untuk pertanyaan di luar korpus · p95 pencarian < 500 ms · p95 token pertama < 3 detik.
- Skala rancangan: 10.000 dokumen, 2.000 pengguna.

## Angka yang TIDAK boleh dipakai

Semua angka di mockup (1.284 dokumen, 318 pengguna, 96% akurasi AI, 41 hasil, 18.402 pencarian) adalah pengisi tampilan dan **saling tidak konsisten satu sama lain**. Jangan pernah ditampilkan sebagai fakta di aplikasi. Angka di aplikasi selalu dihitung dari basis data.

## Yang tidak dipakai (jangan diusulkan ulang)

Microservices · Kubernetes · Kafka · Elasticsearch · vector DB terpisah · GraphQL · monorepo · Redis · LangChain/LlamaIndex/framework agent · fine-tuning · framework i18n · state manager global · auth buatan sendiri · realtime · OCR · aplikasi mobile · headless CMS · Docusaurus/MkDocs · Postgres RLS · Dockerfile/compose.

Alasan tiap penolakan ada di ADR-0001 dan ADR-0007. Kalau mau membuka ulang salah satunya, tulis ADR baru — jangan diam-diam menambah dependensi.
