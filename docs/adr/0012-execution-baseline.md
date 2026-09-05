# ADR-0012: Baseline eksekusi berbasis mutu

- Status: Diterima sebagai baseline dokumentasi; berlaku setelah rangkaian PR penyelarasan digabung.
- Tanggal: 2026-09-05. Dasar: pemilik meminta perbaikan sampai siap dieksekusi dan mengutamakan kualitas.
- Memperbarui sebagian ADR-0007 (tenggat/pemotongan), memperjelas ADR-0004/0009 (model izin) dan ADR-0010 (riwayat). ADR lama tidak disunting.

## Konteks

Spec lama masih meminta Docker, invite, dan fitur versi yang sudah ditunda. Kontrak AI kehilangan scope dan identitas percakapan. Pernyataan bahwa ADR-0004 melarang clearance adalah interpretasi keliru: satu penegak izin dapat membaca beberapa atribut. Keputusan role-only ditetapkan di sini, bukan dinisbatkan ke ADR-0004.

## Keputusan

1. Tanggal rilis fleksibel. Gerbang integrasi dan rilis dibedakan di `docs/scope-mvp.md`; fitur wajib tidak dipotong otomatis karena nomor urutan potong. Mengubah cakupan memerlukan persetujuan dan revisi scope lebih dahulu.
2. MVP memakai klasifikasi maksimum turunan role, tanpa `user.clearance`. `user.unit`/`document.owner_unit` adalah metadata, bukan grant tambahan. Cakupan kategori membatasi pembacaan restricted/secret bagi non-admin; admin global. Aturan draft dan aksi ditetapkan pada matriks RBAC. Admin bercakupan sempit bukan kemampuan MVP.
3. Route AI tunggal adalah `POST /api/ai/ask`. Scope kategori/dokumen, riwayat milik pengguna, sitasi berversi, dan ID percakapan dalam stream tetap wajib. Tidak ada alias route lama.
4. Pencarian boleh memanggil `embed()`, tidak `chat()`. Kegagalan embedding menurunkan pencarian ke kata kunci dengan penanda mode. Full-text memakai `tsvector`/`ts_rank_cd`, bukan klaim BM25 native PostgreSQL. RRF tetap dipakai.
5. Riwayat tidak mengalahkan izin saat ini: autentikasi aktif, kepemilikan, serta izin sumber diperiksa sebelum riwayat ditampilkan atau dikirim ke provider, termasuk kondensasi. Riwayat yang sumbernya tidak lagi diizinkan ditolak utuh dengan 404; jangan sekadar menyembunyikan sitasinya. Maksimal dua putaran sebelumnya sesuai ADR-0010.
6. `.md`/`.txt` masuk lewat form teks/pembaca berkas sederhana ke payload dokumen, tanpa penyimpanan berkas asli/queue. Publikasi sinkron, pemindaian sensitif, dan pengindeksan mempunyai pemilik task T-006. T-015 bukan task tambahan.
7. Task besar dipecah menjadi subtask yang sudah ditulis dalam spec sebelum coding. Satu subtask terkecil = satu sesi/branch/PR, maksimal 8 berkas kode dan 400 baris total diff termasuk migrasi. Parent task selesai setelah seluruh subtask lulus.

## Konsekuensi dan biaya

Tidak menambah fitur produk. Menghilangkan interpretasi ganda membutuhkan sinkronisasi kontrak, fixture, dan seluruh spec. Pemeriksaan riwayat serta fallback search menambah test dan query, tetapi diperlukan oleh requirement izin/pencarian yang sudah ada. Tenggat bergeser bila mutu belum lulus.

## Alternatif dan pembatalan

Tidak mempertahankan clearance independen: MVP tidak membutuhkan editor clearance. Jangan mengklaim itu tidak mungkin secara arsitektural. Mengembalikannya, membatasi admin, atau mengubah aturan riwayat memerlukan ADR pengganti dan test kebocoran. Aturan 404 ADR-0011 tetap berlaku; atribusi historisnya: 404 sudah eksplisit di scope/matriks/AGENTS, sementara ADR-0004 mendasari larangan kebocoran judul.
