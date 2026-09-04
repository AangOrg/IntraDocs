# T-014 — Pemolesan dan README

Hari 6 · orang B · perkiraan setengah hari

## Tujuan

Membuat aplikasi bisa dicoba orang lain tanpa didampingi.

## Baca dulu

`docs/ui-inventory.md`

## Berkas yang disentuh

`README.md` · `components/ui/*` · berkas halaman yang keadaannya masih kurang

## Langkah

1. Telusuri setiap halaman dan pastikan tiga keadaan ada: kosong, memuat, gagal. Halaman tanpa keadaan kosong terlihat rusak justru saat demo, karena demo sering menyentuh jalur yang datanya sedikit.
2. Periksa semua halaman di lebar 375 px.
3. Periksa halaman dokumen tidak ditemukan mengembalikan **404**, dan dokumen tanpa izin juga 404 — bukan 403.
4. Tulis ulang `README.md`: apa ini, cara menjalankan lokal dalam lima langkah, cara seed, akun demo beserta role dan cakupannya, dan **skenario demo lima menit yang ditulis sebagai langkah-langkah**.
5. Pastikan `pnpm seed` bisa dijalankan berulang dan hasilnya sama setiap kali.
6. Tag commit terakhir yang diketahui baik sebelum demo.

## Kriteria terima

- Orang yang belum pernah melihat repo ini bisa menjalankannya di lokal hanya dari README.
- Skenario demo bisa dijalankan dari awal sampai akhir tanpa menyentuh basis data secara manual.
- Tidak ada halaman yang menampilkan tumpukan error mentah.
- Tidak ada angka pengisi dari mockup yang tersisa di mana pun.

## Di luar ruang lingkup

Animasi, mode gelap, aksesibilitas menyeluruh, optimasi performa di luar target yang sudah ada.
