# Roadmap

Urutan pekerjaan aktif ada di docs/tasks/README.md; gerbang/cakupan di scope-mvp. Tanggal rilis fleksibel menurut ADR-0012. Tidak mengulang angka mutu atau jadwal per hari di sini.

## MVP

Irisan vertikal dokumentasi → publikasi → pencarian berizin → jawaban bersitasi. CRUD/auth juga memiliki risiko; tidak menganggap risiko hanya pada AI. T-013 menguji premis kualitas sebelum menyatakan rilis.

## Pasca-MVP — bukan izin implementasi sekarang

1. Tata kelola: approval, UI versi, admin pengguna/kategori/audit. Tabel versi/audit sudah ada; workflow approval tetap membutuhkan desain/data tambahan.
2. Konten: konversi format, object storage, queue, metadata AI/deteksi duplikat. Butuh spec/ADR, biaya parser dan operasi, bukan sekadar menyalakan flag.
3. Kecerdasan: reranker, dashboard, ekspor jawaban ke draf, topik populer dari log.
4. Perluasan: SSO/AD, ticketing/chat, API publik, on-prem; pengujian keamanan/operasi baru sebelum data sungguhan.

Permintaan akses tunduk ADR-0011: tidak membocorkan judul dokumen tersembunyi. Admin bercakupan sempit/clearance independen memerlukan ADR pengganti model, tidak otomatis tersedia karena kolom scope ada.

Setiap fitur tambahan menyebut biaya, dependency, penerima manfaat, dan apa yang diganti atau status pasca-MVP. Daftar teknologi ditolak tetap berlaku sampai keputusan baru; waktu longgar bukan alasan memasang infrastruktur.
