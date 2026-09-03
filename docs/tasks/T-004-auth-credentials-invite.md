# T-004: Auth — akun lokal + invite

- Pemilik: Orang A · Minggu 1 · Estimasi: 1 hari

## Tujuan

Login yang aman tanpa bergantung pada SSO maupun infrastruktur email.

## Konteks

Baca `docs/rbac-matrix.md`, `docs/api-contract.md` bagian Autentikasi,
`docs/adr/0001-tech-stack.md`.

## Lingkup

- Auth.js dengan credentials provider, session berbasis cookie.
- Hash password dengan `@node-rs/argon2` (Argon2id, parameter yang wajar).
- Alur invite: admin membuat invite → sistem menghasilkan tautan bertoken (kadaluarsa 7 hari)
  → **admin menyalin tautan secara manual** → penerima menetapkan nama dan password.
- Baris pengguna membawa `role`, `clearance`, `unit` — ditetapkan saat invite dibuat.
- Rate limit pada login (per IP dan per email).
- Cookie `httpOnly`, `secure`, `sameSite=lax`.
- `scripts/seed-admin.ts` untuk membuat admin pertama.
- Provider OIDC sudah **ter-wire di balik feature flag** `AUTH_OIDC_ENABLED`, mati secara
  default.

## Kenapa invite manual, bukan email

Menyiapkan pengiriman email di lingkungan korporat butuh persetujuan, domain, dan SPF/DKIM —
semuanya di luar kendali kita dan bisa memakan berhari-hari. Menyalin tautan secara manual
membuat kita tidak diblokir sama sekali, dan pengiriman email bisa ditambahkan kapan pun
tanpa mengubah alur.

## Acceptance criteria

- [ ] Admin bisa membuat invite dan mendapatkan tautan yang bisa disalin
- [ ] Invite kadaluarsa dan invite yang sudah dipakai ditolak dengan pesan jelas
- [ ] Login gagal tidak membocorkan apakah email tersebut terdaftar
- [ ] Password di bawah kebijakan minimum ditolak di server, bukan hanya di klien
- [ ] Session memuat `role`, `clearance`, `unit`; tipenya aman di seluruh aplikasi
- [ ] Rate limit terbukti bekerja lewat test
- [ ] Menyalakan `AUTH_OIDC_ENABLED` menampilkan tombol SSO tanpa memutus login lokal
- [ ] `scripts/seed-admin.ts` berjalan idempotent

## Batasan

- Jangan menulis logic auth sendiri di luar Auth.js.
- Jangan mencatat password atau token ke log.
- Token invite harus acak secara kriptografis dan disimpan sebagai hash.

## Cara menguji

Seed admin → buat invite → buka di jendela incognito → selesaikan pendaftaran → login →
periksa isi session. Coba pakai ulang invite yang sama; harus ditolak.
