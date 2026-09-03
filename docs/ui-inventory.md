# Inventaris UI

Mockup `intradocs-mockup_1.html` adalah **spesifikasi visual**, bukan wireframe: taksonomi,
klasifikasi, dan alur approval sudah dipikirkan di dalamnya. Jangan merancang ulang dari nol.

Seluruh delapan screen sudah dibaca dan diverifikasi langsung dari file.

## Langkah pertama sebelum menulis komponen

Mockup mendefinisikan design system lengkap sebagai CSS variable (`--blue-600`, `--ink`,
`--muted`, `--line`, `--r`, `--sh-md`, dan lain-lain). **Ekstrak semuanya menjadi design token
di `tailwind.config.ts` di hari 1, sebelum komponen apa pun dibuat.**

Alasan: kalau token belum ada, setiap agent AI akan mengarang warna dan spacing sendiri di
setiap file. Itu penyebab utama UI terlihat seperti hasil tempelan. Setelah token ada, aturan
"nol hex hardcoded" di `AGENTS.md` bisa ditegakkan.

Ikon di mockup memakai pola SVG sprite (`<use href="#ic-*">`). Pertahankan pola itu — tidak
perlu icon library.

## Peta screen

| # | Screen | URL di mockup | MVP? | Hari |
| --- | --- | --- | --- | --- |
| s1 | Help Center — hero search, 6 kartu kategori, paling dibaca, banner AI | `/help-center` | sebagian | 4 |
| s2 | Hasil pencarian — filter, kotak jawaban AI + sitasi, kartu hasil | `/cari` | **ya** | 4–5 |
| s3 | Katalog dokumen — tabel, filter chip, pagination | `/katalog` | **ya** | 2 |
| s4 | Baca dokumen — nav dokumen hierarkis, isi, metadata | `/{kategori}/{slug}` | **ya** | 3 |
| s5 | Unggah Knowledge — wizard 4 langkah | `/unggah` | disederhanakan | 3 |
| s6 | Antrean approval admin | `/admin/approval` | tidak — fase 2 | — |
| s7 | Kategori & Label + aturan otomatis | `/admin/kategori-label` | tidak — fase 2 | — |
| s8 | Pengguna & RBAC + sinkron AD | `/admin/pengguna` | tidak — fase 2 | — |

## Detail terverifikasi per screen

**s1 Help Center.** Hero "Ada yang bisa kami bantu?", search besar + tombol Tanya AI
(`⌘K`, `⌥I`), chip topik populer, 6 kartu kategori dengan jumlah dokumen, bagian "Paling
banyak dibaca bulan ini", statistik hero, footer dengan penanda klasifikasi.

**s2 Hasil pencarian.** Filter: kategori, label, format berkas, terakhir diperbarui, status.
Header hasil menyebut "pencarian semantik + kata kunci". Kotak `.ai-answer` dengan sitasi
bernomor `[1][2][3]` + chip sumber + "Lanjutkan di chat". Kartu hasil dengan breadcrumb,
snippet ber-`<mark>`, versi, pemilik, badge Terverifikasi. Catatan mockup menegaskan tiga
hal: hybrid BM25 + vector, sitasi yang bisa diklik, dan **dokumen terbatas tetap terlihat
judulnya tetapi isinya terkunci** dengan opsi ajukan permintaan akses.

**s3 Katalog.** Tabel dengan kolom Dokumen / Kategori / Label / Versi / Pemilik / Diperbarui /
Status. Baris menampilkan badge tipe berkas (PDF, MD, DOC, XLS, PPT, TXT), jumlah halaman,
ukuran, jumlah dibaca. Baris "Menunggu Approval" diberi latar amber. Sidebar tiga bagian:
Navigasi, Kontribusi, Kategori — dengan penghitung. Catatan mockup menyebut versioning
dengan rollback.

**s4 Baca dokumen.** Panel `.docnav` kiri berisi **daftar isi hierarkis bernomor**
(Bagian 1 / 1.1 / 2.1 / 2.1.1), tombol "Tanya AI tentang halaman ini" di topbar, favorit,
unduh, dan search dalam dokumen. Ini layar yang paling sering dilihat pengguna — kualitasnya
menentukan persepsi produk.

**s5 Unggah.** Wizard 4 langkah: Unggah Berkas → Metadata & Klasifikasi → Pratinjau → Kirim
untuk Approval. Dropzone maks 50 MB. Blok `.ai-extract` untuk metadata otomatis. Field:
Judul, Ringkasan, Kategori, Sub-kategori, Label, **Klasifikasi Keamanan**, Pemilik Dokumen,
**Periode Tinjau Ulang**, Reviewer/Approver.

**s7 Kategori & Label.** 42 kategori 3 tingkat, 68 label. Panel "Aturan Otomatis" yang
menghubungkan taksonomi dengan kebijakan akses dan approval — ide bagus untuk fase 2. Saran
perapian taksonomi berbasis AI (deteksi label mirip dan label mati).

**s8 Pengguna & RBAC.** Menyebut **6 role**: Super Admin, Admin Knowledge, Reviewer, dan tiga
lainnya — ditambah **sinkronisasi otomatis dari Active Directory**. MVP memakai **4 role**
(`viewer`, `contributor`, `reviewer`, `admin`) tanpa sinkron AD. Pemetaannya lurus: Super
Admin dan Admin Knowledge digabung menjadi `admin`, Reviewer tetap, sisanya `contributor`
dan `viewer`.

## Dua koreksi model terhadap mockup

**1. Klasifikasi bukan status.** Screen s3 menampilkan "Terbatas" dan "Kedaluwarsa" di kolom
Status, bersama "Draft", "Menunggu Approval", dan "Published". Di skema, ketiganya hal
berbeda: `status` adalah siklus hidup (`draft`/`in_review`/`published`/`archived`),
`classification` adalah kepekaan (`public`/`internal`/`restricted`/`secret`), dan
"Kedaluwarsa" adalah **turunan** dari `updated_at` + `review_period_days`. Menggabungkan
ketiganya menjadi satu kolom akan menimbulkan kondisi mustahil seperti dokumen yang
"Terbatas" tetapi belum `published`. Tampilkan sebagai satu kolom gabungan di UI kalau mau —
tetapi simpan sebagai tiga hal terpisah.

**2. Satu kategori per dokumen, label bebas dan banyak.** Catatan mockup menyatakan ini
eksplisit, dan ini keputusan yang benar. Pertahankan di skema: `document.category_id` sebagai
foreign key tunggal, `document_label` sebagai relasi banyak-ke-banyak.

## Komponen bersama yang dibangun lebih dulu

`AppShell` (topbar + sidebar) · `SearchInput` · `Pill` / `Tag` · `ClassificationBadge` ·
`StatusBadge` · `FileTypeBadge` · `Card` · `DataTable` · `DocNav` · `EmptyState` ·
`ErrorState` · `PermissionDeniedState` · `CitationChip` · `MarkdownViewer`.

Bangun ini di hari 1–2 sebelum menyalin screen satu per satu. Menyalin screen tanpa komponen
bersama menghasilkan duplikasi yang mahal untuk dirapikan.

## Yang sengaja tidak ditiru

Angka statistik pada hero ("1.284 dokumen", "42 kategori", "318 pengguna", **"96% akurasi
jawaban AI"**) adalah placeholder. Tampilkan angka nyata dari database, atau jangan tampilkan
sama sekali. **Jangan pernah menampilkan angka akurasi AI yang tidak diukur oleh eval
harness** — itu satu-satunya cara cepat menghancurkan kredibilitas produk yang justru
dibangun untuk kredibel.

Deteksi duplikasi, approval berjenjang, OCR, XLSX/PPTX/ZIP, auto-label AI, saran taksonomi,
dan sinkron AD muncul di mockup tetapi bukan bagian MVP. Lihat `docs/scope-mvp.md`.
