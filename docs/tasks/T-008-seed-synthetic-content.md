# T-008: Korpus sintetis dan baseline eval

Pemilik B. Penulisan Markdown/eval dapat dimulai tanpa kode; integrasi seed menunggu T-003 dan pipeline T-006b.

## Baca tambahan

Scope bagian korpus, eval README, tabel seed matriks RBAC.

## Subtask

- T-008a: kontrak frontmatter dan batch 1 korpus.
- T-008b: batch lanjutan sampai total 20–25 dokumen; satu PR per batch <=400 baris.
- T-008c: integrasi scripts/seed.ts memakai pipeline publikasi dan fixture akun T-003; sepuluh kasus eval final.

Lokasi db/seed/documents/. Frontmatter divalidasi zod: title, slug unik, categorySlug, labels, classification, owner fixture, reviewPeriodDays; metadata tanggal/versi sintetis bila diperlukan. Tentukan schema bersama T-006, bukan dua parser.

Enam kategori: Infrastruktur & Jaringan; Keamanan Informasi; Aplikasi Internal; SOP & Proses Bisnis; Onboarding & SDM; Data & Integrasi. Semua klasifikasi terwakili. Sertakan near-duplicate, satu kedaluwarsa, tabel, kode, heading tiga tingkat pada minimal tiga dokumen, serta restricted pada dua kategori.

## Kriteria terima

- [ ] Total 20–25 dokumen fiktif; tidak menyalin helpdesk, chat internal, data Telkom asli atau kredensial sungguhan.
- [ ] Seed ulang idempotent: tidak menggandakan akun/dokumen/versi/chunk; perubahan konten menggunakan pipeline versi.
- [ ] Tepat enam akun sesuai matriks; Fajar nonaktif, Viewer Demo aktif, reviewer Dwi cakupan sempit.
- [ ] Sepuluh kasus baseline terdiri dari delapan answerable (termasuk dua parafrase) dan dua outside-corpus wajib abstain. Tidak menyatakan kasus abstain bisa dijawab dari korpus.
- [ ] expect_docs mengacu slug yang benar-benar ada dan boleh dibaca actor; kasus abstain memakai daftar kosong.
- [ ] Kasus RBAC lintas role dan multi-turn disiapkan terpisah sebagai test integrasi, tidak merusak denominasi baseline retrieval.
- [ ] Teks dibaca manusia dan struktur layak dokumentasi, bukan sekadar lolos parser.

Angka/nama mockup bukan statistik produksi. Semua bukti kualitas menyebut ukuran sampel.
