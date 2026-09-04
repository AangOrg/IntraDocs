# Lingkungan, CI/CD, dan rilis

Keputusan dan alasannya ada di ADR-0008. Berkas ini adalah cara menjalankannya.

## Alur perubahan, dari menulis sampai dipakai

```
lokal (pnpm dev)
  ↓  git push branch feat/T-00X-slug
PR dibuka
  ↓  otomatis: CI (typecheck, lint, test, build) + preview URL Vercel
review: baca diff + klik preview URL
  ↓  merge ke main (hanya kalau CI hijau)
produksi: Vercel deploy otomatis
```

Tidak ada langkah manual di jalur ini selain review. Itu memang tujuannya.

## Aturan migrasi database — mengikat

Preview memakai database `dev` yang sama, jadi migrasi yang merusak akan mengganggu pekerjaan
orang lain. Empat aturan:

1. **Migrasi dihasilkan, bukan didorong.** `pnpm db:generate` menghasilkan berkas SQL,
   berkas itu **di-commit**. Jangan pernah menjalankan `drizzle-kit push` ke database yang
   dipakai orang lain.
2. **Aditif dulu.** Menambah kolom nullable, menambah tabel, menambah index: aman. Menghapus
   atau mengganti nama kolom: lakukan dalam dua langkah terpisah dengan jeda — tambah yang
   baru, pindahkan data, baru hapus yang lama di PR berikutnya.
3. **Satu PR, satu migrasi.** Kalau sebuah PR punya dua berkas migrasi, biasanya itu tanda
   PR-nya harus dipecah.
4. **Reset itu boleh selama konten masih hasil seed.** `pnpm db:reset && pnpm db:seed` jauh
   lebih murah daripada mengakali migrasi rumit. Aturan ini berhenti berlaku begitu ada
   dokumen yang ditulis manual di produksi — setelah itu, migrasi sungguhan saja.

## Rahasia

| Tempat | Isi | Aturan |
| --- | --- | --- |
| `.env.local` | Rahasia pengembangan | Ada di `.gitignore`. **Jangan pernah di-commit** |
| `.env.example` | Nama variabel + nilai contoh, tanpa nilai nyata | Di-commit. Wajib diperbarui saat ada variabel baru |
| Vercel › Environment Variables | Rahasia preview dan produksi | Diatur per lingkungan. Kunci produksi tidak dipakai untuk preview |

Variabel yang dibutuhkan: `DATABASE_URL`, `AUTH_SECRET`, `AI_PROVIDER`, `AI_API_KEY`,
`AI_MAX_CLASSIFICATION`.

Aktifkan **push protection** dan **secret scanning** di GitHub › Settings › Code security.
Gratis, sekali klik, dan menangkap kesalahan yang paling memalukan sebelum sampai ke remote.

Kalau sebuah kunci pernah ter-commit walau langsung di-revert: **anggap kunci itu bocor dan
putar kunci baru.** Riwayat Git tidak lupa.

## CI

Satu berkas, satu job, empat langkah. Ditambahkan pada PR scaffold, ketika `package.json`
sudah ada.

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build
        env:
          DATABASE_URL: postgres://ci:ci@localhost:5432/ci
          AUTH_SECRET: ci-only-not-a-real-secret
          AI_PROVIDER: openai
          AI_API_KEY: ci-placeholder
```

`pnpm build` memerlukan variabel lingkungan saat build, jadi nilai palsu di atas memang
disengaja — build tidak menyentuh database.

**Yang sengaja tidak ditambahkan ke CI, dan alasannya:**

| Tidak ditambahkan | Alasan |
| --- | --- |
| Playwright E2E | Test RBAC di Vitest menutup risiko terbesar dengan biaya jauh lebih kecil |
| Lighthouse CI | Budget performa dicek manual di hari 6 |
| Coverage gate | Mendorong test yang menaikkan angka, bukan test yang menangkap bug |
| Build Docker | Docker ditunda ke fase 2 (ADR-0007) |
| Matriks Node versi | Kita menargetkan satu versi, yang tercatat di `.nvmrc` |

## Branch protection pada `main`

GitHub › Settings › Branches › Add rule untuk `main`:

- Require a pull request before merging
- Require status checks to pass → pilih `verify`
- Require branches to be up to date before merging
- Do not allow bypassing the above settings

Dua menit setup. Mencegah kegagalan paling klasik: `main` rusak pada pagi hari demo.

## Rilis dan rollback

Vercel menyimpan setiap deployment. Rollback berarti membuka dashboard, memilih deployment
sebelumnya, dan menekan **Promote to Production** — hitungan detik, tanpa git.

Sebelum demo:

```bash
git tag -a demo-v1 -m "Kondisi yang didemokan" && git push origin demo-v1
```

Jadi kalau ada yang rusak setelah demo, keadaan yang berhasil masih punya nama.

## Dua peringatan yang perlu diketahui lebih awal

**1. Batas durasi fungsi.** Endpoint chat AI melakukan streaming. Pada paket Vercel gratis,
durasi fungsi dibatasi. Setel `export const maxDuration = 60` pada route handler chat, dan
jaga jawaban tetap pendek — yang memang sudah jadi bagian kontrak jawaban. Kalau streaming
terputus di produksi tetapi jalan di lokal, batas inilah tersangka pertama.

**2. Ketentuan pemakaian paket gratis.** Paket Hobby Vercel ditujukan untuk pemakaian
non-komersial. Untuk eksperimen magang dan demo internal, ini wilayah abu-abu yang bisa
diterima. Begitu proyek ini benar-benar dipakai sebagai perangkat internal perusahaan,
lingkungannya perlu dipindah ke paket berbayar atau di-hosting sendiri. Itulah gunanya
aturan portabilitas di ADR-0005 — pemindahan itu tidak boleh menuntut penulisan ulang.

## Checklist hari demo

- [ ] `main` hijau di CI
- [ ] URL produksi dibuka dari perangkat lain, bukan dari laptop yang dipakai ngoding
- [ ] Tiga akun demo bisa login, dan kredensialnya tertulis di tempat yang mudah dilihat
- [ ] Tag `demo-v1` sudah dibuat
- [ ] Satu pertanyaan di luar korpus sudah dicoba, dan AI-nya memang mengaku tidak tahu
