# PRD — IntraDocs v1.0 (MVP)

| | |
| --- | --- |
| Versi | 0.1 (draft, menunggu review tim) |
| Timeline | 4 minggu |
| Tim | 2 orang |
| Status | Perencanaan |

## 1. Masalah

Dokumentasi teknis, SOP, dan panduan aplikasi internal tersebar di share drive, chat, email,
dan kepala orang. Akibatnya:

1. Pertanyaan yang jawabannya sudah tertulis tetap masuk ke helpdesk dan grup chat.
2. Pencarian gagal karena istilah penanya berbeda dari istilah di dokumen
   ("ganti sandi domain" vs "reset password Active Directory").
3. Tidak jelas dokumen mana yang masih berlaku — banyak versi beredar tanpa pemilik.
4. Dokumen sensitif tersimpan di tempat yang kontrol aksesnya lemah atau tidak terekam.

> **Baseline yang harus diukur sebelum Minggu 2** (pemilik: Orang B): jumlah pertanyaan
> berulang per bulan untuk 5 topik teratas. Tanpa baseline, dampak proyek tidak bisa
> dibuktikan di presentasi akhir.

## 2. Tujuan

1. Satu tempat kanonik untuk dokumentasi internal, tersimpan sebagai Markdown.
2. Pengguna menemukan jawaban dalam < 1 menit, lewat search atau AI chat.
3. Jawaban AI **kredibel**: bersumber dari dokumen resmi, bersitasi, dan jujur ketika
   tidak tahu.
4. Akses terkontrol per dokumen dan terekam — lolos review keamanan internal.
5. Kontributor non-developer bisa menambah knowledge tanpa pelatihan khusus.

## 3. Persona

| Persona | Kebutuhan | Sukses berarti |
| --- | --- | --- |
| **Pencari info** (mayoritas karyawan) | Jawaban cepat, tidak mau membaca 22 halaman PDF | Dapat jawaban + tautan ke bagian dokumen yang tepat |
| **Kontributor** (IT Ops, engineer) | Menaruh runbook/SOP tanpa ribet | Upload berkas → jadi dokumen rapi < 10 menit |
| **Peninjau / pemilik dokumen** | Menjaga akurasi & kewenangan | Antrean review jelas, tahu apa yang kedaluwarsa |
| **Admin Knowledge** | Tata kelola taksonomi, user, klasifikasi | Bisa audit siapa mengakses apa |

## 4. Scope MVP

| Kemampuan | Status | Catatan |
| --- | --- | --- |
| Auth (akun lokal + invite link) + 4 role | **Wajib** | OIDC ter-wire di balik flag |
| RBAC: 4 role x 4 klasifikasi + cakupan unit | **Wajib** | Filter di dalam SQL |
| CRUD dokumen Markdown + viewer + metadata + versi | **Wajib** | |
| Katalog: kategori hierarkis + label + filter | **Wajib** | |
| Upload → konversi ke Markdown (MD, TXT, DOCX, PDF ber-text-layer) | **Wajib** | Preview & perbaikan manual wajib |
| Alur review 1 tingkat → publish | **Wajib** | |
| Hybrid search (full-text + vector, RRF) + filter | **Wajib** | |
| AI chat: retrieval terfilter izin, sitasi wajib, abstain eksplisit | **Wajib** | |
| Audit log akses dokumen `restricted`/`secret` | **Wajib** | |
| Eval harness (30+ pertanyaan, hit@5 + groundedness) | **Wajib** | Gerbang kualitas, bukan opsional |
| Peringkat "paling banyak dibaca" | Sederhana | Satu query `COUNT` dari `document_view` |
| AI auto-fill metadata saat upload | Nice-to-have | Minggu 3, kalau RAG sudah stabil |
| Peringatan dokumen melewati periode tinjau ulang | Nice-to-have | Murah, dampak trust besar |

## 5. Non-goals (eksplisit — jangan dibangun di v1.0)

Mockup menampilkan lebih banyak dari yang bisa dibangun 2 orang dalam 4 minggu. Berikut
yang **sengaja tidak** dikerjakan, beserta alasannya:

| Tidak dibangun | Alasan | Rencana |
| --- | --- | --- |
| OCR untuk berkas hasil pindai | Dependency berat, kualitas tidak terkontrol | v1.2 |
| Upload ZIP, XLSX, PPTX | Nilai per usaha rendah | v1.2 |
| Deteksi duplikasi dokumen | Sudah punya embedding; nanti ~50 baris | v1.1 |
| Approval berjenjang + auto-routing | Workflow engine adalah lubang waktu | v1.1 |
| Dashboard analytics | Satu angka di halaman admin sudah cukup | v1.1 |
| Alur "permintaan akses" untuk dokumen terbatas | MVP: tampilkan pemilik dokumen | v1.1 |
| Editor kolaboratif realtime, komentar inline | Overkill | — |
| Aplikasi mobile, fine-tuning model, i18n | Di luar kebutuhan | — |
| Git sebagai source of truth | Lihat ADR-0002 | Export satu arah saja |

**Aturan perubahan scope:** menambah satu item dari daftar ini harus disertai
pengurangan item lain yang setara. Tidak ada penambahan tanpa trade-off tertulis.

## 6. User stories & acceptance criteria

Format: setiap story punya minimal satu acceptance criteria yang bisa dijadikan nama file test.
Yang tidak bisa dites tidak masuk sprint.

### Autentikasi & akses

**US-001 — Login akun lokal**
Sebagai pengguna, saya bisa masuk dengan email dan password.
- AC: password disimpan dengan argon2; 5 kali gagal → rate limit 15 menit; session cookie `httpOnly` + `secure`.
- Test: `tests/auth/login.spec.ts`

**US-002 — Admin mengundang pengguna**
Sebagai admin, saya membuat invite dengan role, clearance, dan unit; sistem memberi link bertoken.
- AC: token hanya disimpan sebagai hash; kedaluwarsa 7 hari; sekali pakai; link bisa dicopy manual (tanpa dependency email di MVP).
- Test: `tests/auth/invite.spec.ts`

### RBAC (paling kritis)

**US-010 — Katalog mengikuti izin**
Sebagai pembaca dengan clearance `internal`, saya hanya melihat dokumen yang boleh saya lihat.
- AC: dokumen `restricted`/`secret` tidak muncul di daftar, search, sitemap, maupun API response.
- Test: `tests/rbac/catalog-visibility.spec.ts`

**US-011 — Search mengikuti izin**
- AC: filter izin diterapkan di dalam SQL kedua kanal retrieval (full-text dan vector), bukan setelah query.
- Test: `tests/rbac/search-visibility.spec.ts`

**US-012 — AI chat tidak membocorkan apa pun**
Sebagai pembaca tanpa kewenangan, ketika saya bertanya hal yang jawabannya hanya ada di
dokumen `restricted`, sistem tidak menampilkan isi, judul, maupun snippet dokumen tersebut.
- AC: jawaban berupa abstain eksplisit; tercatat satu baris `ai_query` tanpa kebocoran konten; tidak ada `chunk_id` dari dokumen terlarang di `retrieved_chunk_ids`.
- Test: `tests/rbac/ai-retrieval-leak.spec.ts` — **test pertama yang ditulis di proyek ini**

**US-013 — Akses dokumen sensitif terekam**
- AC: setiap akses dokumen `restricted`/`secret` menghasilkan baris `audit_log` berisi aktor, waktu, dan jalur akses (katalog/search/chat/langsung).
- Test: `tests/rbac/audit-log.spec.ts`

### Konten

**US-020 — Unggah berkas menjadi Markdown**
Sebagai kontributor, saya mengunggah PDF/DOCX/MD dan mendapat draft Markdown yang bisa saya perbaiki.
- AC: berkas asli tersimpan dengan checksum; hasil konversi ditampilkan untuk diperiksa; publish diblokir sebelum ada pemeriksaan manusia; berkas > 50 MB atau format tak didukung ditolak dengan pesan jelas.
- Test: `tests/ingest/convert.spec.ts`

**US-021 — Metadata & klasifikasi**
- AC: kategori, klasifikasi, dan pemilik dokumen wajib diisi sebelum submit review; label opsional dan boleh banyak.

**US-022 — Review dan publikasi**
- AC: hanya `reviewer` pada kategori terkait atau `admin` yang bisa menyetujui; publikasi membuat baris versi baru yang immutable; hanya versi `published` yang masuk index.
- Test: `tests/review/publish.spec.ts`

**US-023 — Membaca dokumen**
- AC: Markdown dirender di server melalui `rehype-sanitize`; menampilkan klasifikasi, pemilik, versi, tanggal update, dan status verifikasi; dokumen melewati periode tinjau ulang menampilkan peringatan.
- Test: `tests/render/sanitize.spec.ts`

### Search & AI

**US-030 — Hybrid search**
- AC: pencarian identifier persis (misal `SOP-IT-014`) muncul di peringkat 1; pencarian parafrase menemukan dokumen yang relevan; p95 < 500 ms pada 10.000 dokumen.
- Test: `tests/search/hybrid.spec.ts`, `tests/search/rrf.spec.ts`

**US-031 — Jawaban AI bersitasi**
- AC: setiap kalimat yang memuat klaim punya penanda sitasi; sitasi bisa diklik ke bagian dokumen (`heading_path`); tidak ada sitasi ke dokumen yang tidak boleh dibuka pengguna.
- Test: `tests/ai/citations.spec.ts`

**US-032 — Abstain yang jujur**
- AC: kalau skor retrieval tertinggi di bawah threshold, sistem menjawab "tidak ditemukan di knowledge base", tetap menampilkan dokumen yang mungkin relevan, dan menawarkan pengajuan dokumen baru.
- Test: `tests/ai/abstain.spec.ts`

**US-033 — Degradasi berjenjang**
Sebagai pengguna, ketika layanan AI mati, saya masih bisa mencari dan membaca dokumen.
- AC: kegagalan provider AI tidak mengembalikan error pada rute search/katalog; UI chat menampilkan status tidak tersedia.
- Test: `tests/ai/degradation.spec.ts`

## 7. Metrik sukses (diukur di Minggu 4)

**Adopsi** — pengguna aktif mingguan >= 40% target pengguna dalam 30 hari; >= 30 dokumen terpublikasi; >= 5 kontributor aktif selain tim inti.
**Kualitas AI** — hit@5 >= 0,85 pada eval set; 0 klaim tanpa sitasi; abstain benar >= 90% untuk pertanyaan di luar korpus; thumbs-up >= 70%.
**Performa** — p95 search < 500 ms; p95 token pertama AI < 3 s; LCP < 2,5 s.
**Keamanan** — 0 insiden akses tidak sah; 100% akses dokumen sensitif terekam audit.
**Dampak** — penurunan pertanyaan berulang untuk 5 topik teratas dibanding baseline.

> Angka "96% akurasi AI" pada mockup adalah placeholder. **Jangan** menyebut angka akurasi
> apa pun ke stakeholder sebelum eval harness berjalan.

## 8. Asumsi & dependensi

1. Provider LLM/embedding yang boleh memproses data internal **belum ditentukan**.
   Sampai ada persetujuan, pengembangan memakai dokumen sintetis. Lihat ADR-0003.
2. SSO korporat belum tersedia; MVP memakai akun lokal + invite.
3. Target deploy produksi belum final; aplikasi harus tetap bisa dijalankan dengan Docker.
4. Dokumen nyata untuk pilot dan pemiliknya belum dikonfirmasi.
5. Peserta UAT (5–10 orang) belum ditunjuk.

Semua item di atas dilacak di `docs/decisions-open.md` dengan penanggung jawab dan tenggat.

## 9. Risiko utama

| Risiko | Dampak | Mitigasi |
| --- | --- | --- |
| Persetujuan AI provider tertunda | Fatal | Abstraksi provider, 1024 dim, `AI_MAX_CLASSIFICATION`, seed sintetis |
| Tidak ada konten nyata saat rilis | Fatal | Content seeding jadi task Minggu 1 dengan pemilik bernama |
| Kebocoran RBAC lewat AI | Fatal + reputasi | Filter di query, test kebocoran otomatis, audit log |
| Jawaban AI ngawur | Fatal (kepercayaan hilang permanen) | Sitasi wajib, abstain, eval harness |
| Scope creep dari mockup | Tinggi | Non-goals tertulis, feature freeze hari ke-24 |
| Kualitas konversi PDF buruk | Sedang | Pemeriksaan manusia wajib, OCR di luar scope |
| Utang teknis dari kode AI | Tinggi | AGENTS.md, CI sebagai gerbang, PR kecil, review silang |
| Bus factor 2 orang | Sedang | Semua di repo; uji "clone & run" di Minggu 2 |
