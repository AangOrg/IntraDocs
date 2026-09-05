# ADR-0011: Visibilitas dokumen di luar izin

- Status: Diterima
- Tanggal: 2026-09-05
- Hubungan: **memperjelas ADR-0004**, tidak menggantikannya. ADR-0004 tetap berlaku utuh.

## Konteks

ADR-0004 menetapkan bahwa dokumen di luar izin menghasilkan 404, bukan 403, karena 403
sudah memberi tahu bahwa dokumennya ada. Aturan yang sama diulang di `docs/rbac-matrix.md`,
`docs/api-contract.md`, dan `AGENTS.md`.

Pemeriksaan layar 1-8 mockup menemukan bahwa mockup meminta perilaku yang berlawanan.
Pada layar 2 (hasil pencarian), dokumen terbatas **tetap muncul di daftar hasil** dengan pil
"Terbatas - akses via permintaan", dan catatan di layar itu menyatakan bahwa dokumen terbatas
tetap terlihat judulnya namun isinya terkunci, dengan opsi mengajukan permintaan akses ke
pemilik dokumen.

Ini satu-satunya tempat di seluruh mockup di mana rancangan mentor bertentangan **langsung**
dengan aturan keamanan kita, bukan sekadar lebih kaya dari MVP kita. `docs/mockup-alignment.md`
sampai sekarang hanya mencakup layar 9-11, sehingga pertentangan ini belum pernah dicatat.

Tanpa keputusan tertulis, T-006 (viewer dokumen) dan T-007 (katalog dan filter) akan menebak,
dan dua sesi eksekusi yang berbeda bisa menebak berbeda. Judul yang lolos ke daftar hasil
adalah kebocoran yang tidak akan tertangkap oleh `tests/rbac/ai-retrieval-leak.spec.ts`,
karena test itu memeriksa konteks LLM, bukan daftar hasil pencarian.

## Keputusan

**Dokumen di luar izin tidak terlihat sama sekali, termasuk judulnya.**

1. Dokumen yang tidak lolos `visibleDocumentsFilter(user)` tidak muncul di katalog, tidak
   muncul di kedua jalur pencarian, tidak masuk retrieval AI, dan tidak muncul sebagai
   judul terkunci di mana pun.
2. Endpoint dokumen menjawab `404` untuk dokumen semacam itu. Tidak ada `403`, tidak ada
   badan respons yang menyebut judul, pemilik, atau kategori.
3. Pil "Terbatas - akses via permintaan" pada layar 2 **tidak dibangun**. Yang dibangun
   hanyalah `ClassificationBadge` untuk dokumen yang memang boleh dilihat pengguna.
4. Alur "ajukan permintaan akses" tetap di luar MVP. Sudah tercantum pada daftar
   "Tidak masuk MVP" di `docs/scope-mvp.md`, dan ADR ini mengunci alasannya.
5. `PermissionDeniedState` dipakai hanya untuk aksi yang gagal pada sumber daya yang
   keberadaannya memang sudah diketahui pengguna - misalnya viewer menekan tombol sunting.
   `PermissionDeniedState` **tidak** dipakai pada dokumen yang keberadaannya rahasia; di situ
   yang tampil adalah halaman 404 biasa.
6. Jumlah hasil pencarian dihitung **setelah** filter izin. Menampilkan "41 hasil" lalu
   hanya merender 12 di antaranya juga merupakan kebocoran, karena selisihnya bisa dibaca.

## Kalau permintaan akses dibangun di Fase 2

Bentuknya tidak boleh membocorkan judul. Permintaan diajukan **per kategori atau per unit**
("saya butuh akses Terbatas pada kategori Keamanan Informasi"), bukan per dokumen, dan
diproses oleh admin yang memang sudah bisa melihat dokumen tersebut. Bentuk yang ada di
mockup - pengguna mengklik judul yang tidak boleh dilihatnya - tidak boleh dibangun apa pun
alasannya, dan perubahan pada aturan ini butuh ADR baru yang menggantikan ADR ini.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| Ikut mockup: judul terlihat, isi terkunci | Judul dokumen internal adalah kebocoran. "SOP Penanganan Insiden Kebocoran Data Pelanggan" sudah menceritakan kejadiannya tanpa perlu dibuka |
| Judul terlihat tetapi disamarkan | Kategori, tanggal, dan pemilik tetap bocor; menambah kode tanpa menghilangkan risiko |
| Terlihat hanya untuk `internal`, tersembunyi untuk `restricted` ke atas | Dua aturan visibilitas untuk satu pertanyaan; ADR-0004 secara eksplisit melarang duplikasi aturan |
| Tunda keputusannya sampai T-006 | Dua sesi eksekusi akan menebak berbeda, dan yang salah baru terlihat saat demo |

## Konsekuensi

**Lebih mudah:** satu aturan untuk seluruh jalur baca. Tidak ada cabang "terlihat tetapi
terkunci" yang harus diuji di katalog, pencarian, viewer, dan AI secara terpisah.

**Lebih sulit:** demo kehilangan satu momen visual dari mockup. Gantinya dipakai momen yang
lebih kuat dan lebih benar: Dwi Kurniawan (`reviewer`, cakupan Keamanan Informasi saja) dan
Budi Hartono (`admin`) menjalankan pencarian yang sama dan menerima jumlah hasil yang
berbeda, tanpa Dwi pernah tahu ada dokumen yang tidak ia lihat.

**Biaya yang dibayar dan harus disebut saat presentasi:** pada layar 2, produk kita terlihat
lebih sederhana daripada mockup. Itu keputusan sadar, bukan pekerjaan yang belum selesai.
Sebut nomor ADR ini kalau ditanya.

**Risiko yang diterima:** pengguna tidak punya cara menemukan bahwa dokumen yang ia butuhkan
ada tetapi tidak boleh ia lihat. Ia harus bertanya ke unitnya. Untuk korpus 20-25 dokumen
sintetis, biaya ini nol.

## Cara membatalkan

Tidak untuk dibatalkan pada MVP. Perubahannya lewat ADR baru yang menggantikan ADR ini, dan
ADR itu wajib menjelaskan bagaimana permintaan akses bekerja tanpa membocorkan judul.
