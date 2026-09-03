# Sprint 6 hari — MVP

Target: MVP tercapai dan bisa diklik pada **Jumat, 11 September 2026**.
Definisi "tercapai" ada di `docs/scope-mvp.md`. Yang dipotong dan alasannya ada di ADR-0007.

Aturan sprint:

1. **Setiap hari harus berakhir dengan sesuatu yang jalan**, walaupun kecil.
2. **Kalau sebuah hari meleset, potong fitur — jangan geser tenggat.** Urutan potongnya ada
   di bawah.
3. Deploy ke Vercel sejak hari pertama. Jangan menunggu "nanti kalau sudah rapi".

## Hari 1 — Jumat 4 Sep · fondasi

| | Orang A | Orang B |
| --- | --- | --- |
| Kerja | Scaffold Next.js + TS strict + Drizzle + Neon (pgvector) · skema 9 tabel + migrasi · seed 5 user, 6 kategori, 9 label | Design token dari CSS variable mockup → `tailwind.config.ts` · sprite ikon · `AppShell` (topbar + sidebar) · halaman `/dev/tokens` |

**Sinkron pagi (30 menit, wajib):** sepakati skema dan `docs/api-contract.md` bersama.
Setelah ini keduanya jalan paralel tanpa saling menunggu.

**Akhir hari:** aplikasi kosong ter-deploy di Vercel, terhubung Neon, token terlihat di
`/dev/tokens`.

## Hari 2 — Senin 7 Sep · identitas & izin

| | Orang A | Orang B |
| --- | --- | --- |
| Kerja | Auth.js credentials · `visibleDocumentsFilter` · `can.ts` · **`tests/rbac/` ditulis lebih dulu** | Halaman katalog (mockup s3): tabel, kolom, badge status, filter chip · sidebar kategori |

**Akhir hari:** login sebagai tiga role, katalog menampilkan daftar yang berbeda, test RBAC hijau.

Hari ini adalah hari terpenting di sprint. Kalau RBAC belum benar di sini, jangan lanjut ke
hari 3 — semua yang dibangun di atasnya akan ikut salah.

## Hari 3 — Selasa 8 Sep · konten

| | Orang A | Orang B |
| --- | --- | --- |
| Kerja | Endpoint dokumen (CRUD + publish → `document_version` baru) · `audit_log` untuk akses `restricted` | Viewer dokumen (mockup s4): render server + sanitize, TOC hierarkis, panel metadata, badge klasifikasi · form buat/edit Markdown |
| Paralel | — | **20–25 dokumen sintetis** — dicicil sepanjang hari |

**Akhir hari:** katalog terisi konten yang realistis, dokumen bisa dibaca dengan rapi.

## Hari 4 — Rabu 9 Sep · search

| | Orang A | Orang B |
| --- | --- | --- |
| Kerja | `tsvector` + `pgvector` + **RRF** · embed saat publish · endpoint search dengan filter izin | Halaman hasil pencarian (mockup s2): kartu hasil, snippet ter-highlight, panel filter, state kosong |

**Akhir hari:** cari `SOP-IT-014` (identifier persis) dan "cara ganti sandi domain"
(parafrase) — keduanya menemukan dokumen yang benar.

## Hari 5 — Kamis 10 Sep · AI

| | Orang A | Orang B |
| --- | --- | --- |
| Kerja | `lib/ai/provider.ts` · endpoint RAG streaming · kontrak jawaban + jalur abstain · guard `AI_MAX_CLASSIFICATION` | Kotak jawaban AI di halaman search · halaman chat · chip sitasi yang bisa diklik ke bagian dokumen · state loading/error/abstain |

**Akhir hari:** tanya AI → jawaban bersitasi. Tanya di luar korpus → mengaku tidak tahu.

## Hari 6 — Jumat 11 Sep · bukti & rapikan

| | Orang A | Orang B |
| --- | --- | --- |
| Kerja | Eval 10 pertanyaan · jalankan, perbaiki retrieval berdasarkan angka · index DB · `/api/health` | Lengkapi state kosong/error/tanpa izin · navigasi keyboard · kontras · cek bundle · `README` + skrip demo |

**Akhir hari:** checklist "MVP tercapai" di `docs/scope-mvp.md` seluruhnya tercentang.

## Urutan potong kalau waktu habis

Dipotong dari atas. Jangan improvisasi urutan ini saat panik di hari ke-5:

1. Form buat/edit dokumen di UI — konten cukup dari seed
2. Halaman chat terpisah — cukup kotak jawaban AI di halaman search
3. Filter selain kategori
4. `audit_log` write
5. TOC hierarkis — cukup daftar heading datar

**Tidak boleh dipotong dalam keadaan apa pun:** `visibleDocumentsFilter` + test kebocoran,
sitasi, jalur abstain, dokumen sintetis, deploy yang bisa diakses.

## Fase 2 — setelah MVP diterima

Urutan yang disarankan, satu per satu, masing-masing satu PR:

1. Konversi PDF/DOCX + job queue + object storage (paket alami, ~2 hari)
2. Alur review/approval + konsol antrean (mockup s6)
3. Alur invite + konsol pengguna (mockup s8)
4. UI riwayat versi + rollback
5. Konsol taksonomi (mockup s7) + kategori hierarkis
6. `Dockerfile` + `docker-compose` + uji portabilitas
7. Rate limit + observability + backup/restore
8. Reranker, deteksi duplikat, permintaan akses, sinkron AD

Roadmap empat minggu versi awal tetap valid sebagai peta jalan setelah MVP — hanya urutannya
yang berubah, bukan isinya.
