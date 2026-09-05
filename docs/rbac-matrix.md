# Matriks RBAC

Sumber: layar 8 mockup, ditambah definisi "terbatas" pada catatan di bawah matriksnya. Keputusan pemetaan ada di ADR-0004 dan ADR-0009.

Berkas ini adalah acuan pemeriksaan diff. Setiap perubahan pada jalur izin harus diperiksa terhadap tabel di bawah.

## Izin punya dua dimensi

Ini bagian yang paling mudah salah dibaca dari mockup, dan paling mahal kalau salah dimodelkan.

- **Role** menentukan *apa* yang boleh dilakukan
- **Cakupan kategori** menentukan *di mana* izin itu berlaku
- **Klasifikasi** menentukan *seberapa sensitif* dokumen yang boleh dilihat

Contoh dari mockup sendiri: Reviewer Keamanan tidak dapat menyetujui dokumen Aplikasi Internal. Rolenya cukup, cakupannya tidak.

Tanda pada matriks di bawah:

- `Y` diizinkan
- `S` diizinkan, terbatas pada cakupan kategori pengguna
- `N` tidak diizinkan

## Pemetaan role

Mockup punya lima role bawaan ditambah kemampuan membuat role kustom. MVP memakai empat.

| Role mockup | Role kita | Catatan |
| --- | --- | --- |
| Super Admin | `admin` | Digabung |
| Admin Knowledge | `admin` | Digabung |
| Reviewer | `reviewer` | |
| Contributor | `contributor` | |
| Viewer | `viewer` | |
| Role kustom | — | Fase 2, butuh editor izin |

Super Admin dan Admin Knowledge berbeda pada tepat satu sel matriks, yaitu mengelola pengguna dan role. Konsol pengguna tidak ada di MVP, jadi perbedaannya belum dapat diamati. Memisahkannya nanti berarti menambah satu nilai role dan satu pemeriksaan izin.

## Matriks izin

| Izin | admin | reviewer | contributor | viewer | Status MVP |
| --- | --- | --- | --- | --- | --- |
| Membaca dokumen Publik dan Internal | Y | Y | Y | Y | Masuk |
| Membaca dokumen Terbatas dan Rahasia | Y | S | S | N | Masuk |
| Mengunggah dan menyunting dokumen | Y | Y | Y | N | Masuk |
| Memakai AI Assistant | Y | Y | Y | Y | Masuk |
| Menyetujui atau menolak pengajuan | Y | S | N | N | Fase 2 |
| Mengelola kategori dan label | Y | N | N | N | Fase 2 |
| Mengelola pengguna dan role | Y | N | N | N | Fase 2 |
| Melihat audit log dan analitik | Y | S | N | N | Fase 2 untuk UI; penulisan log masuk MVP |

## Klasifikasi maksimum per role

| Role | Klasifikasi tertinggi yang boleh dibaca | Dibatasi cakupan kategori? |
| --- | --- | --- |
| `viewer` | `internal` | Tidak relevan |
| `contributor` | `restricted` | Ya, untuk `restricted` ke atas |
| `reviewer` | `restricted` | Ya, untuk `restricted` ke atas |
| `admin` | `secret` | Tidak |

Dokumen `public` dan `internal` terlihat lintas kategori untuk semua role, sesuai baris pertama matriks.

Tabel ini adalah **satu-satunya** penurunan klasifikasi tertinggi per pengguna. Tidak ada kolom clearance di tabel `user`: dua sumber kebenaran untuk satu aturan visibilitas dilarang ADR-0004. Lihat lingkup T-003.

## Satu filter, banyak pemakai

`lib/rbac/visible-documents.ts` mengekspor `visibleDocumentsFilter(user): SQL`. Fungsi ini adalah satu-satunya tempat aturan visibilitas ditulis.

Pemakainya wajib empat, tanpa kecuali:

1. Katalog dokumen
2. Pencarian kata kunci
3. Pencarian vektor
4. Retrieval untuk jawaban AI

Aturan yang tidak boleh dilanggar:

- Filter berada di klausa `WHERE`, tidak pernah di prompt LLM. Prompt bisa dibujuk, klausa `WHERE` tidak
- Penyaringan terjadi **sebelum** potongan dokumen masuk ke konteks LLM, bukan sesudah
- Penulisan ulang pertanyaan lanjutan tidak mengubah identitas penanya. Filter tetap memakai identitas asli
- Dokumen di luar izin menghasilkan 404, bukan 403. 403 memberi tahu bahwa dokumennya ada. ADR-0011 memperjelas: judulnya pun tidak muncul di mana pun, termasuk pada jumlah hasil pencarian
- Tidak ada jalur pintas untuk `admin` yang melewati fungsi ini. `admin` lolos karena aturannya, bukan karena dilewatkan

## Catatan penting tentang batas klasifikasi jalur AI

Env `AI_MAX_CLASSIFICATION` membatasi klasifikasi tertinggi yang boleh dikirim ke penyedia AI eksternal. Ini pengaman untuk penerapan sungguhan, dan menjawab persyaratan "data tetap di lingkungan perusahaan" pada layar 11.

Selama sprint, nilainya **harus** dibuka sampai `secret`, karena seluruh korpus bersifat sintetis dan fiktif sehingga tidak ada yang perlu dilindungi. Kalau dibiarkan pada `public`, jalur AI hanya pernah melihat dokumen publik, dan perbedaan jawaban antar role — yang justru merupakan inti demo — tidak akan pernah terlihat.

Saat sistem dipakai dengan dokumen sungguhan, nilainya diturunkan kembali dan model lokal dipakai untuk klasifikasi di atas batas itu.

## Test yang wajib ada

Ditulis pada hari 2, sebelum jalur AI dibangun. Ketujuhnya juga jadi kriteria terima T-005, dan prasyarat datanya jadi kriteria terima T-003 dan T-008.

| Berkas | Yang dibuktikan |
| --- | --- |
| `tests/rbac/ai-retrieval-leak.spec.ts` | Potongan dokumen di luar izin tidak pernah masuk konteks LLM |
| `tests/rbac/catalog-visibility.spec.ts` | Katalog per role berisi tepat dokumen yang boleh dilihat |
| `tests/rbac/search-visibility.spec.ts` | Kedua jalur pencarian menyaring dengan filter yang sama |
| `tests/rbac/category-scope.spec.ts` | Reviewer dengan cakupan sempit tidak menerima dokumen terbatas dari kategori lain, lewat katalog, pencarian, maupun AI |
| `tests/rbac/ai-max-classification.spec.ts` | Batas klasifikasi jalur AI dihormati |
| `tests/rbac/audit-log.spec.ts` | Pembacaan dokumen terbatas tercatat |
| `tests/rbac/inactive-user.spec.ts` | Pengguna nonaktif ditolak saat login meski kredensial benar, pada setiap role; Fajar Nugroho adalah fixture permanennya |

## Pengguna seed

Dipilih agar setiap perbedaan izin terlihat saat demo, dan agar peninjau bisa berganti role sendiri.

Tepat **enam pengguna: lima aktif dan satu nonaktif**. Lima nama berasal dari mockup; **Viewer Demo** adalah akun sintetis tambahan, bukan pengguna yang diklaim ada di layar 8. Kolom klasifikasi bukan kolom basis data — nilainya diturunkan dari `role` lewat tabel "Klasifikasi maksimum per role" di atas, dan ditulis di sini hanya sebagai pengingat saat membaca demo.

| Nama | Role | Klasifikasi (turunan `role`) | Cakupan kategori | Status |
| --- | --- | --- | --- | --- |
| Budi Hartono | `admin` | `secret` | Semua | Aktif |
| Andi Wijaya | `admin` | `secret` | Semua | Aktif |
| Dwi Kurniawan | `reviewer` | `restricted` | Keamanan Informasi saja | Aktif |
| Rizky Ananda | `contributor` | `restricted` | Infrastruktur, Data dan Integrasi | Aktif |
| Fajar Nugroho | `viewer` | `internal` | SOP & Proses Bisnis (baca) | **Nonaktif** (`is_active = false`) |
| Viewer Demo (sintetis) | `viewer` | `internal` | Semua | Aktif |

Dwi Kurniawan adalah pengguna terpenting untuk demo: dialah yang membuktikan bahwa izin punya dua dimensi, bukan satu.

Fajar Nugroho mengikuti layar 8: `viewer`, unit **Finance (mitra)**, status nonaktif, dan cakupan "SOP & Proses (baca)". Nama kategori itu dipetakan ke kategori seed **SOP & Proses Bisnis**; kata "(baca)" menjelaskan izin viewer, bukan kategori baru. Cakupan tersebut tidak mengubah aturan ADR-0009: dokumen `public` dan `internal` tetap terlihat lintas kategori bagi pengguna aktif. Fajar ditolak karena `is_active = false`, bukan karena kategorinya atau karena viewer tidak boleh menulis.

**Viewer Demo** memakai unit **Demo**, `is_active = true`, dan `category_scope = NULL` (semua kategori). Akun ini menggantikan peran viewer aktif untuk demo dan menjadi kontrol positif login pada `tests/rbac/inactive-user.spec.ts`. Empat pengguna aktif lainnya tetap seperti pada tabel; Fajar hanya dipakai untuk skenario penolakan login. Semua akun dan identitas di seed bersifat sintetis.

Pengujian penanda nonaktif pada role yang boleh menulis tetap dipertahankan: T-005 memakai salinan fixture sementara untuk `contributor`, `reviewer`, dan `admin` dengan `is_active = false`. Itu data uji terisolasi, bukan tambahan pengguna pada seed. Rencana seed tetap enam pengguna dan tidak lagi memakai Sari Puspita.
