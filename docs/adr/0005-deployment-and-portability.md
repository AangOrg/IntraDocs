# ADR-0005: Strategi deploy & portabilitas

- Status: Diterima
- Tanggal: 2026-09-03

## Konteks

Target produksi belum ditetapkan — kemungkinan cloud publik seperti Vercel, kemungkinan
server internal Telkom. Untuk aplikasi dokumentasi internal perusahaan, hosting on-prem
adalah kemungkinan yang sangat nyata.

Kita perlu bergerak cepat sekarang tanpa membuat keputusan yang mengunci nanti.

## Keputusan

**Vercel-friendly, bukan Vercel-locked.** Berjalan di Vercel hari ini; berjalan dengan
`docker compose up` di server internal kapan pun, tanpa perubahan kode.

| Kebutuhan | Pilihan | Kenapa portabel |
| --- | --- | --- |
| Postgres | Neon (dev/demo) | pgvector + branching per PR; hanya SQL standar, tanpa fitur khusus vendor |
| Berkas | `@aws-sdk/client-s3` → Cloudflare R2 | API S3 yang sama untuk MinIO on-prem |
| Job | Tabel `job` + `POST /api/jobs/tick` | Vercel Cron memanggilnya, atau loop worker Docker memanggil fungsi yang sama |
| Runtime | `runtime = 'nodejs'` eksplisit | Edge runtime tidak ada di luar Vercel |
| Hash password | `@node-rs/argon2` | Tanpa native build yang rapuh; jalan di mana saja |
| Build | Next.js `output: 'standalone'` + `Dockerfile` | Image mandiri, siap sejak hari pertama |

Aturan yang menegakkan keputusan ini:

- **Jangan gunakan Vercel Blob, KV, atau Edge runtime.**
- **Jangan gunakan filesystem lokal untuk berkas** — tidak bertahan di serverless dan tidak
  bisa di-scale.
- **Jangan gunakan SDK Auth atau Storage milik Supabase**, bahkan bila Postgres-nya dipakai.
- `Dockerfile` dan `docker-compose.yml` ditulis pada hari pertama, dan **`docker compose up`
  diuji setiap minggu di CI**. Portabilitas yang tidak diuji akan membusuk dalam dua minggu;
  menemukannya di minggu keempat berarti kehilangan beberapa hari.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| Optimalkan penuh untuk Vercel | Migrasi ke on-prem menjadi pekerjaan berminggu-minggu |
| Docker saja sejak awal | Iterasi lebih lambat; preview per PR sangat berharga saat vibecoding |
| Kubernetes | Sangat berlebihan untuk satu aplikasi dan dua orang |

## Konsekuensi

**Lebih mudah:** deploy preview untuk setiap PR; keputusan hosting bisa ditunda tanpa biaya;
review keamanan internal lebih lancar karena opsi on-prem sudah terbukti bisa jalan.

**Lebih sulit:** beberapa kemudahan platform tidak dipakai; ada disiplin tambahan untuk
menguji jalur Docker secara berkala.

**Risiko yang diterima:** pola tick untuk job sedikit kurang efisien dibandingkan worker
yang berjalan terus. Pada beban ini, perbedaannya tidak berarti.

## Cara membatalkan

Pindah ke on-prem: arahkan `DATABASE_URL` ke Postgres internal, `S3_*` ke MinIO, jalankan
worker sebagai proses terpisah, `docker compose up`. Tidak ada perubahan kode aplikasi.
