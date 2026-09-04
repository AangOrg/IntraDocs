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
- Dokumen di luar izin menghasilkan 404, bukan 403. 403 memberi tahu bahwa dokumennya ada
- Tidak ada jalur pintas untuk `admin` yang melewati fungsi ini. `admin` lolos karena aturannya, bukan karena dilewatkan

## Catatan penting tentang batas klasifikasi jalur AI

Env `AI_MAX_CLASSIFICATION` membatasi klasifikasi tertinggi yang boleh dikirim ke penyedia AI eksternal. Ini pengaman untuk penerapan sungguhan, dan menjawab persyaratan "data tetap di lingkungan perusahaan" pada layar 11.

Selama sprint, nilainya **harus** dibuka sampai `secret`, karena seluruh korpus bersifat sintetis dan fiktif sehingga tidak ada yang perlu dilindungi. Kalau dibiarkan pada `public`, jalur AI hanya pernah melihat dokumen publik, dan perbedaan jawaban antar role — yang justru merupakan inti demo — tidak akan pernah terlihat.

Saat sistem dipakai dengan dokumen sungguhan, nilainya diturunkan kembali dan model lokal dipakai untuk klasifikasi di atas batas itu.

## Test yang wajib ada

Ditulis pada hari 2, sebelum jalur AI dibangun.

| Berkas | Yang dibuktikan |
| --- | --- |
| `tests/rbac/ai-retrieval-leak.spec.ts` | Potongan dokumen di luar izin tidak pernah masuk konteks LLM |
| `tests/rbac/catalog-visibility.spec.ts` | Katalog per role berisi tepat dokumen yang boleh dilihat |
| `tests/rbac/search-visibility.spec.ts` | Kedua jalur pencarian menyaring dengan filter yang sama |
| `tests/rbac/category-scope.spec.ts` | Reviewer dengan cakupan sempit tidak menerima dokumen terbatas dari kategori lain, lewat katalog, pencarian, maupun AI |
| `tests/rbac/ai-max-classification.spec.ts` | Batas klasifikasi jalur AI dihormati |
| `tests/rbac/audit-log.spec.ts` | Pembacaan dokumen terbatas tercatat |
| `tests/rbac/inactive-user.spec.ts` | Pengguna nonaktif tidak bisa masuk |

## Pengguna seed

Dipilih agar setiap perbedaan izin terlihat saat demo, dan agar peninjau bisa berganti role sendiri.

| Nama | Role | Klasifikasi | Cakupan kategori |
| --- | --- | --- | --- |
| Budi Hartono | `admin` | `secret` | Semua |
| Andi Wijaya | `admin` | `secret` | Semua |
| Dwi Kurniawan | `reviewer` | `restricted` | Keamanan Informasi saja |
| Rizky Ananda | `contributor` | `restricted` | Infrastruktur, Data dan Integrasi |
| Fajar Nugroho | `viewer` | `internal` | Semua |

Dwi Kurniawan adalah pengguna terpenting untuk demo: dialah yang membuktikan bahwa izin punya dua dimensi, bukan satu.
