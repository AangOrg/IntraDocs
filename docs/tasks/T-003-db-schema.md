# T-003: Skema, migrasi, fixture minimal

Pemilik A. Prasyarat T-001a dan DB uji Neon/pgvector. Task data lain menunggu seluruh T-003 selesai.

## Baca tambahan

`docs/architecture.md` bagian DB, `docs/rbac-matrix.md` tabel seed, ADR-0006.

## Subtask berurutan

- T-003a: user/category/label/document dan migrasi awal; current_version_id boleh nullable sebelum FK tahap berikut.
- T-003b: document_label/document_version/chunk, FK siklik yang tersisa, indeks GIN/HNSW.
- T-003c: audit_log/ai_query/conversation/message dan fixture minimal enam pengguna.

Migrasi kumulatif; tepat sebelas tabel domain setelah T-003c, tidak menuntut semuanya di migrasi pertama. Tabel bookkeeping migrasi bukan tabel domain. Setiap PR <=8 berkas kode dan <=400 baris termasuk SQL. Pecah lagi sebelum implementasi bila perlu.

## Lingkup

Field minimum mengikuti arsitektur, bukan daftar lama lima belas tabel. Tidak ada user.clearance, invite, job, review, document_file, document_grant atau document_view. Enum status boleh empat nilai; aplikasi MVP hanya menghasilkan draft/published.

Kategori/chunk menyimpan category_id untuk filter; query memeriksa induk current_version/published, tidak mengandalkan salinan izin saja. Snapshot versi immutable; draft_markdown disimpan terpisah. ai_query mencatat search maupun ask, role snapshot, jumlah hasil, mode/error/latensi; tidak salah menganggap semua log berasal dari generasi.

## Kriteria terima

- [ ] Migrasi generate/commit SQL berjalan dari DB kosong; rerun migrator tidak mengulang migrasi. Tidak memakai drizzle-kit push.
- [ ] Ekstensi vector tersedia; buat via migrasi bila izin memungkinkan, jika tidak dokumentasikan provisioning pemilik tanpa mengaku berhasil.
- [ ] Tepat sebelas tabel domain dan seluruh FK/ON DELETE/indeks dijelaskan. Versi tidak bisa dimutasi melalui jalur aplikasi dan diuji.
- [ ] Enam akun persis matriks: lima aktif termasuk Viewer Demo, Fajar nonaktif; seed ulang tidak mengaktifkan Fajar atau menggandakan akun.
- [ ] Fixture kecil mencakup empat klasifikasi, dua kategori sensitif, draft/published, NULL/kosong/sempit pada scope dalam data uji terisolasi.
- [ ] Seed T-003 hanya fixture minimal. Korpus 20–25 dokumen milik T-008; tidak ditulis dua kali.
- [ ] DB integration test membuktikan constraint; EXPLAIN query final ditambahkan T-009, bukan diklaim sudah diuji pada endpoint yang belum ada.
