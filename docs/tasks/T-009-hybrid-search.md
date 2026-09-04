# T-009 — Pencarian hybrid

Hari 4 · orang A · perkiraan setengah hari

## Tujuan

Satu endpoint pencarian yang menggabungkan kata kunci dan vektor, dibatasi izin pengguna, dan mencatat setiap kueri.

## Prasyarat

T-005 (filter izin) dan T-008 (korpus sintetis) sudah masuk `main`. Tanpa korpus, hasil penyetelan tidak berarti apa-apa.

## Baca dulu

`docs/architecture.md` bagian "Dua jalur data" · ADR-0006

## Berkas yang disentuh

`lib/search/hybrid.ts` · `lib/search/rrf.ts` · `app/api/search/route.ts` · `lib/ai/provider.ts` (pakai, jangan ubah) · migrasi indeks · `tests/search/*`

## Langkah

1. Tambah kolom `tsvector` tergenerasi di `chunk` beserta indeks GIN, dan indeks vektor untuk `embedding`. Keduanya lewat migrasi, bukan `push`.
2. Tulis pencarian kata kunci. **Gunakan konfigurasi teks `simple`** — Postgres tidak menyertakan kamus bahasa Indonesia, dan memaksa konfigurasi `english` akan melakukan stemming yang salah pada kata Indonesia. Kehilangan stemming sebagian ditutup oleh jalur vektor.
3. Tulis pencarian vektor dengan `embed()` dari `lib/ai/provider.ts`.
4. Jalankan keduanya paralel, masing-masing ambil 20 teratas, gabungkan dengan Reciprocal Rank Fusion (`k = 60`), kembalikan 10 teratas.
5. **Kedua kueri wajib menyertakan `visibleDocumentsFilter(user)` di klausa `WHERE`-nya sendiri.** Jangan menyaring hasil setelah kueri — penyaringan setelah kueri merusak peringkat dan mudah terlewat.
6. Kelompokkan potongan menjadi hasil per dokumen; tampilkan potongan dengan skor tertinggi sebagai cuplikan.
7. Catat satu baris `ai_query`: teks kueri, jumlah hasil, role penanya, durasi. Ini yang nantinya mengisi Kesenjangan Knowledge di dasbor.

## Kriteria terima

- `GET /api/search?q=...` mengembalikan hasil berperingkat dengan judul dokumen, kategori, klasifikasi, dan cuplikan.
- Kueri yang sama dijalankan sebagai viewer dan sebagai admin menghasilkan jumlah hasil yang berbeda.
- p95 di bawah 500 ms pada korpus sintetis.
- Setiap pencarian menghasilkan tepat satu baris `ai_query`.

## Test

`tests/search/search-visibility.spec.ts` — dokumen terbatas milik kategori lain tidak pernah muncul untuk reviewer bercakupan sempit, baik lewat jalur kata kunci maupun vektor. Uji keduanya terpisah; bug izin biasanya hanya ada di salah satu jalur.

## Di luar ruang lingkup

Reranker, saran ejaan, sorotan kata di cuplikan, filter selain kategori, penomoran halaman lebih dari satu halaman.
