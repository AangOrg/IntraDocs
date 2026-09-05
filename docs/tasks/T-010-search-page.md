# T-010: Halaman pencarian

Pemilik B; prasyarat T-002. Bisa paralel T-009 dengan fixture kontrak; integrasi wajib sebelum ditutup.

## Baca tambahan

API search dan ui-inventory.

## Lingkup

/cari menerima q/categoryId di URL; daftar judul/snippet/kategori/label/klasifikasi/verified/tanggal memakai respons API. Highlight literal kueri sebagai teks/elemen aman, tidak menginjeksi HTML respons. Cursor/limit mengikuti API.

Tombol Lanjutkan di chat membawa q ke /ai-assistant. Tampilkan mode keyword saat fallback, total/durasi dari respons. Filter kategori saja. Tidak ada kotak jawaban AI, bahkan bila task selesai lebih cepat.

## Kriteria terima

- [ ] Refresh/ganti kategori/pagination mempertahankan URL dan hasil konsisten.
- [ ] Loading/kosong/error serta fallback dapat dibedakan; kosong tidak mengisyaratkan dokumen tersembunyi.
- [ ] Snippet/query berisi markup berbahaya tidak dieksekusi.
- [ ] Tidak ada placeholder angka/judul terkunci; sumber total API, bukan array sebelum izin.
- [ ] Keyboard dan 375 px berfungsi; filter menutup tanpa menutupi hasil.
- [ ] Test komponen state/highlight dan uji integrasi endpoint selesai.

Pisahkan UI dan integrasi menjadi T-010a/b bila melebihi 8 berkas/400 baris; definisikan pembagian sebelum coding.
