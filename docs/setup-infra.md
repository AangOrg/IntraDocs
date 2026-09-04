# Setup infrastruktur — Neon, Vercel, GitHub

Sekali kerja, sekitar 30–45 menit. Dikerjakan sebelum T-001 selesai.

## Urutan tidak boleh dibalik

1. **Neon dulu** — Vercel butuh connection string-nya.
2. **Vercel kedua** — butuh deploy pertama supaya CI dan preview hidup.
3. **Proteksi GitHub terakhir** — GitHub hanya bisa mewajibkan status check yang **sudah pernah berjalan minimal sekali**. Kalau dipasang sebelum CI pertama jalan, namanya tidak muncul di daftar.

---

# 1. Neon

## Langkah

1. Daftar di neon.tech, buat project baru.
2. **Region: Singapore (`ap-southeast-1`).** Ini keputusan performa terbesar di halaman ini. Region Neon harus sama dengan region fungsi Vercel — kalau beda benua, setiap kueri menyeberang samudra dan target p95 < 500 ms mustahil tercapai. Salah pilih di sini tidak bisa diperbaiki tanpa membuat project baru.
3. Versi Postgres 16 atau lebih baru.
4. Buka SQL Editor, jalankan:
   ```sql
   CREATE EXTENSION IF NOT EXISTS vector;
   SELECT extversion FROM pg_extension WHERE extname = 'vector';
   ```
   Baris kedua harus mengembalikan nomor versi. Kalau kosong, ekstensinya belum aktif dan T-003 akan gagal.

## Dua connection string, bukan satu

Neon memberi dua alamat. Bedanya penting dan sering terlewat:

| Variabel | Alamat | Dipakai untuk |
|---|---|---|
| `DATABASE_URL` | yang mengandung `-pooler` | Aplikasi saat berjalan |
| `DATABASE_URL_UNPOOLED` | tanpa `-pooler` | Migrasi dan seed |

Alamat pooled melewati PgBouncer dalam mode transaksi. Bagus untuk banyak koneksi pendek dari fungsi serverless, tapi **migrasi bisa gagal atau berperilaku aneh di atasnya** karena DDL dan fitur tingkat sesi tidak sepenuhnya didukung. Pakai yang direct untuk `drizzle-kit` dan skrip seed.

Keduanya harus diakhiri `?sslmode=require`.

## Buat branch database untuk kerja lokal

Neon punya fitur branching. Buat satu branch bernama `dev` dan pakai itu di `.env.local`.

Alasannya praktis: `pnpm seed` menghapus dan mengisi ulang data. Kalau lokal dan produksi menunjuk basis data yang sama, suatu saat seseorang akan menyeed ulang satu jam sebelum demo dan menghapus semuanya. Branch memisahkan itu tanpa biaya tambahan.

## Yang harus diketahui tentang paket gratis

- Basis data **tidur setelah beberapa menit menganggur**. Kueri pertama setelah tidur lebih lambat. Ini tidak mengganggu pengembangan, tapi **buka aplikasinya beberapa menit sebelum demo** supaya tidak ada jeda aneh di depan penonton.
- Kuota penyimpanan jauh lebih dari cukup untuk 25 dokumen sintetis.

## Verifikasi

- [ ] `SELECT extversion FROM pg_extension WHERE extname = 'vector';` mengembalikan versi
- [ ] Punya dua connection string, tahu mana pooled dan mana direct
- [ ] Ada branch `dev` terpisah dari produksi
- [ ] Region Singapore

---

# 2. Vercel

## Langkah

1. Daftar, lalu **Add New Project** dan import `AangOrg/IntraDocs`.
2. Framework preset akan terdeteksi sebagai Next.js. Biarkan perintah build bawaan.
3. **Region fungsi: Singapore (`sin1`).** Harus sama dengan region Neon. Ada di pengaturan project, bukan di halaman import — periksa lagi setelah deploy pertama.
4. Versi Node disamakan dengan isi `.nvmrc`.

Deploy pertama akan gagal karena aplikasinya belum ada. Itu normal; akan hijau setelah T-001 masuk.

## Environment variable

Vercel memisahkan **Production**, **Preview**, dan **Development**. Isi minimal Production **dan** Preview — kalau Preview kosong, setiap preview PR akan mati dan langkah review jadi tidak berguna.

| Nama | Nilai |
|---|---|
| `DATABASE_URL` | Neon pooled |
| `DATABASE_URL_UNPOOLED` | Neon direct |
| `AI_PROVIDER` | `openai` (atau sesuai penyedia) |
| Kunci penyedia AI | dari dasbor penyedia |
| `AI_MAX_CLASSIFICATION` | `secret` — lihat catatan di bawah |
| `AUTH_SECRET` | string acak, buat dengan `openssl rand -base64 32` |

**Catatan `AI_MAX_CLASSIFICATION`.** Nilainya `secret` selama sprint karena seluruh korpus fiktif dan tidak ada yang perlu dilindungi. Kalau disetel `public`, jalur AI hanya akan melihat dokumen publik dan perbedaan jawaban antar role — inti demo — hilang tanpa pesan error apa pun. Kalau nanti dipakai dengan dokumen sungguhan, turunkan nilainya sesuai kebijakan.

## Lindungi preview

Aktifkan **Vercel Authentication** untuk deployment Preview. Isinya memang sintetis, tapi tampilannya seperti dokumen internal perusahaan dan URL preview bisa ditebak. Tidak ada alasan membiarkannya terbuka.

## Jangan jalankan migrasi di perintah build

Menaruh `drizzle-kit migrate` di build command berarti setiap preview PR mengubah skema basis data produksi. Jalankan migrasi secara sadar dari lokal memakai `DATABASE_URL_UNPOOLED`, setelah PR-nya di-merge.

## Batas waktu fungsi — periksa ini di hari 5

Jawaban AI dialirkan lewat SSE, artinya fungsinya harus tetap hidup selama seluruh jawaban mengalir. Ada batas durasi fungsi, dan besarnya berbeda antar paket.

Di lokal batas ini tidak ada, jadi **kelas bug ini hanya muncul setelah dideploy.** Di hari 5, setelah endpoint chat jalan, ajukan satu pertanyaan yang jawabannya panjang **di preview Vercel, bukan di localhost**. Kalau aliran terpotong di tengah, itu batas durasi — bukan bug kode. Jalan keluarnya: `export const maxDuration` di route handler, dan kalau paketnya tidak mengizinkan, perpendek jawaban atau naikkan paket.

## Catatan lisensi

Paket Hobby Vercel ditujukan untuk penggunaan non-komersial. Untuk tugas magang dan eksperimen, wajar. **Kalau proyek ini benar-benar diadopsi Telkom, harus pindah ke paket berbayar atau dihosting sendiri.** Aturan portabilitas di ADR-0005 memang ditulis supaya perpindahan itu murah.

## Verifikasi

- [ ] Region fungsi Singapore
- [ ] Env var terisi di Production **dan** Preview
- [ ] Vercel Authentication aktif untuk Preview
- [ ] Tidak ada perintah migrasi di build command

---

# 3. GitHub

## Langkah

1. **Tambahkan rekan kerja** sebagai collaborator dengan akses Write. Tanpa ini yang bersangkutan tidak bisa membuka PR dari branch di repo ini.
2. **Aktifkan secret scanning dan push protection** di Settings → Code security. Push protection menolak commit yang mengandung kunci API sebelum sampai ke GitHub. Berdua dengan beberapa kunci berbeda, ini akan menyelamatkan Anda setidaknya sekali.
3. **Proteksi `main`** — lakukan **setelah CI pertama berjalan**:
   - Wajib lewat pull request
   - Wajib status check `verify` lulus
   - Larang force push dan penghapusan branch
4. **Izin Actions**: token bawaan cukup read-only.

## Kalau proteksi branch tidak tersedia

Proteksi branch tidak tersedia di semua kombinasi paket dan visibilitas repo. Kalau opsinya tidak muncul, jangan buang waktu — aturannya sudah tertulis di `AGENTS.md` dan berlaku tetap. Berdua orang, disiplin sudah cukup asalkan aturannya eksplisit.

## Yang sengaja tidak dipasang

Dependabot, CodeQL, dan templat issue. Semuanya berguna di proyek berumur panjang, tapi dalam sprint enam hari yang dihasilkan hanya notifikasi yang tidak sempat dibaca.

## Verifikasi

- [ ] Rekan kerja bisa membuka PR
- [ ] Push protection aktif
- [ ] `main` hanya bisa diubah lewat PR
- [ ] CI hijau di PR pertama yang berisi kode

---

# Di mana setiap rahasia tinggal

| Nilai | `.env.local` | Vercel Prod | Vercel Preview | Repo |
|---|---|---|---|---|
| `DATABASE_URL` | Neon branch `dev` | Neon produksi | Neon produksi | tidak pernah |
| `DATABASE_URL_UNPOOLED` | Neon branch `dev` | Neon produksi | — | tidak pernah |
| Kunci penyedia AI | ya | ya | ya | tidak pernah |
| `AUTH_SECRET` | ya | ya | ya | tidak pernah |
| `AI_PROVIDER` | ya | ya | ya | nama saja di `.env.example` |

`.env.example` hanya berisi nama variabel dengan nilai kosong, dan berkas itu yang di-commit. `.env.local` masuk `.gitignore`.

**Jangan pernah menempelkan nilai-nilai ini ke chat AI mana pun**, termasuk saat meminta bantuan debug. Kalau sebuah kunci pernah masuk chat, anggap bocor dan buat ulang.

---

# Urutan pengujian setelah semuanya siap

1. `pnpm install && pnpm dev` — aplikasi hidup di localhost
2. `pnpm db:migrate` memakai `DATABASE_URL_UNPOOLED` — tabel terbentuk
3. `pnpm seed` — data contoh masuk
4. Buka URL Vercel — halaman yang sama tampil dari internet
5. Buka PR percobaan — CI jalan dan preview terbuat

Kalau kelima langkah ini lulus, infrastrukturnya selesai dan tidak perlu disentuh lagi sampai demo.
