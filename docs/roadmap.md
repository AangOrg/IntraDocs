# Roadmap

## Sprint ini — irisan vertikal (enam hari kerja)

Papan task harian ada di `docs/tasks/README.md`. Ruang lingkup mengikat ada di `docs/scope-mvp.md`.

**Penomoran hari di seluruh repo ini adalah urutan pekerjaan, bukan tanggal kalender.** "Hari 4" berarti hari kerja keempat sprint ini, bukan tanggal tertentu. `docs/scope-mvp.md` menetapkan bahwa kalau gerbang mutu tidak tercapai, yang bergeser adalah tanggalnya dan bukan gerbangnya — jadi penomoran di bawah tetap berlaku meski tanggal rilisnya mundur. Urutan antar task yang tidak boleh dibalik ada di `docs/eksekusi.md`.

Sprint ini menembus fase 1 dan fase 3 roadmap mockup sekaligus, dan sengaja melewati fase 2.

Alasannya perlu dipahami sebelum ada yang mengusulkan mengurutkannya ulang: fase 1 (CRUD, kategori, pencarian kata kunci) hampir tanpa risiko teknis — hasilnya bisa diprediksi tanpa dikerjakan. Seluruh risiko produk menumpuk di fase 3: apakah jawaban berbahasa Indonesia cukup kredibel, dan apakah izin tetap rapat ketika dokumen mengalir melewati LLM. Mengerjakan fondasi berbulan-bulan lebih dulu berarti menemukan jawabannya di bulan kelima. Irisan vertikal menemukannya di hari kerja keenam. Kalau jawabannya jelek, yang hilang enam hari kerja, bukan lima bulan.

Gerbang: kalau di akhir hari 5 jawaban AI belum kredibel, atau hit@5 masih di bawah 0,7 seperti diukur pada hari 4 di dalam T-009, hari 6 dipakai untuk memperbaiki retrieval — bukan untuk menambah halaman. Angka gerbangnya ada di bagian "Gerbang mutu retrieval" pada `docs/scope-mvp.md`, dan tidak diulang di sini. Kalau satu hari tidak cukup, yang bergeser adalah tanggal rilisnya.

## Setelah MVP

Urutan di bawah mengikuti roadmap mockup, dikurangi yang sudah selesai lebih awal.

**Berikutnya — tata kelola** (fase 2 mockup)
Alur approval satu tahap, riwayat versi dengan UI, konsol pengguna dan role, konsol kategori & label, pengingat tinjau ulang, halaman audit log. Tabel `document_version` dan `audit_log` sudah ada sejak MVP, jadi ini pekerjaan tampilan dan alur, bukan pekerjaan data.

**Kemudian — kelengkapan konten**
Unggah multi-format (PDF, Word, Excel) lewat `unpdf` dan `mammoth`, penyimpanan berkas asli di S3-kompatibel (R2 atau MinIO), antrean pekerjaan untuk ekstraksi, usulan metadata oleh AI saat publikasi, deteksi nyaris duplikat. Tabel penopangnya (`document_file`, `job`) sengaja tidak dibuat di MVP; menambahkannya butuh ADR baru.

**Kemudian — kecerdasan lanjutan**
Reranker `bge-reranker-v2-m3`, dasbor analitik lengkap termasuk Kesenjangan Knowledge dengan tombol Tugaskan, ekspor jawaban jadi draf dokumen, rekomendasi topik di halaman muka dari log pencarian.

**Kemudian — perluasan** (fase 4 mockup)
SSO/OIDC penuh dengan sinkronisasi AD, integrasi ticketing dan chat internal, API publik, IntraDocs sebagai server MCP yang memakai ulang `visibleDocumentsFilter` sehingga agent lain bisa bertanya dengan izin yang sama, perluasan ke divisi non-IT.

Permintaan akses ke dokumen di luar izin, kalau memang dibangun, bentuknya per kategori atau per unit dan tidak boleh membocorkan judul. Syaratnya ada di ADR-0011.

## Yang tidak akan dikerjakan tanpa ADR baru

Memisahkan pencarian atau vektor keluar dari Postgres, memecah aplikasi jadi beberapa layanan, menambah Redis atau antrean eksternal, mengganti Drizzle, atau memakai framework agent. Ambang teknis untuk meninjau ulang yang pertama ada di `docs/architecture.md`.
