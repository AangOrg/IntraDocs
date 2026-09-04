# T-001: Repo setup, CI, docker-compose

- Pemilik: Orang A · Minggu 0 · Estimasi: 1 hari

## Tujuan

Siapa pun bisa melakukan clone lalu menjalankan seluruh stack dengan dua perintah. CI
menolak kode yang tidak bisa dikompilasi.

## Konteks

Baca `docs/architecture.md`, `docs/adr/0001-tech-stack.md`, `docs/adr/0005-deployment-and-portability.md`.

## Lingkup

- Next.js 15 App Router, TypeScript **strict**, pnpm, `.nvmrc`.
- Struktur folder sesuai `docs/context-pack.md`.
- ESLint + Prettier, aturan minimal tetapi ditegakkan.
- `docker-compose.yml`: Postgres 16 + pgvector, MinIO. Volume persisten.
- `Dockerfile` multi-stage dengan Next.js `output: 'standalone'`.
- `.env.example` berisi semua variabel dengan komentar, tanpa nilai rahasia.
- Validasi env saat startup (zod) — gagal cepat dengan pesan jelas bila ada yang kurang.
- GitHub Actions: `typecheck` → `lint` → `test` → `build`, ditambah job mingguan
  `docker compose up` + cek `/api/health`.
- Branch protection pada `main`: wajib PR, wajib CI hijau, tanpa force-push.
- Pindahkan `intradocs-mockup_1.html` ke `design/mockup/` agar tidak ikut ter-build.

## Di luar lingkup

Skema database (T-003), autentikasi (T-004), komponen UI (T-002).

## Acceptance criteria

- [ ] `docker compose up` lalu `pnpm dev` → aplikasi berjalan di `localhost:3000`
- [ ] `pnpm typecheck && pnpm lint && pnpm test && pnpm build` semuanya hijau
- [ ] `docker build` menghasilkan image yang bisa dijalankan
- [ ] Menghapus satu env wajib → startup gagal dengan pesan yang menyebut nama variabelnya
- [ ] CI berjalan pada PR dan memblokir merge saat gagal
- [ ] `README.md` berisi setup yang bisa diikuti orang lain tanpa bertanya

## Batasan

- Tanpa Redis, tanpa dependensi tambahan di luar ADR-0001.
- `runtime = 'nodejs'` eksplisit pada route handler.
- Tanpa `any` di TypeScript.

## Cara menguji

Hapus `node_modules` dan volume Docker, lalu ikuti README dari nol seolah-olah baru pertama
kali. Kalau ada langkah yang tidak tertulis, README-nya belum selesai.
