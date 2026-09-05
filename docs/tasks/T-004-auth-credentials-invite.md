# T-004: Auth.js kredensial tanpa invite

Nama berkas lama dipertahankan agar tautan tidak putus; invite tetap Fase 2. Pemilik A; prasyarat T-003 selesai.

## Baca tambahan

API bagian autentikasi; matriks RBAC; ADR-0001 bagian auth.

## Subtask

- T-004a: Auth.js credentials, hashing Argon2id sesuai stack, cookie/session dan facade login/logout.
- T-004b: guard identitas terkini dan test login/sesi nonaktif.

Tidak membuat mekanisme cookie/token session sendiri. Endpoint facade mengikuti kontrak dan menjalankan mekanisme/CSRF Auth.js. OIDC tidak diaktifkan; tidak membangun alur invite, UI pendaftaran, rate limiter aplikasi, atau email.

Session mengidentifikasi user; role/category_scope/is_active diperiksa ulang dari DB pada setiap permintaan terlindungi. Jangan percaya klaim role lama dalam cookie sebagai otorisasi final. Tidak ada clearance independen.

## Kriteria terima

- [ ] Lima akun aktif termasuk Viewer Demo login; Fajar dengan password benar mendapat 401 generik tanpa sesi.
- [ ] Email tak terdaftar/password salah/nonaktif tidak dapat dibedakan melalui pesan.
- [ ] Cookie httpOnly, secure pada HTTPS, sameSite sesuai Auth.js; logout membatalkan akses sesi.
- [ ] Sesi yang dibuat sebelum user dinonaktifkan ditolak pada permintaan berikutnya. Perubahan role/scope berlaku segera.
- [ ] Kredensial tidak masuk log; password seed hanya sintetis untuk demo, tidak dipakai di layanan nyata.
- [ ] Test negatif CSRF dan login/logout serta guard DB berjalan; suite inactive-user dipakai ulang T-005, bukan diduplikasi.

Bukti: test akun aktif/nonaktif dan sesi lama. Pengamanan produksi tambahan di luar MVP dicatat, bukan dijadikan alasan menyebut demo aman untuk data sungguhan.
