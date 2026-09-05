# T-001: Scaffold, CI, deploy

Pemilik A. Prasyarat: baseline dokumen digabung; Neon/Vercel dari pemilik. Tidak ada tenggat tetap.

## Baca tambahan

`docs/environments.md` untuk kredensial/deploy; jika bertentangan, scope dan ADR-0007/0012 menang.

## Subtask berurutan

- T-001a: Next.js 15 App Router, TypeScript strict, pnpm, lint/format/test runner, scripts typecheck/lint/test/build; output standalone.
- T-001b: CI empat pemeriksaan, env validation zod, README setup, deployment preview/production. Masing-masing PR maksimal 8 berkas kode/400 baris; pecah rencana lagi sebelum coding bila hasil scaffold melebihi batas.

## Lingkup dan batas

Auth, skema, UI fitur milik task lain. Tidak membuat Dockerfile/compose, MinIO, storage, queue, atau job mingguan Docker. Jangan memindahkan mockup; sumber tetap `intradocs-mockup_1.html` di root. Tidak pernah membukanya utuh.

`.env.example` tanpa secret; nama/letak variable mengikuti environments. `AI_MAX_CLASSIFICATION=secret` hanya untuk data sintetis. Validasi secret AI saat jalur AI diaktifkan, bukan memblokir halaman scaffold yang belum memakai AI.

Branch protection dikonfigurasi pemilik/admin; jangan mengklaim sudah aktif tanpa memeriksa. Tidak merge otomatis.

## Kriteria terima

- [ ] Fresh install mengikuti README, `pnpm dev` jalan tanpa Docker.
- [ ] typecheck/lint/test/build hijau; ada smoke test bermakna, bukan test kosong agar CI hijau.
- [ ] CI PR menjalankan empat pemeriksaan termasuk build; build gagal terlihat.
- [ ] Env wajib pada fitur aktif gagal dengan pesan nama variable, tanpa mencetak nilainya.
- [ ] URL preview/production scaffold dapat dibuka; DB/provider belum siap ditulis sebagai blocker, bukan ditandai selesai.
- [ ] Region fungsi dan DB disamakan; pemilik melengkapi secret via pengaturan layanan, bukan chat/repo.

## Bukti

Lampirkan hasil perintah dan URL deploy; pembaruan STATUS hanya oleh chat eksekusi maksimal 15 baris.
