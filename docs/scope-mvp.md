# Scope MVP — sprint 6 hari

Dokumen ini **mengikat**. Kalau ada konflik dengan dokumen lain (termasuk
`docs/context-pack.md`, `docs/architecture.md`, atau `docs/prd.md`), yang berlaku adalah
dokumen ini sampai MVP selesai. Alasan pemotongan ada di
`docs/adr/0007-mvp-scope-reduction.md`.

Konteks: dua orang, enam hari kerja, dikerjakan dengan bantuan agent AI. Statusnya
eksperimen — tetapi harus rapi dan layak dipakai kalau ternyata diteruskan.

## Prinsip pemotongan

Yang dipertahankan adalah hal yang **tidak bisa ditambahkan belakangan tanpa membongkar
fondasi**, ditambah hal yang membentuk cerita demo. Yang dipotong adalah hal yang bisa
ditempelkan nanti tanpa mengubah skema atau arsitektur.

Satu kalimat penguji: *kalau fitur ini tidak ada, apakah demo masih membuktikan bahwa
dokumentasi internal bisa ditanyakan ke AI secara kredibel dan aman?* Kalau masih, potong.

## MASUK — dikerjakan minggu ini

| Fitur | Kenapa tidak bisa ditunda |
| --- | --- |
| Auth akun lokal (5 user hasil seed) | Tanpa identitas, RBAC tidak bisa didemokan |
| 4 role × 4 klasifikasi + `visibleDocumentsFilter` di SQL | Ini pembeda utama produk, dan menyusupkannya belakangan berarti membongkar setiap query |
| Test kebocoran RBAC | Satu-satunya bukti bahwa klaim keamanan benar |
| Katalog + kategori + label + filter | Layar utama (mockup s3) |
| Viewer dokumen + TOC + metadata + badge klasifikasi | Layar yang paling sering dilihat (mockup s4) |
| Buat/edit draft Markdown → publish | Alur kontribusi minimum |
| 20–25 dokumen sintetis | Tanpa konten, tidak ada yang bisa didemokan |
| Hybrid search: FTS + pgvector + RRF | Inti "kredibel"; sekitar 100 baris |
| AI chat: RAG + sitasi + abstain | Fitur utama yang diminta |
| Eval 10 pertanyaan | Bukti angka, bukan klaim |
| Deploy ke Vercel + Neon | Harus bisa diklik orang lain |
| Design token dari mockup | Membuat hasilnya terlihat profesional sejak hari pertama, biayanya setengah hari |
| Struktur modular (`lib/` logic murni) | Gratis kalau dari awal, mahal kalau menyusul |

## KELUAR — fase 2, sudah dipikirkan tapi tidak dikerjakan sekarang

| Yang dipotong | Kenapa aman ditunda | Biaya menambahkan nanti |
| --- | --- | --- |
| Konversi PDF/DOCX → Markdown | Minggu ini terima `.md` dan `.txt` saja. Ini sumber bug terbesar dan risiko kualitas tertinggi | 1–2 hari, terisolasi di `lib/parsers/` |
| Job queue (`job` + `/api/jobs/tick` + cron) | 25 dokumen bisa di-embed langsung saat publish. Satu dokumen 10 chunk = satu panggilan API | 0,5 hari, dibutuhkan hanya saat upload PDF masuk |
| Object storage (R2/MinIO) | Tanpa upload berkas biner, tidak ada yang perlu disimpan | 2 jam, API S3 sudah standar |
| Alur invite | 5 user hasil seed cukup untuk demo | 0,5 hari |
| Alur review/approval berjenjang | Cukup tombol publish oleh reviewer/admin | 1 hari |
| UI riwayat versi + rollback | Tabel `document_version` tetap dibuat, jadi datanya sudah ada | 0,5 hari, murni UI |
| `document_grant` + alur permintaan akses | Klasifikasi + unit sudah cukup untuk mendemokan RBAC | 0,5 hari |
| Kategori hierarkis 3 tingkat | Enam kategori datar + label sudah cukup pada 25 dokumen | 0,5 hari |
| Konsol admin (taksonomi, user, audit) | Kelola lewat SQL/script dulu | 1–2 hari |
| Docker + docker-compose | Vercel + Neon lebih cepat untuk minggu ini. Aturan portabilitas tetap ditegakkan | 0,5 hari, dan itulah gunanya aturan |
| Rate limit, Sentry, `pino` | Belum ada beban dan belum ada pengguna nyata | 2 jam |
| Playwright E2E | Vitest untuk RBAC sudah menutup risiko terbesar | 0,5 hari |
| Sinkron Active Directory (mockup s8) | Butuh akses infrastruktur yang tidak kita punya | Fase 3 |
| Auto-label AI, saran perapian taksonomi (mockup s7) | Fitur pemanis | Fase 2 |
| Deteksi duplikat, OCR, XLSX/PPTX, notifikasi email | Semuanya pemanis | Fase 2–3 |

## Tetap ditegakkan walaupun scope dipotong

Ini yang membedakan "MVP rapi" dari "MVP asal jadi":

- **Nol logic izin di prompt LLM.** Filter tetap di SQL. Tidak ada pengecualian.
- **Skema tetap lengkap** untuk `document_version`, `audit_log`, `chunk.embedding_model`,
  `content_hash` — walaupun UI-nya belum ada. Menambah kolom belakangan pada tabel besar
  jauh lebih mahal daripada membuatnya kosong sekarang.
- **Embedding tetap 1024 dimensi** (ADR-0006).
- **`lib/` tetap berisi logic murni tanpa dependensi framework**, dan tetap diuji.
- **Nol warna hardcoded**; semua lewat token.
- **Nol dokumen Telkom asli.** Seluruh konten sintetis dan fiktif.
- **TypeScript strict, tanpa `any`.**
- **Setiap tampilan punya state loading, kosong, error, dan tanpa izin.**

## Definisi "MVP tercapai"

MVP dinyatakan selesai bila seluruh pernyataan berikut benar, diperiksa dari URL produksi:

- [ ] Login sebagai tiga role berbeda menghasilkan katalog yang berbeda, sesuai matriks RBAC
- [ ] Membuka dokumen di luar izin menghasilkan 404, bukan 403
- [ ] Search satu identifier persis (`SOP-IT-014`) dan satu parafrase, keduanya menemukan
      dokumen yang benar
- [ ] Tanya AI → jawaban muncul dengan sitasi yang bisa diklik ke bagian dokumen
- [ ] Tanya sesuatu di luar korpus → AI menjawab tidak tahu, tidak mengarang
- [ ] Tanya sebagai `viewer` tentang dokumen `restricted` → tidak ada isi maupun judul yang bocor
- [ ] `pnpm test` hijau, termasuk seluruh test di `tests/rbac/`
- [ ] `pnpm eval` mencetak angka, dan `rbac_leak = 0`
- [ ] Aplikasi bisa dibuka orang lain lewat URL tanpa dibantu
