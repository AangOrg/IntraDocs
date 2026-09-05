# Scope MVP — Mengikat

Urutan wewenang: berkas ini > ADR bernomor > `docs/context-pack.md` > dokumen lain. ADR-0012 menyelaraskan baseline eksekusi; ADR-0011 mengunci penyembunyian dokumen di luar izin.

## Definisi selesai

Tanggal rilis fleksibel: kualitas lebih utama daripada target kalender lama. MVP bukan klaim siap memakai dokumen perusahaan sungguhan. Seluruh konten/akun demo sintetis. Kelulusan dibuktikan di deployment yang dapat diakses penguji, bukan hanya localhost.

1. Empat role aktif bisa login; Fajar nonaktif ditolak. Katalog berbeda sesuai izin, tidak harus berbeda untuk setiap pasangan role.
2. Dokumen di luar izin mengembalikan 404 tanpa judul/snippet/jumlah tersembunyi.
3. Reviewer bercakupan sempit tidak menerima dokumen restricted kategori lain melalui katalog, kata kunci, vektor, AI, atau riwayat.
4. Pencarian identifier dan parafrase bekerja; saat embedding gagal, kata kunci tetap tersedia dengan penanda mode.
5. Jawaban faktual mempunyai sitasi yang benar, berisi versi dan tautan bagian dokumen; pertanyaan tanpa dasar menghasilkan abstain.
6. Pertanyaan lanjutan, penyimpanan percakapan, dan scope seluruh KB/kategori/dokumen bekerja tanpa melewati izin.
7. Form buat/sunting serta publikasi `.md`/`.txt` bekerja, termasuk pemeriksaan sensitif dan indeks sinkron.
8. `pnpm typecheck`, `pnpm lint`, `pnpm test`, `pnpm build` hijau; seluruh suite izin dan integrasi jalur baca lulus.
9. Seluruh gerbang rilis di bawah lulus; bukti angka, denominasi, lingkungan, dan skenario manual dilampirkan.
10. Halaman inti terbaca di 375 px dan bisa dinavigasi dengan keyboard; orang lain mengikuti README tanpa bantuan.

## Gerbang mutu — sumber angka tunggal

| Gerbang | Syarat | Konsekuensi gagal |
| --- | --- | --- |
| Lanjut integrasi RAG | hit@5 >= 0,70 pada kasus answerable | Perbaiki retrieval sebelum T-011 |
| Rilis | hit@5 >= 0,85 | Belum lulus rilis walau integrasi boleh lanjut |
| Keamanan | Kebocoran izin = 0 pada query, prompt, respons, sitasi, dan riwayat | Hentikan merge/rilis yang terdampak |
| Jawaban | 0 klaim faktual tanpa sitasi yang mendukung; abstain recall >= 0,90 pada kasus wajib abstain | Perbaiki, jangan ganti pertanyaan supaya lulus |
| Kinerja | p95 search < 500 ms; p95 token pertama < 3 detik | Perbaiki atau ajukan perubahan target, jangan sembunyikan mode gagal |

Metode dan ukuran sampel ada di `docs/eval/README.md`. Retrieval diukur tanpa LLM generasi; mutu jawaban dan token pertama membutuhkan evaluasi end-to-end terpisah. Sepuluh kasus sintetis adalah regression baseline kecil, bukan estimasi akurasi populasi.

## Masuk MVP

- Empat role, empat klasifikasi, cakupan kategori dan status aktif; enam akun seed sesuai `docs/rbac-matrix.md`. Auth.js kredensial; tanpa invite/SSO aktif.
- Satu filter SQL `visibleDocumentsFilter(user)` dan satu pemeriksa aksi `can(user, action, resource)`; sesi selalu memakai identitas/izin terkini.
- Sebelas tabel sesuai arsitektur; dokumen satu kategori, banyak label; kategori satu tingkat dengan `parent_id` untuk masa depan.
- Alur draft ke published tanpa approval. Riwayat versi disimpan immutable, tetapi tidak ada UI riwayat/rollback.
- Form teks dan masukan `.md`/`.txt` sederhana; berkas dibaca menjadi teks, bukan disimpan sebagai objek. Pratinjau Markdown disanitasi.
- Heading hierarkis dan anchor stabil; `chunk.heading_path` lengkap. Kedaluwarsa turunan tanggal/periode, bukan status baru.
- Publikasi: pemindaian pola sensitif, konfirmasi false-positive yang terikat revisi, embedding sinkron, perubahan versi/indeks atomik. Data asli/kredensial sungguhan tetap dilarang.
- Hybrid full-text + pgvector, RRF, filter kategori. Tidak ada filter lain di MVP.
- AI Assistant terpisah, scope, multi-turn, riwayat, sitasi berversi, abstain, dan feedback jawaban.
- `audit_log` untuk pembacaan restricted/secret termasuk penggunaan sumber AI. `ai_query` untuk setiap search/ask, termasuk nol hasil, abstain, mode, error, dan latensi.
- UI memakai token mockup, AppShell responsif, state loading/kosong/gagal; aksi ditolak 403 hanya bila keberadaan resource boleh diketahui.
- Korpus 20–25 dokumen sintetis; near-duplicate, dokumen kedaluwarsa, heading bertingkat, semua klasifikasi/kategori; 10 kasus eval baseline.

## Tidak masuk MVP

PDF/DOCX/XLSX/PPTX/HTML/OCR/ZIP ingest, object storage, queue, invite, approval, UI versi, permintaan akses, admin terbatas kategori, kategori hierarkis, role kustom, konsol admin/dashboard, favorit, riwayat baca, topik populer, penghitung popularitas, feedback dokumen, notifikasi, AD/SSO aktif, metadata AI, reranker, Docker/compose, rate limit aplikasi, Playwright sebagai dependensi test, PWA/offline.

## Perubahan cakupan

Tidak ada potongan otomatis. Riwayat percakapan, form, scope, dan heading hierarkis tetap wajib sampai pemilik menyetujui revisi scope. Penyederhanaan tampilan tidak pernah menghapus data heading/sitasi atau melemahkan izin.

Pasca-MVP: usulan metadata AI dan ekspor jawaban menjadi draf dapat ditinjau setelah gerbang rilis lulus. Setiap tambahan menyebut biaya dan apa yang diganti; waktu longgar bukan izin menambah fitur diam-diam.
