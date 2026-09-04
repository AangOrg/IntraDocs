# Serah terima ke chat AI lain

Dipakai saat pindah chat, pindah akun, atau kuota habis. Ada dua prompt — pakai yang sesuai.

## Prompt A — memulai satu task (yang paling sering dipakai)

Ini prompt normal untuk setiap sesi eksekusi. Ganti nomor task-nya.

```
Repo: https://github.com/AangOrg/IntraDocs
Saya mau kamu mengerjakan satu task saja: T-0XX.

Baca hanya empat berkas ini, dengan urutan ini:
1. docs/STATUS.md
2. AGENTS.md
3. docs/context-pack.md
4. docs/tasks/T-0XX-*.md

Jangan membaca berkas lain kecuali spec task itu menyebutnya.
Jangan membuka intradocs-mockup_1.html secara utuh.

Setelah membaca, sebelum menulis kode, balas dengan:
- rencana perubahan dalam maksimal 10 baris
- daftar berkas yang akan kamu sentuh
- apa yang belum jelas

Tunggu konfirmasi saya. Setelah selesai: buka PR, perbarui docs/STATUS.md di PR
yang sama, lalu berhenti. Jangan lanjut ke task berikutnya.
```

## Prompt B — pindah akun atau kuota habis

Dipakai kalau konteks percakapan hilang sepenuhnya dan orang barunya belum tahu proyek ini.

```
Kamu melanjutkan proyek yang sudah berjalan. Jangan menulis kode sebelum membaca.

Repo: https://github.com/AangOrg/IntraDocs
Proyek: IntraDocs — web dokumentasi internal dengan RBAC dan AI chat bersitasi.
Konteks: tugas magang 2 orang, dikerjakan dengan bantuan agent AI, MVP ditargetkan selesai
dalam enam hari kerja sejak sprint dimulai.

Baca berkas ini dengan urutan ini, dan hanya ini:
1. docs/STATUS.md        — keadaan hari ini
2. docs/context-pack.md  — fakta stabil dan nama identifier
3. docs/scope-mvp.md     — apa yang masuk MVP dan apa yang tidak (mengikat)
4. AGENTS.md             — aturan menulis kode
5. docs/eksekusi.md      — protokol satu task satu sesi
6. docs/adr/README.md    — indeks keputusan, baca ADR hanya bila disebut spec

Jangan membaca seluruh folder docs/. Itu akan menurunkan kualitas kerjamu.

Batasan yang tidak boleh dilanggar:
- Jangan pernah push ke main. Satu task satu branch feat/T-0XX-slug, lalu PR.
- Jangan merge PR atau menghapus branch tanpa saya minta eksplisit.
- Filter izin selalu di SQL lewat visibleDocumentsFilter, tidak pernah di prompt LLM.
- Seluruh konten dokumen bersifat sintetis. Jangan pernah memakai data Telkom asli.
- Bahasa Indonesia untuk teks UI dan dokumen, bahasa Inggris untuk identifier dan commit.

Setelah membaca, ringkas dalam 5 baris: hari sprint keberapa, apa yang sudah jadi, apa task
berikutnya, dan apa yang kamu butuhkan dari saya. Tunggu konfirmasi sebelum menulis kode.
```

## Yang harus ikut pindah bersama prompt

Tidak ada. Itu memang tujuannya. Semua yang dibutuhkan ada di repo.

Kalau ada keputusan penting yang diambil di dalam chat dan belum masuk repo, tuliskan dulu sebelum pindah — sebagai ADR baru kalau itu keputusan teknis, atau sebagai baris di `docs/STATUS.md` kalau itu keadaan sementara.

## Kredensial

Jangan pernah menempel `DATABASE_URL`, API key, atau secret apa pun ke dalam chat. Simpan di `.env.local` di mesin lokal dan di pengaturan environment Vercel. Repo hanya berisi `.env.example` dengan nama variabel dan nilai kosong.
