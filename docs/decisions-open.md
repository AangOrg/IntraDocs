# Keputusan — dijawab sendiri

Sebelumnya daftar ini berisi sepuluh pertanyaan yang menunggu jawaban stakeholder. Untuk
konteks eksperimen dan tenggat satu minggu, menunggu jawaban akan menghabiskan seluruh waktu
yang ada.

Jadi **kita jawab sendiri**, dengan asumsi paling aman, lalu catat asumsinya di sini. Ketika
proyek ini benar-benar diteruskan, daftar ini menjadi agenda pertanyaan yang sudah rapi —
lengkap dengan apa yang sudah kita putuskan sementara dan biaya mengubahnya.

> Prinsip yang dipakai: **asumsi yang murah dibatalkan lebih baik daripada keputusan yang
> tertunda.** Setiap baris di bawah dipilih karena membalikkannya murah.

| # | Pertanyaan | Jawaban sementara kita | Biaya kalau ternyata berbeda |
| --- | --- | --- | --- |
| 1 | Bolehkah dokumen internal diproses LLM cloud? | **Tidak perlu dijawab sekarang.** Seluruh konten MVP sintetis dan fiktif, jadi tidak ada data nyata yang keluar. `AI_MAX_CLASSIFICATION=public`, provider cloud dipakai untuk pengembangan | Ganti env + `pnpm reindex`. Tanpa perubahan skema (ADR-0003, ADR-0006) |
| 2 | Di mana produksi berjalan? | **Vercel + Neon** untuk MVP. Aturan portabilitas tetap ditegakkan | ~0,5 hari untuk `Dockerfile` + compose (ADR-0007) |
| 3 | Dokumen nyata mana untuk pilot? | **Tidak ada.** 20–25 dokumen sintetis bergaya mockup. Ini juga menjadi eval set | Tidak ada — dokumen nyata cukup diunggah setelah fase 2 |
| 4 | Siapa peserta UAT? | **Pembimbing magang + rekan satu tim.** Cukup demo 15 menit dengan 3 skenario, bukan UAT formal | Tidak ada |
| 5 | SSO/OIDC tersedia? | **Tidak dipakai.** Akun lokal hasil seed; provider OIDC tetap ter-wire di balik flag | ~0,5 hari, kode sudah disiapkan |
| 6 | Baseline pertanyaan berulang per bulan? | **Dilewati.** Relevan untuk membuktikan dampak bisnis, bukan untuk MVP eksperimen | Tidak ada |
| 7 | Daftar unit resmi? | **Empat unit fiktif**: Infrastruktur, Keamanan Informasi, Aplikasi Internal, Data & Integrasi. Cukup untuk mendemokan cakupan unit pada RBAC | Ganti data seed |
| 8 | Retensi log pertanyaan AI? | **90 hari.** Datanya sintetis, jadi tidak ada risiko | Ubah satu konstanta |
| 9 | Ekspor PDF dengan watermark? | **Tidak.** | Fase 3 |
| 10 | Pemilik operasional setelah handover? | **Belum relevan** — statusnya eksperimen | Ditinjau bila diteruskan |

## Satu hal yang tetap tidak boleh dilanggar

Baris 1 dijawab "tidak perlu dijawab sekarang" **hanya karena seluruh konten sintetis.**
Asumsi itu runtuh begitu ada satu dokumen Telkom asli masuk ke sistem.

Jadi aturannya tetap mengikat: **jangan unggah dokumen internal asli ke environment yang
terhubung API publik** sampai baris 1 benar-benar dijawab oleh yang berwenang. Ini bukan
formalitas — ini satu-satunya asumsi di tabel ini yang mahal kalau salah.

## Kalau nanti proyek diteruskan

Tiga pertanyaan yang perlu diajukan, dengan urutan ini:

1. Izin pemrosesan AI untuk dokumen internal — cloud dengan kontrak, atau wajib on-prem?
2. Hosting produksi — internal atau cloud publik?
3. Dokumen apa yang boleh dipakai untuk pilot, dan siapa pemilik yang berwenang menyetujui?

Dua pertanyaan pertama sudah punya jalur teknis yang siap, jadi jawabannya tidak akan
memblokir apa pun.
