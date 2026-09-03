# ADR-0002: Postgres sebagai source of truth konten

- Status: Diterima
- Tanggal: 2026-09-03

> **ADR ini adalah satu-satunya file yang perlu diubah bila tim memilih pendekatan
> Git-as-source-of-truth.** Keputusannya sengaja dibuat mudah dibalik; lihat bagian terakhir.

## Konteks

Konten disimpan sebagai Markdown. Pertanyaannya: di mana kebenarannya tinggal — di
repositori Git (seperti gitdoc.ai dan docs-as-code pada umumnya) atau di database?

Ini keputusan paling menentukan pada proyek ini karena mempengaruhi RBAC, alur kontribusi,
siapa yang bisa berkontribusi, dan pengalaman produk secara keseluruhan.

## Keputusan

**Postgres adalah source of truth.** Isi dokumen disimpan sebagai teks Markdown dengan YAML
frontmatter di kolom `document_version.content`. Git dipakai untuk kode, bukan untuk konten.

Berkas asli (PDF, DOCX) tetap disimpan di object storage untuk keperluan audit dan
penelusuran ulang.

## Alternatif yang dipertimbangkan

| Kriteria | Git sebagai SoT | Postgres sebagai SoT |
| --- | --- | --- |
| RBAC per dokumen | Sangat sulit — Git tidak punya izin per file; tetap harus disimpan di DB | Alami, satu query |
| Kontributor non-developer | Harus belajar Git/PR, atau butuh CMS di atasnya | Langsung bisa lewat web |
| Riwayat versi | Sangat baik, gratis | Perlu tabel `document_version` (sederhana) |
| Diff & review | Sangat baik | Perlu dibuat sendiri di UI |
| Search & filter | Harus tetap diindeks ke DB | Sudah di tempatnya |
| Alur approval | PR review | Alur `review` di aplikasi |
| Backup | Gratis lewat remote | Perlu diatur (harus diatur juga untuk data lain) |
| Latensi tulis | Perlu commit + build/sync | Instan |
| Konsistensi izin & konten | Dua sistem yang harus disinkronkan | Satu sistem |
| Ketergantungan operasional | Perlu Git host + worker sinkron | Sudah ada |
| Cocok dengan mockup | Tidak — mockup menampilkan wizard unggah 4 langkah | Ya |

Git kalah pada dua hal yang justru merupakan persyaratan inti: **RBAC per dokumen** dan
**kontributor non-developer**. Yang menentukan: bahkan dengan Git sebagai SoT, kita **tetap**
membutuhkan Postgres untuk izin, search, embedding, dan audit — sehingga hasilnya adalah dua
sistem yang harus disinkronkan. Dalam proyek satu bulan dengan dua orang, itu adalah
sumber bug dan pekerjaan yang tidak terbayar.

## Konsekuensi

**Lebih mudah:** RBAC dan alur approval menjadi wajar; kontributor dari unit bisnis bisa ikut
tanpa mengenal Git; search, embedding, dan izin selalu konsisten karena satu sumber.

**Lebih sulit:** riwayat versi dan diff harus dibangun sendiri (tabel `document_version` yang
immutable, ditambah tampilan diff sederhana); backup Postgres menjadi kritis.

## Pengaman reversibilitas

Tiga pengaman ini mengubah keputusan ini dari "terkunci" menjadi "mudah dibalik":

1. **Format portabel.** Konten disimpan sebagai Markdown + YAML frontmatter, bukan format
   internal atau HTML. Isi kolom database bisa langsung ditulis menjadi berkas `.md` yang
   valid tanpa transformasi.
2. **`scripts/export-to-git.ts`** — sekitar 100 baris, mengekspor semua dokumen terpublikasi
   menjadi struktur folder `.md` dan melakukan commit ke repositori Git setiap malam. Ini
   memberi kita riwayat Git, diff, backup di luar database, dan **exit strategy** — dengan
   biaya sangat kecil. Satu arah saja: dari aplikasi ke Git.
3. **Impor satu arah (v1.1).** Bila developer ingin menulis dokumentasi teknis sebagai kode,
   tambahkan job impor dari repositori Git ke dalam aplikasi. Tetap satu arah, sehingga tidak
   ada resolusi konflik dua arah.

## Cara membatalkan

Jalankan `scripts/export-to-git.ts` — seluruh konten sudah berbentuk `.md` yang siap dipakai
oleh generator statis mana pun. Yang perlu dibangun ulang hanyalah lapisan izin, dan itu
memang tidak pernah bisa dipenuhi oleh Git.
