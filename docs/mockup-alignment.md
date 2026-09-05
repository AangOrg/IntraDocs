# Kesesuaian Rencana dengan Mockup

Diperiksa 4 September 2026, setelah layar 9-11 dibaca utuh. Layar 1-8 ditambahkan dan diverifikasi 5 September 2026. Berkas ini adalah matriks keterlacakan antara mockup dan rencana MVP, sekaligus daftar koreksi yang keluar dari pemeriksaan itu.

## Koreksi inventaris

| Yang tertulis sebelumnya | Yang benar |
| --- | --- |
| Mockup punya 8 layar | **11 layar.** Layar 9 AI Assistant, 10 Dashboard, 11 Arsitektur & Roadmap |
| 6 role | **5 role bawaan + kemampuan membuat role kustom.** Judul layar 8 menulis "6 role", tetapi kartu yang ada lima: Super Admin (3 pengguna), Admin Knowledge (8), Reviewer (21), Contributor (96), Viewer (184) |
| Izin satu dimensi (role + clearance) | **Dua dimensi:** role menentukan *apa*, cakupan kategori menentukan *di mana* |

`docs/ui-inventory.md` sudah memuat kesebelas layar beserta rutenya, jadi berkas ini tidak lagi perlu menambalnya. Yang tinggal di sini adalah alasan dan keputusannya, bukan daftar layarnya.

Angka pada mockup tidak konsisten satu sama lain. Jumlah pengguna per role berjumlah 312 sementara header layar 8 menulis 318 aktif. Halaman muka menyebut "1.284 dokumen terverifikasi", tetapi donut layar 10 memecah 1.284 dokumen menjadi 950 published, 128 menunggu approval, 129 draft, dan 77 kedaluwarsa — jadi angka yang sama dipakai untuk "terverifikasi", "aktif", dan "total". Ini menegaskan aturan yang sudah ada: seluruh angka pada mockup adalah pengisi tempat dan tidak boleh ditampilkan sebagai fakta.

## Layar 1-8

Pemeriksaan ini menutup lubang yang disebut ADR-0011: sampai 4 September berkas ini hanya mencakup layar 9-11, sehingga pertentangan paling serius — pil "Terbatas — akses via permintaan" di layar 2 — tidak tercatat di mana pun.

Isi bagian ini **diverifikasi langsung terhadap `intradocs-mockup_1.html`** pada 5 September 2026. `AGENTS.md` melarang membuka berkas itu utuh; larangan dicabut pemilik repo khusus untuk pemeriksaan ini. Elemen di bawah dikutip dari mockup, bukan disimpulkan dari dokumen lain.

### Layar 1 — Halaman Muka (`/help-center`)

| Elemen di mockup | Status MVP |
| --- | --- |
| Pencarian besar di tengah hero, dengan tombol "Tanya AI" di dalam kotak yang sama | Masuk |
| Enam kartu kategori: Infrastruktur & Jaringan, Keamanan Informasi, Aplikasi Internal, SOP & Proses Bisnis, Onboarding & SDM, Data & Integrasi | Masuk — persis enam kategori seed kita, satu tingkat |
| **Enam chip "Topik yang sering dicari"** | **Ditunda.** Catatan mockup sendiri menyebutnya "chip dinamis yang dihitung dari log pencarian 30 hari terakhir", dan log baru terisi hari 4. Chip yang diisi manual berarti menampilkan topik karangan |
| Empat penghitung hero: 1.284 dokumen, 42 kategori, 318 pengguna, 96% akurasi AI | **Tidak ditampilkan dengan angka mockup.** Kalau ditampilkan, dihitung dari basis data. "96% akurasi AI" khususnya tidak akan ditampilkan — angka mutu kita berasal dari `pnpm eval` |
| Jumlah dokumen per kartu kategori (184, 142, 296, 231, 87, 163) | Masuk sebagai hitungan basis data, **setelah filter izin** (ADR-0011) |
| "Paling banyak dibaca bulan ini" — empat dokumen dengan jumlah pembacaan | **Ditunda** — butuh `document_view`, tabel yang sengaja tidak dibuat di MVP (daftar larangan T-003) |
| "Baru disetujui & dipublikasikan" — tiga dokumen dengan pengunggah dan penyetuju | Sebagian — daftar terbaru masuk lewat `updated_at`. Kolom "disetujui oleh" ikut approval ke Fase 2 |
| Banner AI Assistant dengan contoh tanya jawab bersitasi | Masuk sebagai tautan ke layar 9 |
| Footer "Klasifikasi: Internal" | Masuk — gratis, dan senada dengan pesan produk |

### Layar 2 — Hasil Pencarian (`/search?q=`)

Satu-satunya layar yang isinya bertentangan **langsung** dengan aturan kita, bukan sekadar lebih kaya.

| Elemen di mockup | Status MVP |
| --- | --- |
| Daftar hasil dengan snippet dan istilah tersorot | Masuk |
| **Pil "Terbatas — akses via permintaan"** pada hasil keempat (Kebijakan Password ISMS-POL-07) | **Tidak dibangun** — ADR-0011. Catatan mockup menegaskan maksudnya: "dokumen terbatas tetap terlihat judulnya namun isinya terkunci, dengan opsi ajukan permintaan akses ke pemilik dokumen". Itu justru yang dilarang |
| "41 hasil · ditemukan dalam 0,18 detik" | Masuk, tetapi **dihitung setelah filter izin** (ADR-0011). Angka 41 sendiri pengisi tempat |
| **Kotak `.ai-answer`** di atas daftar hasil, dengan tiga sitasi bernomor | **Tidak dibangun di halaman pencarian** — alasan di bawah |
| Tombol **"Lanjutkan di chat"** di dalam kotak `.ai-answer` | Masuk — dan justru inilah jalan keluarnya: mockup sendiri sudah menyediakan jembatan ke layar 9 |
| Filter: kategori, label, format berkas, terakhir diperbarui, status | Sebagian — kategori saja; sisanya potongan #4. Filter "format berkas" tidak berlaku di MVP karena semua dokumen Markdown |
| Filter "Draft milik saya" | Ditunda bersama form buat dan sunting (potongan #2) |

**Kenapa kotak `.ai-answer` tidak dibangun di layar 2.** `docs/api-contract.md` menetapkan pencarian tidak boleh memanggil provider AI, supaya pencarian tetap hidup ketika AI mati atau kuota habis. Menempelkan jawaban AI di halaman hasil berarti dua jalur jawaban dengan dua tempat sitasi, dua tempat jalur abstain, dan dua tempat yang harus diuji kebocorannya — pada minggu yang tujuan utamanya membuktikan satu jalur AI bekerja rapat. Tombol "Lanjutkan di chat" pada mockup membuat pilihan ini murah: layar 2 menautkan ke layar 9 dengan kueri terbawa, dan komponennya sama kalau nanti disatukan.

### Layar 3 — Katalog Dokumen (`/katalog`)

| Elemen di mockup | Status MVP |
| --- | --- |
| Tabel dengan kolom Dokumen, Kategori, Label, Versi, Pemilik, Diperbarui, Status | Masuk |
| Sakelar tampilan daftar dan grid | Ditunda — satu tampilan cukup |
| Pil filter: Kategori, Label, Format, Pemilik, Status | Sebagian — kategori saja (potongan #4) |
| Ukuran berkas dan jumlah halaman per baris ("14 halaman · 2,4 MB") | Tidak berlaku — dokumen MVP Markdown, tanpa berkas asli |
| Jumlah pembacaan per baris ("4.281 dibaca") | Ditunda — `document_view` tidak dibuat |
| **Sidebar "Favorit Saya" (12) dan "Riwayat Baca"** | **Ditunda** — favorit butuh tabel yang tidak ada di sebelas tabel, riwayat baca butuh `document_view` yang dilarang |
| Sidebar "Draft Saya" (3) dan "Menunggu Review" (2) | Draft ikut potongan #2; Menunggu Review ikut approval ke Fase 2 |
| **Status yang dipakai: Draft, Menunggu Approval, Published, Terbatas, Kedaluwarsa** | **Dikoreksi** — lihat temuan 4. "Terbatas" itu klasifikasi, "Kedaluwarsa" itu turunan tanggal; keduanya bukan status |
| Paginasi "Menampilkan 1–8 dari 1.284 dokumen" | Masuk — dengan `total` yang dihitung setelah filter izin |
| Catatan mockup: versioning dengan opsi *rollback* | Tabel `document_version` ada sejak MVP; UI riwayat dan rollback Fase 2 |

### Layar 4 — Baca Dokumen (`/{kategori}/{slug}`)

| Elemen di mockup | Status MVP |
| --- | --- |
| Tiga kolom: navigasi dokumen, isi, daftar isi "Di halaman ini" | Masuk |
| `DocNav` bertingkat (Bagian 2 → 2.1 → 2.1.1) | Tampilan satu tingkat (potongan #5), tetapi `chunk.heading_path` tetap penuh |
| Sitasi ke "Bagian 2.1.2" dan tautan balik dari jawaban AI | Masuk — bergantung pada `heading_path`, ada di daftar "tidak boleh dipotong" |
| Kepala dokumen: pemilik, versi, "Disetujui Andi Wijaya · 10 Agu 2026" | Sebagian — pemilik dan versi masuk; jejak persetujuan ikut approval ke Fase 2 |
| Callout "Ditinjau ulang setiap 6 bulan", dengan pengingat otomatis 14 hari sebelumnya | Penanda kedaluwarsa masuk (turunan `review_period_days`); pengingat otomatis ditunda — butuh penjadwal |
| "4.281 dibaca" dan "9 menit baca" | Jumlah baca ditunda (`document_view`); estimasi waktu baca boleh dihitung dari panjang teks |
| Tombol "Tanya AI tentang halaman ini" | Masuk sebagai navigasi ke `/ai-assistant` dengan pertanyaan awal terisi. Parameter ruang lingkup jawaban adalah potongan #3 |
| Panel "Riwayat versi" dengan v3.1/v3.0/v2.5 dan tombol "Bandingkan versi" | Fase 2 |
| Ikon favorit dan unduh di topbar | Favorit ditunda; unduh berkas asli tidak berlaku tanpa object storage |
| Umpan balik "Ya / Belum / Usulkan perbaikan" | Umpan balik dua tombol masuk; "Usulkan perbaikan" butuh antrean pemilik dokumen, ditunda |
| Blok kode dan callout di dalam isi dokumen | Masuk — Markdown terender, dibersihkan `rehype-sanitize` |
| Pencatatan `audit_log` saat membaca `restricted`/`secret` | Masuk — persyaratan kepatuhan |

### Layar 5 — Unggah Knowledge (`/unggah`)

| Elemen di mockup | Status MVP |
| --- | --- |
| **Wizard empat langkah:** 1 Unggah Berkas, 2 Metadata & Klasifikasi, 3 Pratinjau, 4 Kirim untuk Approval | **Tidak dibangun.** MVP memakai satu form sederhana, dan form itu sendiri potongan #2 — dokumen MVP masuk lewat `pnpm seed`. Langkah 1 butuh pemroses berkas, langkah 4 butuh approval; keduanya Fase 2 |
| Sembilan jenis masukan: PDF, DOCX/DOC, MD, XLSX, PPTX, TXT, HTML, gambar hasil pindai (OCR), ZIP — maksimal 50 MB | Fase 2 — MVP hanya `.md` dan `.txt`. Daftar dan batas ini dicatat di `docs/api-contract.md` bagian Upload & ingest |
| Panel `.ai-extract`: usulan kategori, usulan label, dan **"Deteksi 4 dokumen serupa"** | Usulan metadata jadi tambahan opsional; deteksi duplikasi ditunda — butuh perbandingan embedding antar dokumen |
| Form metadata: Judul, Ringkasan, Kategori, **Sub-kategori**, Label, Klasifikasi Keamanan, Pemilik Dokumen, **Periode Tinjau Ulang** | Masuk kalau form dibangun. Sub-kategori ditunda (kategori MVP satu tingkat, kolom induk sudah ada). Periode tinjau ulang membenarkan `review_period_days` sejak MVP |
| Reviewer/Approver dan sakelar "Perlu persetujuan berjenjang", yang menyala otomatis karena label Kritikal | Fase 2 bersama approval |
| Catatan mockup: OCR untuk berkas hasil pindai | Ditunda — pekerjaan pemroses berkas, bukan pekerjaan yang membuktikan premis |

### Layar 6 — Approval Admin (`/admin/approval`)

Fase 2 seluruhnya. Tabel `review` tidak dibuat di MVP, dan alur MVP hanya `draft` → `published`. Approval berjenjang juga fase 2 pada roadmap mockup sendiri, jadi ini bukan penyimpangan. Yang dicatat di sini adalah bentuk yang harus dituju nanti, karena beberapa bagiannya menyentuh keputusan MVP:

| Elemen di mockup | Catatan |
| --- | --- |
| Antrean dengan prioritas, penanda "Tahap 1 dari 2", dan nomor pengajuan (#KM-2026-0847) | Bentuk Fase 2. Jumlah tahap mengikuti label dan klasifikasi |
| **Kartu "Pra-pemeriksaan AI"**: kesesuaian template, **"terindikasi memuat kredensial pada halaman 12"**, tumpang tindih 38% dengan dokumen lain | Lihat temuan 7. Pemindaian kredensial tetap masuk MVP, tetapi di tombol publish, bukan di antrean approval yang belum ada |
| Tiga keputusan: Setujui & Publikasikan, Minta Revisi, Tolak — semuanya terekam di audit log | Bentuk Fase 2; nilai keputusan dicatat di `docs/api-contract.md` |
| Alur: Diajukan → Pra-pemeriksaan otomatis → Review Admin Knowledge → Persetujuan Manajer Infrastruktur → Publikasi & indeks AI | Bentuk Fase 2 |
| **"Indeks AI hanya berjalan setelah approval"** | **Sudah jadi aturan kita** — hanya dokumen `published` yang di-embed. Di MVP ini berarti embed serentak saat publish |
| Pratinjau: Berkas asli / Hasil konversi / Perbandingan versi | Fase 2 |
| Catatan reviewer dengan @mention dan notifikasi email | Fase 2 — notifikasi tidak ada di MVP |
| Ketentuan publikasi: indeks ke AI, pengumuman ke kanal `#it-knowledge`, tandai bacaan wajib | Fase 2. Integrasi chat ada di fase 4 roadmap mockup |

### Layar 7 — Kategori & Label (`/admin/kategori-label`)

UI Fase 2; datanya dibangun penuh di MVP.

| Elemen di mockup | Status MVP |
| --- | --- |
| "42 kategori (3 tingkat)" dengan sub-kategori bernama (Infrastruktur punya Jaringan & Konektivitas, Server & Virtualisasi, Data Center & Fasilitas) | MVP satu tingkat, enam kategori. Kolom `parent_id` sudah ada supaya penambahan tingkat tidak butuh migrasi |
| 68 label dengan jumlah pemakaian (Kritikal 61, Runbook 148, SOP 231, …) | Label masuk lewat seed; jumlah pemakaian turunan |
| Seret untuk mengubah urutan dan hierarki, ekspor taksonomi | Fase 2 |
| Pil "Akses terbatas" pada kategori Keamanan Informasi | Lihat temuan 5 |
| **Kartu "Aturan Otomatis" (4 aturan aktif):** label Kritikal → dua tahap approval; **kategori Keamanan Informasi → hanya role Security & Admin**; tanpa pembaruan > 12 bulan → status Kedaluwarsa; label otomatis dari AI saat unggah | Lihat temuan 4 dan 5. Tidak ada mesin aturan di MVP |
| `.ai-extract` "Saran perapian taksonomi" (label mirip 87%, 6 label mati) | Fase 2 |

Konsekuensi untuk demo: kategori dan label tidak bisa diubah dari UI, hanya lewat seed.

### Layar 8 — Pengguna & RBAC (`/admin/pengguna`)

Layar ini sumber utama `docs/rbac-matrix.md`. Yang dicatat di sini hanya hal yang berubah atau baru terverifikasi:

| Elemen di mockup | Status MVP |
| --- | --- |
| Matriks izin delapan baris dengan tiga nilai: Diizinkan, **Terbatas**, Tidak | Sudah jadi dasar `docs/rbac-matrix.md`. Catatan kaki mockup mendefinisikan "Terbatas" sebagai "izin hanya berlaku pada kategori atau unit kerja yang ditugaskan" — dasar ADR-0009 |
| **Tidak ada kolom clearance per pengguna**; kolomnya Role, Unit Kerja, Cakupan Kategori, Status | Cocok dengan keputusan kita: klasifikasi tertinggi diturunkan dari `role`, dan `user.clearance` dihapus di T-003 |
| Daftar tujuh pengguna, termasuk **Fajar Nugroho (Viewer, Finance (mitra), cakupan "SOP & Proses (baca)", status Nonaktif)** | Lihat temuan 8. Fixture pengguna nonaktif ternyata sudah ada di mockup |
| **Maya Lestari: Admin Knowledge dengan cakupan "SOP, Onboarding" saja** | Membenarkan sel matriks "Admin Knowledge → Mengelola pengguna & role = Terbatas": admin pun bisa bercakupan sempit. Model kita mendukungnya lewat `category_scope`, dan MVP tidak perlu membuktikannya di UI |
| "Sinkronisasi otomatis dari Active Directory" dan tombol "Sinkron AD" | Ditunda — butuh akses direktori perusahaan. OIDC disiapkan di balik flag |
| Tombol "Undang Pengguna" | Fase 2 — tabel `invite` tidak dibuat |
| "Terakhir Aktif" per pengguna | Ditunda — satu kolom kecil, tetapi tidak dibutuhkan MVP |

### Delapan temuan dari layar 1-8

Temuan 1 sampai 3 sudah tercatat sejak revisi pertama bagian ini; 4 sampai 8 muncul setelah mockup dibaca langsung. Pil "Terbatas" di layar 2 tidak dihitung di sini karena sudah menjadi ADR-0011.

1. **Dua peringkat di layar 1 tidak punya sumber data di MVP.** "Topik yang sering dicari" butuh log pencarian yang baru terisi hari 4 — catatan mockup sendiri menyebutnya dihitung dari log 30 hari; "paling banyak dibaca" butuh `document_view` yang tidak dibuat. Keduanya ditunda, bukan diisi angka karangan.
2. **Kotak `.ai-answer` di layar 2 melanggar batasan "pencarian tanpa AI".** Diselesaikan dengan tombol "Lanjutkan di chat" yang sudah ada di mockup, bukan dengan menduplikasi jalur jawaban.
3. **Wizard empat langkah layar 5 melampaui MVP pada semua sisinya** — ekstraksi berkas, usulan metadata AI, deteksi duplikasi, dan approval semuanya Fase 2.
4. **"Terbatas" dan "Kedaluwarsa" dipakai sebagai status dokumen.** Layar 3 menampilkan keduanya di kolom Status, dan aturan otomatis layar 7 bahkan menetapkan "tanpa pembaruan > 12 bulan → status Kedaluwarsa". Keduanya bukan status: "Terbatas" adalah klasifikasi (dan menurut ADR-0011 tidak boleh muncul di daftar bagi yang tidak berhak), "Kedaluwarsa" adalah turunan `updated_at` dan `review_period_days` yang bisa berubah tanpa ada yang menyunting dokumen. Menyimpannya sebagai status berarti satu fakta punya dua sumber kebenaran, dan dokumen kedaluwarsa akan tampak tidak lagi `published`. Enum `DocStatus` tetap empat nilai; kedaluwarsa dihitung, dan klasifikasi tetap kolomnya sendiri.
5. **Layar 7 memperkenalkan dimensi izin ketiga.** Aturan "kategori Keamanan Informasi → hanya role Security & Admin" adalah izin yang menempel pada kategori, di luar model kita (role × `category_scope`, dengan klasifikasi tertinggi turunan role). Ditolak untuk MVP: hasil yang sama dicapai dengan memberi dokumen kategori itu klasifikasi `restricted` dan mengatur `category_scope`, tanpa mesin aturan dan tanpa jalur ketiga yang harus diuji kebocorannya. Kalau nanti dibutuhkan, itu ADR baru.
6. **Layar 3 punya "Favorit Saya" dan "Riwayat Baca".** Favorit butuh tabel yang sama sekali tidak ada di sebelas tabel T-003; riwayat baca butuh `document_view` yang dilarang. Keduanya ditunda, dan disebut di sini supaya tidak muncul diam-diam sebagai "kan cuma satu tabel kecil".
7. **Pemindaian konten sensitif muncul di antrean approval, bukan hanya saat unggah.** Layar 6 menampilkannya sebagai kartu pra-pemeriksaan dengan temuan konkret: kredensial di halaman 12 dan tumpang tindih 38%. Karena approval Fase 2, temuan 4 versi lama tetap berlaku — pemindaian pola sederhana berjalan di tombol publish. Deteksi tumpang tindih antar dokumen ditunda: ia butuh perbandingan embedding antar dokumen, bukan pencocokan pola.
8. **Fixture pengguna nonaktif sudah ada di mockup.** Layar 8 mencantumkan Fajar Nugroho sebagai Viewer dari Finance (mitra) dengan cakupan "SOP & Proses (baca)" dan status **Nonaktif** — lengkap dengan "Terakhir Aktif: 3 minggu lalu". Rencana seed sebelumnya keliru menjadikan Fajar viewer aktif lalu menambahkan Sari Puspita sebagai pengguna nonaktif. **Koreksi disetujui pemilik repo pada 5 September 2026 dan sudah diterapkan pada dokumentasi:** Fajar menjadi fixture `viewer` nonaktif dengan cakupan kategori seed **SOP & Proses Bisnis**; Sari dihapus dari tabel dan digantikan **Viewer Demo**, akun sintetis aktif untuk demo serta kontrol positif login. Tabel `docs/rbac-matrix.md` tetap enam pengguna (lima aktif, satu nonaktif); T-003, T-005, dan T-008 sudah diselaraskan. Pengujian nonaktif pada role yang boleh menulis memakai fixture sementara terisolasi, bukan pengguna seed tambahan. Implementasi seed dan test tetap pekerjaan task terkait, bukan perubahan kode di PR dokumentasi ini.

## Layar 9 — AI Assistant

Layar ini adalah halaman penuh di `/ai-assistant`, bukan sekadar kotak jawaban di halaman pencarian. Isinya:

| Elemen | Status MVP |
| --- | --- |
| Riwayat percakapan di sisi kiri | Masuk — dua tabel kecil, lihat ADR-0010. Endpoint `GET /api/ai/conversations`, potongan #1 |
| Pertanyaan lanjutan yang tidak berdiri sendiri | **Masuk** — sebelumnya terlewat, lihat ADR-0010 |
| Pil "Jawaban dibatasi hak akses Anda" | Masuk — teks statis, gratis, dan justru inti pesan produk |
| Panel "Sumber jawaban — 3 dokumen" | Masuk |
| Sitasi dengan lokasi presisi (bagian, halaman, versi) | Sebagian — `heading_path` memberi nama bagian dan versi. Nomor halaman butuh PDF, ditunda |
| Ruang lingkup jawaban: seluruh KB / kategori tertentu / dokumen yang dibuka | **Potongan #3** — tombol "Tanya AI tentang halaman ini" di layar 4 cukup mengisi pertanyaan awal. Menambahkannya nanti adalah satu field opsional pada `POST /api/ai/ask` |
| Umpan balik jempol atas/bawah | Masuk — satu kolom pada `ai_query` |
| "Dijawab dalam 2,4 detik · 3 dokumen dirujuk" | Masuk — kita sudah mengukur latensi. Angkanya dari pengukuran, bukan dari mockup |
| Chip pertanyaan lanjutan | Ditunda — hiasan, bukan kemampuan |
| "Ekspor jawaban" menjadi dokumen baru | Peluang tambahan, lihat bagian akhir |
| "Lampirkan dokumen" pada kotak chat | Ditunda bersama unggah multi-format |

Catatan mockup menegaskan empat hal yang semuanya sudah menjadi keputusan kita: sitasi wajib, penyaringan hak akses pada tahap pengambilan data, menjawab "tidak ditemukan" alih-alih mengarang, dan hanya dokumen yang telah disetujui yang masuk basis pengetahuan.

## Layar 10 — Dashboard

Konsol admin ditunda, dan itu tetap keputusan yang benar untuk enam hari. Tetapi seluruh isi layar ini adalah **data turunan**, dan data turunan hanya ada kalau dicatat sejak awal.

| Kartu di mockup | Sumber datanya | Konsekuensi untuk MVP |
| --- | --- | --- |
| Pencarian & pertanyaan AI | Log setiap kueri | Catat sejak hari 4 |
| Pencarian tanpa hasil (4,1%) | Log + penanda nol hasil | Catat penandanya |
| Kesenjangan knowledge (61 topik) | Kueri yang tidak terjawab | Turunan langsung dari jalur abstain |
| Pencarian terpopuler | Agregasi log | Gratis kalau log ada. Kartu ini sendiri menyebut dirinya "dasar rekomendasi topik di halaman muka" — sumber chip layar 1 |
| Status dokumen (donut) | Kolom `status` | Sudah ada |
| Dokumen kedaluwarsa (77) | `updated_at` + `review_period_days` | Sudah ada, turunan |
| Rata-rata waktu approval | Tabel approval | Ditunda bersama approval |

**Keputusan:** menulis satu baris log per pencarian dan per pertanyaan AI, berisi kueri, jumlah hasil, apakah menjawab atau abstain, dan role penanya. Biayanya satu `INSERT`. Tanpa itu, dashboard fase 2 harus menunggu lalu lintas nyata selama berminggu-minggu sebelum bisa menampilkan apa pun.

Catatan mockup menyebut kesenjangan knowledge sebagai pembeda strategis produk: pertanyaan yang sering diajukan namun tidak terjawab menjadi backlog penulisan yang terukur. Jalur abstain kita menghasilkan data itu sebagai efek samping.

## Layar 11 — Arsitektur & Roadmap

### Arsitektur tiga lapisan

| Komponen di mockup | Keputusan kita | Sesuai? |
| --- | --- | --- |
| Portal Web (Help Center) | Rute publik + katalog | Ya |
| Konsol Admin | Ditunda ke fase 2 | Ditunda, sadar |
| AI Assistant | Layar inti MVP | Ya |
| Mobile Web / PWA | **Terlewat** — lihat temuan 3 | Diperbaiki |
| API Gateway + SSO/OIDC | Route handler Next.js; OIDC di balik flag | Disederhanakan, sadar |
| Layanan Dokumen | Modul `lib/documents` | Ya |
| Mesin Approval (Workflow) | Ditunda ke fase 2 | Ditunda, sesuai roadmap mockup sendiri |
| Layanan RBAC | `lib/rbac/visible-documents.ts` | Ya |
| Layanan Pencarian | tsvector di Postgres | Disederhanakan |
| Layanan RAG | `POST /api/ai/ask` (`app/api/ai/ask/route.ts`) | Ya |
| Pemroses Berkas & OCR | Ditunda | Ditunda, sadar |
| Notifikasi | Ditunda | Ditunda |
| PostgreSQL (metadata & RBAC) | Sama persis | Ya |
| Object Storage (berkas asli) | Ditunda; MVP hanya `.md`/`.txt` | Ditunda |
| Vector DB (embedding) | pgvector di Postgres yang sama | **Disederhanakan sengaja** |
| Search Index | tsvector di Postgres yang sama | **Disederhanakan sengaja** |
| Audit Log | Tabel `audit_log` | Ya |

Baris Layanan RAG sebelumnya menulis `app/api/chat`, yang bertentangan dengan `POST /api/ai/ask` di `docs/api-contract.md`. Nama resminya `POST /api/ai/ask`; `chat()` adalah nama fungsi pada `lib/ai/provider.ts`, bukan rute. Alasan lengkapnya ada di bagian "Satu nama untuk endpoint AI" pada `docs/api-contract.md`.

Dua penyederhanaan yang perlu dijelaskan terbuka: mockup menggambar Vector DB dan Search Index sebagai komponen tersendiri, kita menaruh keduanya di dalam PostgreSQL lewat pgvector dan tsvector. Pada skala yang ditunjukkan mockup sendiri (1.284 dokumen) dan target kita (10.000 dokumen), komponen terpisah menambah dua sistem untuk dirawat tanpa menambah kemampuan. Batas wajarnya ada di ADR-0001, dan pemisahannya adalah perubahan lokal karena antarmuka retrieval sudah dipisah.

Callout keamanan mockup meminta opsi model AI lokal untuk dokumen berklasifikasi tinggi. Itu sudah dipenuhi ADR-0003 lewat `AI_PROVIDER` dengan dukungan Ollama dan vLLM. Ini salah satu titik terkuat rencana kita terhadap layar 11, dan layak disebut saat demo.

### Alur knowledge tujuh langkah

| Langkah | Status MVP |
| --- | --- |
| 1. Unggah berkas (PDF, Word, MD, Excel, PPT, pindaian) | Sebagian — hanya `.md` dan `.txt`, dan masuk lewat seed |
| 2. Ekstraksi & normalisasi, OCR | Tidak perlu untuk Markdown; OCR ditunda |
| 3. Pengayaan metadata oleh AI | Ditunda; kandidat tambahan opsional — panel `.ai-extract` layar 5 |
| 4. Review & approval satu atau dua tahap | Ditunda — hanya `draft` ke `published` |
| 5. Publikasi & pengindeksan | **Masuk** — embed serentak saat publish |
| 6. Konsumsi: pencarian, pembacaan, tanya jawab bersitasi | **Masuk, inti MVP** |
| 7. Pemeliharaan berkala | Sebagian — penanda kedaluwarsa masuk, pengingat otomatis ditunda |

MVP kita memotong lebar pada langkah 1 sampai 4 dan mengerjakan langkah 5 dan 6 secara utuh. Layar 5 adalah tampilan langkah 1 sampai 4, dan itu sebabnya layar itu yang paling jauh berbeda dari mockup.

### Empat fase mockup versus sprint kita

Mockup mengusulkan 5-6 bulan: fase 1 fondasi, fase 2 tata kelola, fase 3 kecerdasan termasuk RAG, fase 4 perluasan.

Sprint enam hari kita **bukan fase 1 yang dipercepat.** Ia adalah irisan vertikal tipis yang memotong fase 1 dan fase 3 sekaligus, dan sengaja melewati fase 2.

Alasannya perlu dinyatakan, karena ini keputusan yang paling mudah disalahpahami: fase 1 hampir tidak punya risiko teknis. CRUD dokumen, kategori, dan pencarian kata kunci adalah pekerjaan yang hasilnya bisa diprediksi. Seluruh risiko proyek ini menumpuk di fase 3 — apakah jawaban AI berbahasa Indonesia cukup kredibel, dan apakah izin tetap rapat ketika dokumen mengalir melalui LLM. Mengerjakan enam bulan fondasi lebih dulu berarti menemukan jawaban pertanyaan itu di bulan kelima.

Irisan vertikal menjawabnya di hari kerja keenam. Kalau jawabannya buruk, yang hilang enam hari, bukan lima bulan.

Konsekuensinya jujur: MVP kita lebih dangkal dari fase 1 pada sisi ingest, dan lebih dalam dari fase 3 pada sisi AI. Perbandingan yang adil bukan "berapa persen fase 1 selesai", melainkan "apakah premis produk terbukti".

### Manfaat dan aspek keamanan

Empat manfaat yang diklaim mockup semuanya bergantung pada satu hal yang sama: jawaban harus bisa dipercaya. Tidak ada yang perlu diubah dari rencana.

Empat aspek keamanan, dan status kita:

| Aspek | Status |
| --- | --- |
| Klasifikasi berjenjang: Publik, Internal, Terbatas, Rahasia | Cocok persis dengan model kita |
| Audit trail: siapa mengunggah, menyetujui, membaca, mengunduh | Masuk untuk pembacaan dokumen terbatas. **Dikeluarkan dari daftar potong** — biayanya satu `INSERT`, dan ini persyaratan kepatuhan, bukan fitur |
| Pemindaian konten sensitif saat unggah | **Terlewat** — lihat temuan 4 di bagian berikut, dan temuan 7 layar 1-8 |
| Data tetap di lingkungan perusahaan, opsi model lokal | Sudah dipenuhi ADR-0003 |

## Lima temuan yang mengubah rencana

Diurutkan menurut selisih biaya antara mengerjakannya hari ini dan mengerjakannya nanti. Temuan ini lahir dari layar 9-11; temuan dari layar 1-8 ada di bagian "Delapan temuan dari layar 1-8" di atas.

### 1. Izin dua dimensi — cakupan kategori

Matriks izin layar 8 punya tiga nilai, bukan dua, dan "terbatas" didefinisikan sebagai "izin hanya berlaku pada kategori atau unit kerja yang ditugaskan". Skema kita hanya punya `user.unit` bernilai tunggal.

**Biaya sekarang:** satu kolom array pada hari 1, sekitar sepuluh menit. **Biaya nanti:** menulis ulang `visibleDocumentsFilter`, test kebocoran, dan seluruh data seed setelah semuanya sudah hijau. Keputusan di ADR-0009.

### 2. Percakapan berlanjut

Layar 9 memperlihatkan pertanyaan lanjutan yang tidak bisa dipahami sendirianya. Rencana kita menyebut "RAG satu langkah" dan menaruh halaman chat di urutan potong nomor dua. Keduanya salah kalibrasi.

**Biaya sekarang:** dua tabel kecil dan satu langkah penulisan ulang pertanyaan. **Biaya nanti:** pertanyaan lanjutan gagal di depan penonton demo. Keputusan di ADR-0010.

### 3. Layak pakai di ponsel

Lapisan pengguna layar 11 mencantumkan Mobile Web / PWA. Rencana UI kita seluruhnya desktop dengan sidebar 240px tetap.

**Biaya sekarang:** menutup sidebar di bawah titik potong saat `AppShell` dibuat pada hari 1, sekitar dua puluh baris. **Biaya nanti:** menyisipkan perilaku responsif ke enam halaman yang sudah jadi. Yang **tidak** dikerjakan: manifest, service worker, mode luring. PWA sesungguhnya adalah pekerjaan tersendiri dan tidak dibutuhkan untuk membuktikan apa pun minggu ini. Targetnya cukup "bisa dibuka dan dibaca di ponsel tanpa memalukan".

### 4. Pemindaian konten sensitif saat publikasi

Layar 11 mencantumkan ini sebagai persyaratan kepatuhan: kredensial dan data pribadi ditandai sebelum publikasi. Layar 6 memperlihatkan bentuk konkretnya — kartu pra-pemeriksaan yang menandai "terindikasi memuat kredensial pada halaman 12".

**Biaya sekarang:** pemeriksaan pola sederhana pada tombol publish — pola kata sandi, bentuk kunci API, dan angka enam belas digit menyerupai NIK — lalu peringatan yang bisa dilewati dengan sadar. Sekitar empat puluh baris, tanpa ketergantungan baru.

Alasannya bukan kelengkapan fitur. Seluruh premis produk ini adalah kredibilitas dalam menangani dokumen sensitif, dan pemeriksaan ini juga melindungi kita dari aturan kita sendiri: tidak ada data Telkom asli yang boleh masuk. Ini pengaman yang berpihak pada kita.

### 5. Pencatatan kueri sejak hari pertama

Lihat layar 10 di atas. Satu `INSERT` per kueri sekarang, atau dashboard yang tidak punya data untuk ditampilkan nanti. Layar 1 menambah alasan kedua: chip topik populer dan peringkat "paling banyak dibaca" juga berdiri di atas log yang sama, dan kartu "Pencarian Terpopuler" layar 10 menyebut dirinya sendiri sebagai sumber chip itu.

## Perbedaan yang dipertahankan, dan alasannya

Delapan perbedaan berikut sengaja tidak diubah. Semuanya perlu dijawab kalau ditanya saat demo, jadi jawabannya ditulis di sini.

| Perbedaan | Alasan |
| --- | --- |
| 5 role mockup menjadi 4 role kita | Super Admin dan Admin Knowledge hanya berbeda pada satu sel matriks, yaitu mengelola pengguna. Konsol pengguna belum ada di MVP, jadi perbedaannya belum bisa diamati. Memisahkannya nanti berarti menambah satu nilai role dan satu pemeriksaan |
| Role kustom | Butuh editor izin. Ditunda ke fase 2 bersama konsol admin |
| Vector DB dan Search Index terpisah | Satu Postgres cukup pada skala mockup dan target kita. Batas wajar ada di ADR-0001 |
| SSO / sinkronisasi Active Directory | Butuh akses ke direktori perusahaan yang belum kita punya. OIDC sudah disiapkan di balik flag |
| Approval berjenjang | Fase 2 pada roadmap mockup sendiri. MVP hanya `draft` ke `published` |
| Unggah multi-format dan OCR | Waktunya habis di penanganan format, bukan di hal yang membuktikan premis |
| Kategori tiga tingkat | MVP satu tingkat. Kolom induk sudah ada di skema supaya tidak perlu migrasi |
| Mesin "Aturan Otomatis" layar 7 | Izin per kategori adalah dimensi ketiga di luar model kita. Hasil yang sama dicapai lewat klasifikasi dan `category_scope`. Lihat temuan 5 |

## Dua kandidat tambahan, hanya kalau daftar wajib sudah selesai

Keduanya berbiaya kecil dengan kesan besar, dan keduanya ada di mockup. Statusnya tambahan, bukan bagian MVP, dan tidak boleh menyingkirkan pekerjaan pada daftar wajib.

1. **Usulan metadata oleh AI saat publikasi.** Langkah 3 alur knowledge dan panel `.ai-extract` di layar 5. Satu prompt yang mengusulkan ringkasan, kategori, dan label. Memakai ulang `chat()` yang sudah ada.
2. **Ekspor jawaban menjadi draf dokumen.** Tombolnya ada di layar 9. Menutup lingkaran: pertanyaan yang tidak terjawab menjadi dokumen baru, dan itu persis mekanisme yang membuat basis pengetahuan tumbuh sendiri.

## Yang dikatakan terbuka saat demo

Menyebut batas sendiri lebih dipercaya daripada menunggu ditemukan.

1. Seluruh dokumen sintetis dan fiktif. Tidak ada satu pun dokumen Telkom asli.
2. Empat role, bukan lima ditambah role kustom. Alasannya satu sel matriks.
3. Hanya Markdown dan teks. PDF dan Word adalah pekerjaan pemroses berkas, bukan pekerjaan yang membuktikan premis.
4. Tanpa SSO. Menunggu akses direktori.
5. Tanpa approval berjenjang. Fase 2 pada roadmap usulan itu sendiri.
6. Angka kualitas yang ditampilkan berasal dari `pnpm eval` atas sepuluh pertanyaan, bukan dari klaim. Angka mockup seperti "96% akurasi AI" adalah pengisi tempat dan tidak akan ditampilkan.
7. Layar 2 kita lebih sederhana daripada mockup: tanpa kotak jawaban AI, dan tanpa pil "Terbatas — akses via permintaan". Yang kedua adalah keputusan keamanan, bukan pekerjaan yang belum selesai — sebut ADR-0011 kalau ditanya.
8. Halaman muka tanpa chip topik populer dan tanpa peringkat "paling banyak dibaca". Keduanya butuh data pemakaian yang belum ada, dan kami memilih tidak mengarang angka.
9. Tanpa favorit dan riwayat baca. Keduanya butuh tabel yang sengaja tidak dibuat di MVP.
