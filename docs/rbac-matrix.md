# Matriks RBAC

Acuan diff izin: ADR-0004, ADR-0009, ADR-0011, dan klarifikasi model pada ADR-0012. Satu penegak SQL tidak berarti hanya boleh membaca satu atribut. MVP secara eksplisit memilih maksimum klasifikasi turunan role, bukan clearance independen.

## Pembacaan

| Role | Maksimum | Cakupan restricted/secret |
| --- | --- | --- |
| viewer | internal | Tidak dapat membaca keduanya |
| contributor | restricted | `category_scope` |
| reviewer | restricted | `category_scope` |
| admin | secret | Global; tidak dibatasi kategori |

- Semua pengguna harus aktif dan terautentikasi. `public` bukan endpoint anonim.
- `category_scope` berupa array ID kategori; NULL = semua, array kosong = tidak ada kategori sensitif. Public/internal yang published tetap terlihat lintas kategori.
- `unit`/`owner_unit` adalah metadata, bukan jalur grant. Tidak ada `user.clearance` (keputusan ADR-0012, bukan larangan di ADR-0004).
- Selain klasifikasi/cakupan: published mengikuti tabel; draft hanya pemilik, reviewer pada kategori dalam cakupannya, atau admin. Batas klasifikasi tetap berlaku, termasuk draft milik sendiri. Status in_review/archived tidak dihasilkan alur MVP dan ditolak jalur baca MVP.
- Search dan RAG menambah syarat published pada filter visibilitas yang sama. Pemilik tidak dapat memasukkan draft ke jawaban AI.

## Aksi

| Aksi | viewer | contributor | reviewer | admin |
| --- | --- | --- | --- | --- |
| Cari/baca/AI | Sesuai izin baca | Sesuai izin baca | Sesuai izin baca | Sesuai izin baca |
| Buat dokumen | Tidak | Dalam cakupan tulis | Dalam cakupan tulis | Ya |
| Sunting/publish langsung | Tidak | Dokumen milik sendiri dalam cakupan | Dalam cakupan | Ya |
| Approval, kelola pengguna/kategori, UI audit | Fase 2 | Fase 2 | Fase 2 | Fase 2 |

Cakupan tulis non-admin mengikuti `category_scope` untuk semua klasifikasi, dengan batas klasifikasi role; NULL semua, kosong tidak ada. `can(user, action, resource)` memeriksa pemilik, status, kategori, dan klasifikasi target, termasuk saat perubahan metadata. Mengetahui dokumen tidak berarti boleh menyuntingnya.

## Penegakan lintas jalur

`lib/rbac/visible-documents.ts` → `visibleDocumentsFilter(user): SQL`; `lib/rbac/can.ts` → `can(user, action, resource)`. Admin tetap melewati fungsi, tidak bypass. Filter berlaku sebelum LIMIT/ranking/agregasi. Dokumen, snippet, judul, hitungan kategori/hasil, dan sitasi tidak membocorkan resource tersembunyi; akses langsung 404.

`AI_MAX_CLASSIFICATION` diperiksa terpisah sebelum teks dikirim ke provider, termasuk embedding dokumen dan riwayat untuk kondensasi. Nilai secret hanya aman untuk korpus sintetis ini. Tidak ada instruksi prompt sebagai pengganti kontrol akses.

Riwayat dan feedback hanya milik pengguna. Periksa pengguna aktif, kepemilikan, dan izin semua sumber dengan keadaan DB terkini sebelum membaca teks riwayat atau memanggil provider. Jika sumber hilang/tidak diizinkan/di atas batas AI: 404 untuk percakapan; mulai baru, jangan bocorkan teks lama. Perubahan role/cakupan atau penonaktifan tidak menunggu cookie kedaluwarsa.

`audit_log` mencatat aktor, dokumen/versi, aksi dan waktu untuk pembacaan restricted/secret, termasuk sumber yang diberikan ke AI; tidak menyimpan isi. Hitungan hasil bukan jumlah global sebelum filter.

## Pengguna seed — tepat enam

| Nama | Role | Unit | Cakupan | Aktif |
| --- | --- | --- | --- | --- |
| Budi Hartono | admin | Demo | NULL | Ya |
| Andi Wijaya | admin | Demo | NULL | Ya |
| Dwi Kurniawan | reviewer | Demo | Keamanan Informasi | Ya |
| Rizky Ananda | contributor | Demo | Infrastruktur & Jaringan; Data & Integrasi | Ya |
| Fajar Nugroho | viewer | Finance (mitra) | SOP & Proses Bisnis | Tidak |
| Viewer Demo | viewer | Demo | NULL | Ya |

Semua identitas sintetis. Fajar mengikuti status mockup; Viewer Demo tambahan untuk demo dan kontrol positif login. Unit Demo untuk empat akun pertama adalah pilihan fixture, bukan klaim tentang mockup. Tidak ada Sari Puspita atau pengguna ketujuh. Variasi nonaktif role lain dibuat sementara dalam test.

## Test wajib

| Berkas di `tests/rbac/` | Bukti |
| --- | --- |
| ai-retrieval-leak.spec.ts | Sumber tidak berizin tidak mencapai prompt |
| catalog-visibility.spec.ts | Daftar dan total tepat setelah filter |
| search-visibility.spec.ts | Full-text dan vektor masing-masing memakai SQL filter |
| category-scope.spec.ts | Cakupan termasuk NULL/kosong dan perubahan kategori |
| ai-max-classification.spec.ts | Guard empat tingkat, termasuk riwayat/embedding |
| audit-log.spec.ts | Pembacaan sensitif tercatat tanpa isi dokumen |
| inactive-user.spec.ts | Login dan sesi lama ditolak; akun aktif kontrol positif |

T-005 menguji query/guard nyata pada DB uji, tanpa menunggu HTTP AI. T-006/T-007/T-009/T-011 menambah integrasi jalur yang mereka bangun ke suite yang sama. Tes kepemilikan/izin ulang riwayat wajib pada T-011. Tidak boleh mengganti pembuktian SQL dengan filter array tiruan.
