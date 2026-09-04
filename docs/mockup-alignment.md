# Kesesuaian Rencana dengan Mockup

Diperiksa 4 September 2026, setelah layar 9-11 dibaca utuh. Layar 1-8 ditambahkan 5 September 2026; bagian "Layar 1-8" menjelaskan dasar pemeriksaannya. Berkas ini adalah matriks keterlacakan antara mockup dan rencana MVP, sekaligus daftar koreksi yang keluar dari pemeriksaan itu.

## Koreksi inventaris

| Yang tertulis sebelumnya | Yang benar |
| --- | --- |
| Mockup punya 8 layar | **11 layar.** Layar 9 AI Assistant, 10 Dashboard, 11 Arsitektur & Roadmap |
| 6 role | **5 role bawaan + kemampuan membuat role kustom.** Judul layar 8 menulis "6 role", tetapi kartu yang ada lima |
| Izin satu dimensi (role + clearance) | **Dua dimensi:** role menentukan *apa*, cakupan kategori menentukan *di mana* |

`docs/ui-inventory.md` sudah memuat kesebelas layar beserta rutenya, jadi berkas ini tidak lagi perlu menambalnya. Yang tinggal di sini adalah alasan dan keputusannya, bukan daftar layarnya.

Angka pada mockup tidak konsisten satu sama lain (jumlah pengguna per role berjumlah 312, header menulis 318 aktif). Ini menegaskan aturan yang sudah ada: seluruh angka pada mockup adalah pengisi tempat dan tidak boleh ditampilkan sebagai fakta.

## Layar 1-8

Pemeriksaan ini menyusul, dan menutup lubang yang disebut ADR-0011: sampai 4 September berkas ini hanya mencakup layar 9-11, sehingga pertentangan paling serius — pil "Terbatas - akses via permintaan" di layar 2 — tidak pernah tercatat di mana pun.

Dasar pemeriksaan: `docs/ui-inventory.md` (rute, status, dan istilah UI per layar), ADR-0011 (layar 2), `docs/scope-mvp.md` (urutan potong dan daftar "tidak masuk MVP"), `docs/rbac-matrix.md` (layar 8), `docs/api-contract.md`, dan T-003 (daftar tabel yang tidak dibuat).

Baris bertanda **[belum diverifikasi]** adalah elemen yang belum dibaca ulang langsung dari `intradocs-mockup_1.html` pada pemeriksaan ini. `AGENTS.md` melarang membuka berkas itu utuh dan meminta pembacaan per rentang baris; verifikasinya menyusul saat layar itu benar-benar dibangun. Keputusannya tetap berlaku — yang belum pasti adalah kelengkapan daftar elemennya, bukan arah keputusannya.

### Layar 1 — Halaman Muka (`/`)

| Elemen | Status MVP |
| --- | --- |
| Pencarian besar di tengah halaman | Masuk |
| Kartu kategori | Masuk — enam kategori, satu tingkat |
| **Chip topik populer** | **Ditunda.** Sumbernya agregasi log pencarian, dan log baru mulai terisi hari 4. Chip yang diisi manual berarti menampilkan topik karangan, dan itu melanggar aturan "angka dan isi di aplikasi selalu dihitung dari basis data" |
| Penghitung "1.284 dokumen · 318 pengguna" | **Tidak ditampilkan dengan angka mockup.** Kalau ditampilkan, dihitung dari basis data. Angka mockup pengisi tempat dan saling bertentangan |
| Dokumen terbaru | Masuk — turunan `updated_at` |
| Dokumen paling banyak dibaca | **Ditunda** — butuh `document_view`, tabel yang sengaja tidak dibuat di MVP (lihat daftar larangan T-003) |
| Tautan cepat ke AI Assistant | Masuk |

Dua peringkat di layar ini ("topik populer" dan "paling banyak dibaca") adalah data turunan tanpa sumber di MVP. Ini temuan 5 yang muncul lagi di tempat lain: pencatatan log yang dimulai hari 4 baru berguna beberapa hari kemudian, dan peringkat dari korpus 20-25 dokumen sintetis tidak berarti apa-apa. Halaman muka MVP karena itu lebih sepi daripada mockup, dan itu keputusan.

### Layar 2 — Hasil Pencarian (`/cari`)

Satu-satunya layar yang isinya bertentangan **langsung** dengan aturan kita, bukan sekadar lebih kaya. Keduanya kini punya keputusan tertulis.

| Elemen | Status MVP |
| --- | --- |
| Daftar hasil dengan snippet tersorot | Masuk |
| Pil "Terbatas - akses via permintaan" pada dokumen di luar izin | **Tidak dibangun** — ADR-0011. Dokumen di luar izin tidak muncul sama sekali, termasuk judulnya |
| Jumlah hasil ("41 hasil") | Masuk, tetapi **dihitung setelah filter izin** (ADR-0011). Angka 41 sendiri pengisi tempat |
| Kotak `.ai-answer` di atas daftar hasil | **Tidak dibangun di halaman pencarian** — lihat alasan di bawah |
| Filter di sisi kiri | Sebagian — kategori saja; filter lain adalah potongan #4 |
| Tombol menuju AI Assistant dengan kueri terbawa | Masuk — ini penggantinya kotak `.ai-answer` |

**Kenapa kotak `.ai-answer` tidak dibangun di layar 2.** `docs/api-contract.md` menetapkan pencarian tidak boleh memanggil provider AI, supaya pencarian tetap hidup ketika AI mati atau kuota habis. Menempelkan jawaban AI di halaman hasil berarti dua jalur jawaban dengan dua tempat sitasi, dua tempat jalur abstain, dan dua tempat yang harus diuji kebocorannya — pada minggu yang tujuan utamanya membuktikan satu jalur AI bekerja rapat. Jawaban AI karena itu hanya ada di layar 9, dan layar 2 menautkan ke sana dengan kueri terbawa. Biaya menyatukannya nanti kecil: komponennya sama.

### Layar 3 — Katalog Dokumen (`/katalog`) **[belum diverifikasi]**

| Elemen | Status MVP |
| --- | --- |
| Daftar/grid dokumen dengan badge klasifikasi dan status | Masuk |
| Filter kategori | Masuk |
| Filter label, klasifikasi, rentang tanggal | Potongan #4 — dilepas kalau waktu habis |
| Penanda kedaluwarsa dan "Terverifikasi" | Masuk — turunan `updated_at` + `review_period_days` |
| Pengurutan "paling banyak dibaca" | Ditunda — `document_view` tidak dibuat |
| Jumlah dokumen per kategori | Masuk, dihitung setelah filter izin (ADR-0011) |

### Layar 4 — Baca Dokumen (`/{kategori}/{slug}`)

| Elemen | Status MVP |
| --- | --- |
| Isi Markdown terender, dibersihkan `rehype-sanitize` | Masuk |
| `DocNav` daftar heading | Masuk — tampilan satu tingkat (potongan #5), tetapi `chunk.heading_path` tetap penuh |
| Sitasi "Bagian 2.1.2" dan tautan balik dari jawaban AI | Masuk — bergantung pada `heading_path`, ada di daftar "tidak boleh dipotong" |
| Badge "Terverifikasi" dan peringatan tinjau ulang | Masuk |
| Tombol "Tanya AI tentang halaman ini" | Masuk sebagai navigasi ke `/ai-assistant` dengan pertanyaan awal terisi. Parameter ruang lingkup jawaban adalah potongan #3 |
| Riwayat versi di UI | Fase 2 — tabel `document_version` sudah ada sejak MVP |
| Unduh berkas asli | Fase 2 — tanpa object storage tidak ada berkas asli |
| Pencatatan `audit_log` saat membaca `restricted`/`secret` | Masuk — persyaratan kepatuhan |

### Layar 5 — Unggah Knowledge (`/unggah`)

| Elemen | Status MVP |
| --- | --- |
| **Wizard empat langkah** | **Tidak dibangun.** MVP memakai satu form sederhana, dan form itu sendiri potongan #2 — dokumen MVP masuk lewat `pnpm seed`. Wizard empat langkah baru masuk akal setelah ekstraksi berkas dan approval ada, dan keduanya Fase 2 |
| Unggah PDF, Word, Excel, PPT, hasil pindaian | Fase 2 — lihat bagian "Upload & ingest" di `docs/api-contract.md`, seluruhnya bertanda Fase 2 |
| Panel `.ai-extract` — usulan ringkasan, kategori, label oleh AI | Tambahan opsional, hanya kalau daftar wajib sudah selesai |
| Pemindaian konten sensitif sebelum publikasi | Masuk — temuan 4 di bawah |
| Pratinjau Markdown | Masuk kalau form dibangun |

Urutan langkah pada wizard tetap dicatat di sini karena ia menggambarkan alur knowledge tujuh langkah versi ringkas, dan alur itu yang jadi acuan Fase 2 — bukan karena MVP akan membangunnya.

### Layar 6 — Approval Admin (`/admin/approval`) **[belum diverifikasi]**

Fase 2 seluruhnya. Tabel `review` tidak dibuat di MVP, dan alur MVP hanya `draft` → `published`. Nilai `in_review` dan `archived` tetap ada di enum `DocStatus` supaya approval Fase 2 tidak butuh migrasi. Approval berjenjang ada di fase 2 roadmap mockup sendiri, jadi ini bukan penyimpangan.

### Layar 7 — Kategori & Label (`/admin/kategori-label`) **[belum diverifikasi]**

UI Fase 2. Datanya sudah dibangun penuh di MVP: `category` satu tingkat dengan kolom `parent_id` yang sudah ada, `label` dengan relasi banyak-ke-banyak lewat `document_label`. Enam kategori dan seluruh label masuk lewat seed. Konsekuensinya untuk demo: kategori dan label tidak bisa diubah dari UI, hanya lewat seed.

### Layar 8 — Pengguna & RBAC (`/admin/pengguna`)

Layar ini sudah menjadi sumber utama `docs/rbac-matrix.md` pada pemeriksaan pertama, jadi yang dicatat di sini hanya perubahan sesudahnya:

- **Tidak ada kolom clearance per pengguna.** Klasifikasi tertinggi diturunkan dari `role`; dua sumber kebenaran untuk satu aturan visibilitas dilarang ADR-0004.
- **Cakupan kategori** adalah dimensi kedua izin, dan itulah yang membuat contoh mockup sendiri ("Reviewer Keamanan tidak dapat menyetujui dokumen Aplikasi Internal") bisa benar (ADR-0009).
- **Enam pengguna seed, bukan lima**, termasuk satu pengguna nonaktif sebagai prasyarat `tests/rbac/inactive-user.spec.ts`.
- UI-nya ditunda, modelnya tidak. Layar ini yang menentukan bentuk tabel `user`.

### Tiga temuan baru dari layar 1-8

Yang keempat — pil "Terbatas" di layar 2 — sudah menjadi ADR-0011 dan tidak diulang di sini.

1. **Dua peringkat di layar 1 tidak punya sumber data di MVP.** "Topik populer" butuh log pencarian yang baru terisi hari 4; "paling banyak dibaca" butuh `document_view` yang tidak dibuat. Keduanya ditunda, bukan diisi angka karangan. Ini temuan 5 yang berulang.
2. **Kotak `.ai-answer` di layar 2 melanggar batasan "pencarian tanpa AI".** Diselesaikan dengan menautkan ke layar 9, bukan dengan menduplikasi jalur jawaban.
3. **Wizard empat langkah layar 5 melampaui MVP pada semua sisinya** — ekstraksi berkas, usulan metadata AI, dan approval semuanya Fase 2. Keputusan lama (form sederhana, dokumen lewat seed) tetap, tetapi sekarang tercatat sebagai perbedaan yang sadar, bukan kelalaian.

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
| Pencarian terpopuler | Agregasi log | Gratis kalau log ada |
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
| Pemindaian konten sensitif saat unggah | **Terlewat** — lihat temuan 4 |
| Data tetap di lingkungan perusahaan, opsi model lokal | Sudah dipenuhi ADR-0003 |

## Lima temuan yang mengubah rencana

Diurutkan menurut selisih biaya antara mengerjakannya hari ini dan mengerjakannya nanti. Temuan ini lahir dari layar 9-11; temuan dari layar 1-8 ada di bagian "Tiga temuan baru dari layar 1-8" di atas.

### 1. Izin dua dimensi — cakupan kategori

Matriks izin layar 8 punya tiga nilai, bukan dua, dan "terbatas" didefinisikan sebagai "izin hanya berlaku pada kategori atau unit kerja yang ditugaskan". Skema kita hanya punya `user.unit` bernilai tunggal.

**Biaya sekarang:** satu kolom array pada hari 1, sekitar sepuluh menit. **Biaya nanti:** menulis ulang `visibleDocumentsFilter`, test kebocoran, dan seluruh data seed setelah semuanya sudah hijau. Keputusan di ADR-0009.

### 2. Percakapan berlanjut

Layar 9 memperlihatkan pertanyaan lanjutan yang tidak bisa dipahami sendirian. Rencana kita menyebut "RAG satu langkah" dan menaruh halaman chat di urutan potong nomor dua. Keduanya salah kalibrasi.

**Biaya sekarang:** dua tabel kecil dan satu langkah penulisan ulang pertanyaan. **Biaya nanti:** pertanyaan lanjutan gagal di depan penonton demo. Keputusan di ADR-0010.

### 3. Layak pakai di ponsel

Lapisan pengguna layar 11 mencantumkan Mobile Web / PWA. Rencana UI kita seluruhnya desktop dengan sidebar 240px tetap.

**Biaya sekarang:** menutup sidebar di bawah titik potong saat `AppShell` dibuat pada hari 1, sekitar dua puluh baris. **Biaya nanti:** menyisipkan perilaku responsif ke enam halaman yang sudah jadi. Yang **tidak** dikerjakan: manifest, service worker, mode luring. PWA sesungguhnya adalah pekerjaan tersendiri dan tidak dibutuhkan untuk membuktikan apa pun minggu ini. Targetnya cukup "bisa dibuka dan dibaca di ponsel tanpa memalukan".

### 4. Pemindaian konten sensitif saat publikasi

Layar 11 mencantumkan ini sebagai persyaratan kepatuhan: kredensial dan data pribadi ditandai sebelum publikasi.

**Biaya sekarang:** pemeriksaan pola sederhana pada tombol publish — pola kata sandi, bentuk kunci API, dan angka enam belas digit menyerupai NIK — lalu peringatan yang bisa dilewati dengan sadar. Sekitar empat puluh baris, tanpa ketergantungan baru.

Alasannya bukan kelengkapan fitur. Seluruh premis produk ini adalah kredibilitas dalam menangani dokumen sensitif, dan pemeriksaan ini juga melindungi kita dari aturan kita sendiri: tidak ada data Telkom asli yang boleh masuk. Ini pengaman yang berpihak pada kita.

### 5. Pencatatan kueri sejak hari pertama

Lihat layar 10 di atas. Satu `INSERT` per kueri sekarang, atau dashboard yang tidak punya data untuk ditampilkan nanti. Layar 1 menambah alasan kedua: chip topik populer dan peringkat "paling banyak dibaca" juga berdiri di atas log yang sama.

## Perbedaan yang dipertahankan, dan alasannya

Tujuh perbedaan berikut sengaja tidak diubah. Semuanya perlu dijawab kalau ditanya saat demo, jadi jawabannya ditulis di sini.

| Perbedaan | Alasan |
| --- | --- |
| 5 role mockup menjadi 4 role kita | Super Admin dan Admin Knowledge hanya berbeda pada satu sel matriks, yaitu mengelola pengguna. Konsol pengguna belum ada di MVP, jadi perbedaannya belum bisa diamati. Memisahkannya nanti berarti menambah satu nilai role dan satu pemeriksaan |
| Role kustom | Butuh editor izin. Ditunda ke fase 2 bersama konsol admin |
| Vector DB dan Search Index terpisah | Satu Postgres cukup pada skala mockup dan target kita. Batas wajar ada di ADR-0001 |
| SSO / sinkronisasi Active Directory | Butuh akses ke direktori perusahaan yang belum kita punya. OIDC sudah disiapkan di balik flag |
| Approval berjenjang | Fase 2 pada roadmap mockup sendiri. MVP hanya `draft` ke `published` |
| Unggah multi-format dan OCR | Waktunya habis di penanganan format, bukan di hal yang membuktikan premis |
| Kategori tiga tingkat | MVP satu tingkat. Kolom induk sudah ada di skema supaya tidak perlu migrasi |

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
7. Layar 2 kita lebih sederhana daripada mockup: tanpa kotak jawaban AI, dan tanpa pil "Terbatas - akses via permintaan". Yang kedua adalah keputusan keamanan, bukan pekerjaan yang belum selesai — sebut ADR-0011 kalau ditanya.
8. Halaman muka tanpa chip topik populer dan tanpa peringkat "paling banyak dibaca". Keduanya butuh data pemakaian yang belum ada, dan kami memilih tidak mengarang angka.
