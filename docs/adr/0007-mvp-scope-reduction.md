# ADR-0007: Pemotongan scope untuk MVP satu minggu

- Status: Diterima
- Tanggal: 2026-09-03
- Mengubah sebagian: ADR-0005 (Docker sejak hari pertama), `docs/architecture.md` (pola job)

## Konteks

ADR-0001 sampai 0006 dan roadmap awal disusun dengan asumsi **empat minggu** dan proyek yang
akan langsung dipakai produksi.

Dua hal berubah setelah perencanaan awal:

1. **Target MVP menjadi satu minggu**, bukan empat.
2. **Konteksnya tugas magang dan eksperimen.** Kalau hasilnya bagus, baru akan dipakai.

Perubahan kedua penting dan mengubah kalkulasi. Rencana empat minggu memuat pekerjaan yang
benar untuk sistem produksi: job queue, konversi PDF, object storage, alur approval, Docker,
observability. Semua itu **benar secara teknis, tetapi salah secara waktu** — dan yang lebih
buruk, semuanya adalah pekerjaan yang **tidak terlihat saat demo**.

Risiko sebenarnya bukan arsitektur yang kurang matang. Risikonya adalah **menghabiskan enam
hari pada infrastruktur lalu tidak punya apa pun yang bisa ditunjukkan.**

## Keputusan

Potong scope menjadi satu irisan vertikal yang berfungsi utuh, sesuai
`docs/scope-mvp.md`. Tiga pemotongan yang paling berdampak:

**1. Job queue dihapus dari MVP.** Embedding dijalankan langsung saat publish. Pada 25
dokumen, satu dokumen menghasilkan sekitar 10 chunk, yaitu satu panggilan API embedding —
sekitar dua detik. Queue baru punya alasan untuk ada ketika konversi PDF masuk, karena itulah
yang benar-benar lambat. Membangun queue sekarang berarti membangun infrastruktur untuk beban
yang belum ada.

**2. Konversi PDF/DOCX ditunda; MVP menerima `.md` dan `.txt`.** Ini pemotongan yang paling
menyakitkan karena "unggah berkas" adalah salah satu permintaan awal. Tetapi konversi PDF ke
Markdown adalah pekerjaan dengan hasil paling tidak pasti di seluruh proyek: tabel rusak,
list bertingkat kacau, header dan footer menyusup ke isi. Menaruhnya di minggu pertama
berarti menukar fitur yang pasti bisa didemokan dengan fitur yang mungkin gagal.
Dengan `.md` dan `.txt`, alur unggah tetap ada dan tetap benar; hanya parser-nya yang
menyusul.

**3. Docker ditunda; MVP berjalan di Vercel + Neon.** ADR-0005 mengatakan `Dockerfile` ditulis
sejak hari pertama. Itu keputusan yang benar untuk proyek empat minggu yang menuju on-prem,
tetapi setengah hari yang tidak bisa dibayar minggu ini. **Aturan portabilitasnya tetap
berlaku sepenuhnya** — tanpa Vercel Blob, tanpa Edge runtime, tanpa filesystem lokal, tanpa
SDK khusus vendor, `runtime = 'nodejs'` eksplisit. Aturan itulah yang sebenarnya menjaga
portabilitas; `Dockerfile` hanya membuktikannya, dan pembuktiannya bisa menyusul dalam
setengah hari kapan pun dibutuhkan.

## Yang sengaja TIDAK dipotong

Empat hal ini tetap dikerjakan minggu ini walaupun terlihat seperti kandidat pemotongan:

| Tetap dikerjakan | Kenapa tidak dipotong |
| --- | --- |
| `visibleDocumentsFilter` di dalam SQL + test kebocoran | Menyusupkan izin ke sistem retrieval yang sudah jadi berarti menyentuh ulang setiap query. Sekarang murah, nanti mahal |
| Skema lengkap (`document_version`, `audit_log`, `embedding_model`, `content_hash`) | Menambah kolom pada tabel `chunk` yang sudah berisi data jauh lebih mahal daripada membuatnya sekarang dalam keadaan kosong |
| Embedding 1024 dimensi | Gratis sekarang, migrasi skema kalau tidak |
| Eval 10 pertanyaan | Satu-satunya cara membedakan "AI-nya terasa bagus" dari "AI-nya terbukti bagus". Sekitar 80 baris |

Polanya konsisten: **potong pekerjaan yang bisa ditempelkan, jangan potong pekerjaan yang
harus disusupkan.**

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| Jalankan rencana 4 minggu, terima MVP terlambat | Target tenggat adalah pemberian, bukan variabel |
| Bangun UI dulu dengan data palsu, backend menyusul | Menghasilkan demo yang tidak bisa dilanjutkan; utang teknis paling mahal |
| Potong RBAC, tambahkan nanti | RBAC menyentuh setiap query. Ini justru pekerjaan yang harus disusupkan |
| Potong sitasi, cukup jawaban AI biasa | Menghapus satu-satunya alasan produk ini bisa dipercaya |
| Pakai layanan RAG siap pakai | Tidak bisa menegakkan izin per dokumen |

## Konsekuensi

**Lebih mudah:** ada irisan utuh yang bisa didemokan di akhir minggu; keputusan yang tersisa
sedikit; permukaan bug jauh lebih kecil.

**Lebih sulit:** unggah PDF — yang mungkin diharapkan orang — belum ada. Ini harus
disampaikan terbuka saat demo, bukan disembunyikan: sebutkan sebagai langkah berikutnya yang
sudah dirancang, dan tunjukkan `docs/scope-mvp.md` sebagai buktinya. Menunjukkan bahwa
pemotongan dilakukan secara sadar justru lebih meyakinkan daripada fitur setengah jadi.

**Risiko yang diterima:** kalau ternyata ada lebih banyak waktu, sebagian pekerjaan fase 2
akan terasa seperti seharusnya dikerjakan lebih awal. Itu pertukaran yang benar — kelebihan
waktu jauh lebih mudah diatasi daripada kekurangan waktu.

## Cara membatalkan

Setiap item di kolom "KELUAR" pada `docs/scope-mvp.md` sudah punya estimasi biaya dan lokasi
kode. Semuanya bersifat aditif — tidak ada yang menuntut perubahan skema atau pembongkaran,
karena itulah kriteria yang dipakai untuk memilih apa yang boleh dipotong.
