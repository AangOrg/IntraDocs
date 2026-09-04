# Scope MVP — Mengikat

Berkas ini menang atas `docs/prd.md`, `docs/architecture.md`, dan `docs/context-pack.md` bila terjadi pertentangan. Alasan pemotongan ada di ADR-0007. Penyesuaian setelah pemeriksaan layar 9-11 mockup ada di `docs/mockup-alignment.md`.

Target: MVP bisa diklik pada Jumat, 11 September 2026, dari URL produksi.

## Definisi selesai

MVP dinyatakan tercapai bila seluruh butir berikut benar, diperiksa dari URL produksi dan bukan dari localhost.

1. Tiga role berbeda masuk dan melihat katalog yang berbeda
2. Dokumen di luar izin menghasilkan 404, bukan 403
3. Reviewer dengan cakupan kategori sempit tidak melihat dokumen terbatas dari kategori lain
4. Pencarian identifier persis dan pencarian parafrase, keduanya tepat
5. Jawaban AI muncul dengan sitasi yang bisa diklik menuju bagian dokumen
6. Pertanyaan lanjutan yang tidak berdiri sendiri tetap dijawab benar
7. Pertanyaan di luar korpus dijawab "tidak ditemukan", bukan dikarang
8. Viewer tidak bisa memancing isi dokumen terbatas lewat jalur AI
9. `pnpm test` hijau termasuk seluruh `tests/rbac/`
10. `pnpm eval` mencetak angka, dan kebocoran bernilai nol
11. Halaman bisa dibuka dan dibaca di ponsel
12. Orang lain bisa membukanya lewat URL tanpa dibantu

## Masuk MVP

### Identitas dan izin

- Akun lokal, lima pengguna hasil seed, tanpa alur invite
- Empat role: `viewer`, `contributor`, `reviewer`, `admin`
- Empat klasifikasi: `public`, `internal`, `restricted`, `secret`
- Cakupan kategori per pengguna, `NULL` berarti semua kategori (ADR-0009)
- Penanda aktif atau nonaktif pada pengguna; yang nonaktif tidak bisa masuk
- Satu filter izin di SQL, dipakai katalog, kedua jalur pencarian, dan RAG
- Test kebocoran ditulis sebelum jalur AI ada

### Dokumen

- Format `.md` dan `.txt` saja
- Status `draft` dan `published`; tanpa approval berjenjang
- Satu kategori per dokumen, banyak label
- Kategori satu tingkat, tetapi kolom induk sudah ada di skema
- Viewer dokumen dengan daftar heading dan sitasi yang bisa dituju
- Form buat dan sunting sederhana
- Pemeriksaan pola konten sensitif saat publikasi, dengan peringatan yang bisa dilewati sadar
- Penanda kedaluwarsa sebagai nilai turunan dari `updated_at` dan `review_period_days`
- Penulisan `audit_log` untuk pembacaan dokumen `restricted` dan `secret`

### Pencarian dan AI

- Hybrid: BM25 lewat tsvector dan vektor lewat pgvector, digabung RRF
- Halaman hasil pencarian dengan filter kategori
- Halaman AI Assistant tersendiri
- Percakapan berlanjut dengan penulisan ulang pertanyaan lanjutan (ADR-0010)
- Ruang lingkup jawaban: seluruh basis pengetahuan, satu kategori, atau satu dokumen
- Sitasi wajib pada setiap pernyataan, menyebut dokumen, bagian, dan versi
- Jalur abstain bila tidak ada dasar yang cukup
- Umpan balik jempol atas dan bawah
- Satu baris log per pencarian dan per pertanyaan AI, berisi kueri, jumlah hasil, menjawab atau abstain, dan role penanya
- Embedding dihitung serentak saat publikasi, tanpa job queue
- Hanya dokumen `published` yang diindeks

### Antarmuka

- Design token diambil dari variabel CSS mockup, tanpa warna keras di kode
- `AppShell` dengan sidebar yang menutup di bawah titik potong
- Komponen bersama dibuat lebih dulu sebelum halaman
- Setiap daftar punya empat keadaan: memuat, kosong, gagal, tidak berizin

### Kualitas

- Eval sepuluh pertanyaan termasuk pertanyaan di luar korpus
- Dua puluh sampai dua puluh lima dokumen sintetis, termasuk near-duplicate dan satu dokumen kedaluwarsa
- `typecheck`, `lint`, `test`, `build` hijau di CI sebelum merge

## Tidak masuk MVP

Konversi PDF dan DOCX · OCR · job queue · object storage · alur invite · approval berjenjang · UI riwayat versi · permintaan akses · kategori hierarkis · role kustom · konsol admin dan dashboard · pengingat tinjau ulang otomatis · sinkronisasi Active Directory · SSO · notifikasi · pengayaan metadata otomatis · reranker · Docker · rate limit · Playwright · PWA sesungguhnya dengan service worker.

## Tambahan opsional

Hanya dikerjakan bila hari 1 sampai 4 selesai tepat waktu. Tidak boleh menyingkirkan pekerjaan pada daftar wajib.

1. Usulan ringkasan, kategori, dan label oleh AI saat publikasi
2. Ekspor jawaban AI menjadi draf dokumen baru

## Urutan potong bila waktu habis

Dipotong dari atas.

1. Penyimpanan riwayat percakapan — chat tetap jalan, riwayatnya tidak tersimpan
2. Form buat dan sunting di UI — dokumen masuk lewat seed
3. Selektor ruang lingkup jawaban
4. Filter selain kategori
5. Daftar heading hierarkis menjadi datar

## Tidak boleh dipotong

- Filter izin di SQL dan test kebocorannya
- Sitasi pada setiap jawaban
- Jalur abstain
- Penulisan `audit_log` untuk dokumen terbatas — ini persyaratan kepatuhan, bukan fitur
- Log kueri — biayanya satu `INSERT`, dan tanpanya dashboard fase 2 tidak punya data
- Dokumen sintetis
- Deploy yang bisa diakses orang lain

## Aturan yang tidak boleh dilanggar

Seluruh konten dokumen bersifat sintetis dan fiktif. Tidak ada dokumen Telkom asli yang boleh masuk ke lingkungan mana pun yang terhubung API publik. Ini satu-satunya asumsi yang mahal kalau salah.
