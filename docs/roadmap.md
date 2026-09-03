# Roadmap 4 minggu

Aturan keras: **setiap akhir minggu harus ada yang ter-deploy dan bisa didemokan.**
Bukan "hampir jadi".

## Minggu 0 — fondasi (2–3 hari, boleh tumpang tindih dengan Minggu 1)

Kunci keputusan yang tersisa · repo hygiene + branch protection + CI · `docker-compose`
(Postgres+pgvector, MinIO) · design token dari mockup · skema DB + migrasi awal ·
seluruh artifak perancangan · **ajukan permintaan SSO dan persetujuan AI provider ke
stakeholder hari ini**.

**Gate:** `docker compose up` lalu aplikasi kosong berjalan, CI hijau.

Task: T-001, T-002, T-003

## Minggu 1 — fondasi & konten

Auth (akun lokal + invite, OIDC di balik flag) · 4 role + `visibleDocumentsFilter` +
**test kebocoran** · CRUD dokumen Markdown · viewer (render + sanitize + metadata + versi) ·
katalog + kategori/label · **seed 20–30 dokumen sintetis yang realistis**.

**Gate:** login sebagai tiga role berbeda dan melihat katalog yang berbeda sesuai izin.

Task: T-004, T-005, T-006, T-007, T-008

## Minggu 2 — ingest & search

Upload → object storage → job konversi → Markdown → editor pratinjau/perbaikan · metadata &
klasifikasi · review satu tingkat → publish · **hybrid search + RRF** + filter · audit log ·
**verifikasi bahwa jalur AI on-prem bisa berjalan** (walau lambat).

**Gate:** unggah satu PDF nyata → menjadi Markdown → diperiksa → dipublikasikan →
ditemukan lewat search. Dan: uji "clone & run" oleh orang yang tidak menulis setup-nya.

Task: T-009, T-010, T-011, T-012

## Minggu 3 — AI Assistant

Job chunk + embed · endpoint RAG streaming dengan **filter izin di dalam query** · UI chat +
sitasi yang bisa diklik ke bagian dokumen + badge klasifikasi/tanggal/verifikasi · jalur
abstain · rate limit · **eval harness + tuning berbasis angka** · opsional: AI auto-fill
metadata.

**Gate:** hit@5 >= 0,8 pada eval set; test kebocoran RBAC lolos; 100% jawaban bersitasi.

Task: T-013, T-014, T-015, T-016

## Minggu 4 — hardening & benar-benar dipakai

**Feature freeze pada hari ke-24.**

Performa (index DB, budget bundle, LCP) · aksesibilitas dasar (keyboard, kontras, label) ·
**UAT dengan 5–10 pengguna nyata** · perbaikan berdasarkan temuan UAT · **backup + latihan
restore yang benar-benar dijalankan** · runbook operasional + panduan admin · dokumen
handover + materi demo.

**Gate:** lima pengguna nyata menyelesaikan tiga skenario tanpa dibantu.

## Buffer

Buffer 20% sudah termasuk di Minggu 4. Kalau Minggu 1–3 lancar, buffer dipakai untuk
nice-to-have. Kalau tidak — dan biasanya tidak — buffer menyelamatkan tenggat.
**Jangan pernah menjadwalkan fitur di dalam buffer.**

## Prioritas ketika waktu habis

Urutan yang dipotong lebih dulu, dari atas:

1. AI auto-fill metadata
2. Peringkat "paling banyak dibaca"
3. Konsol approval yang rapi (cukup daftar sederhana)
4. Riwayat versi di UI (data tetap disimpan)
5. Peringatan periode tinjau ulang

Yang **tidak boleh** dipotong dalam keadaan apa pun: RBAC dan test kebocoran, sitasi + jalur
abstain, pemeriksaan manusia atas hasil konversi, audit log, eval harness.

## Backlog v1.1 dan sesudahnya

Deteksi duplikasi (murah, sudah punya embedding) · approval berjenjang + auto-routing ·
alur permintaan akses · dashboard analytics · reranker · impor satu arah dari repo Git
(docs-as-code untuk developer) · SSO OIDC · OCR · XLSX/PPTX/ZIP · notifikasi email ·
ekspor PDF dengan watermark klasifikasi.
