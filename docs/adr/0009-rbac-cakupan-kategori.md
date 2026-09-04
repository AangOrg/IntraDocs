# ADR-0009: RBAC dua dimensi — cakupan kategori

- Status: Diterima
- Tanggal: 4 September 2026
- Memperjelas sebagian: ADR-0004

## Konteks

Layar 8 mockup memperlihatkan matriks izin dengan tiga nilai: diizinkan, terbatas, dan tidak. Catatan di bawah matriks mendefinisikan "terbatas" secara eksplisit: izin hanya berlaku pada kategori atau unit kerja yang ditugaskan kepada pengguna tersebut. Contoh yang diberikan mockup sendiri: Reviewer Keamanan tidak dapat menyetujui dokumen Aplikasi Internal.

Kolom "Cakupan Kategori" pada daftar pengguna berisi nilai majemuk: "Infrastruktur, Data & Integrasi", "SOP, Onboarding", "Semua kategori".

Skema MVP kita hanya punya `user.unit` bernilai tunggal. Model itu tidak bisa menyatakan "Reviewer untuk dua kategori", sehingga seluruh kolom terbatas pada matriks tidak dapat direpresentasikan.

## Keputusan

Tambahkan `user.category_scope` pada skema hari 1 sebagai array id kategori, dengan `NULL` berarti seluruh kategori.

`visibleDocumentsFilter` menghormati kolom ini **hanya** untuk dokumen berklasifikasi `restricted` dan `secret`. Dokumen `public` dan `internal` tetap terlihat lintas kategori, sesuai baris pertama matriks yang memberi tanda centang penuh kepada semua role.

Alasan menambahkannya sekarang, bukan nanti: kolom ini masuk ke klausa `WHERE` yang sama dengan seluruh pemeriksaan izin, dan klausa itu diuji oleh `tests/rbac/ai-retrieval-leak.spec.ts` pada hari 2. Menambah dimensi izin setelah test hijau berarti menulis ulang test dan seluruh data seed. Biayanya sekitar sepuluh menit hari ini, beberapa jam pada hari 4.

## Konsekuensi

- Seed harus memberi setidaknya satu pengguna cakupan sempit agar perbedaannya terlihat saat demo. Dwi Kurniawan: Reviewer, cakupan Keamanan Informasi saja.
- `tests/rbac/` bertambah satu kasus: reviewer dengan cakupan sempit tidak boleh menerima potongan dokumen terbatas dari kategori di luar cakupannya, baik lewat katalog, pencarian, maupun jawaban AI.
- Matriks di `docs/rbac-matrix.md` kini punya tiga nilai, bukan dua.
- Ketika approval masuk pada fase 2, kolom ini sudah menjadi tempat yang benar untuk membatasi siapa boleh menyetujui apa. Tidak ada pekerjaan tambahan saat itu.

## Cara membatalkan

Set `category_scope` menjadi `NULL` untuk semua pengguna. Filter menjadi tidak berpengaruh tanpa perubahan kode, dan kolomnya boleh ditinggalkan.
