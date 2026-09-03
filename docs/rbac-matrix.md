# RBAC & model akses

Kontrol akses di IntraDocs punya dua dimensi:

- **Role** menentukan *apa yang boleh dilakukan* (aksi).
- **Klasifikasi + cakupan unit** menentukan *apa yang boleh dilihat* (data).

Keduanya harus dipenuhi. Role `admin` adalah satu-satunya yang melewati batas klasifikasi.

## Role

Empat role. Jangan menambah role tanpa ADR.

| Role | Baca | Buat draft | Setujui & publikasi | Kelola user & taksonomi |
| --- | --- | --- | --- | --- |
| `viewer` | sesuai klasifikasi & unit | — | — | — |
| `contributor` | sesuai klasifikasi & unit | ya | — | — |
| `reviewer` | ditambah semua draft pada kategori yang diampu | ya | ya, pada kategori yang diampu | — |
| `admin` | semua | ya | ya | ya |

Opsional dan murah: `auditor`, read-only terhadap `audit_log` tanpa akses konten.
Belum termasuk MVP.

## Klasifikasi

`public` < `internal` < `restricted` < `secret`

Di UI: Publik, Internal, Terbatas, Rahasia.

Setiap pengguna punya `clearance` bernilai salah satu dari daftar yang sama. Pengguna hanya
boleh melihat dokumen dengan `classification <= clearance`.

## Aturan visibilitas

Sebuah dokumen terlihat oleh pengguna bila:

```
role = admin
ATAU (
  status = 'published'
  DAN classification <= user.clearance
  DAN (
        classification <= 'internal'          -- terbuka untuk seluruh organisasi
     OR owner_unit = user.unit                -- cakupan unit
     OR EXISTS grant eksplisit (document_grant)
  )
)
ATAU (role = 'reviewer' DAN kategori dokumen termasuk yang diampu)   -- termasuk draft
ATAU (created_by = user.id)                                          -- draft milik sendiri
```

Catatan desain: dokumen `restricted` dan `secret` **tidak** otomatis terlihat oleh seluruh
organisasi walaupun clearance mencukupi — masih perlu kecocokan unit atau grant eksplisit.
Ini mencegah clearance tinggi berubah menjadi akses menyeluruh.

## Kontrak `visibleDocumentsFilter`

```ts
// lib/rbac/visible-documents.ts
export function visibleDocumentsFilter(user: SessionUser): SQL
```

Aturan penggunaan:

1. Fungsi ini menghasilkan **predikat SQL**, dan harus dipakai oleh:
   - daftar katalog dan halaman kategori
   - kedua jalur retrieval search (full-text dan vector)
   - retrieval RAG untuk AI chat
   - endpoint API apa pun yang mengembalikan dokumen atau chunk
2. **Tidak boleh** ada penyaringan izin setelah query di layer aplikasi sebagai satu-satunya
   mekanisme, dan **tidak boleh sama sekali** ada penyaringan izin di prompt LLM.
3. Tabel `chunk` menyimpan salinan `classification` dan `owner_unit` supaya predikat yang
   sama bisa dijalankan langsung pada chunk tanpa join.
4. Setiap penambahan jalur baca baru wajib menambah test kebocoran.

### Kenapa ini ditulis sebagai aturan keras

Kebocoran paling umum pada aplikasi RAG adalah: ambil top-50 chunk dari seluruh korpus, lalu
perintahkan LLM "jangan sebutkan yang rahasia". Itu **bukan kontrol keamanan**. Model bisa
dibujuk, dan judul beserta snippet dokumen saja sudah merupakan kebocoran.

## Batas klasifikasi untuk provider AI

Selain izin pengguna, ada satu batas lagi: `AI_MAX_CLASSIFICATION`.

| Nilai env | Arti |
| --- | --- |
| `public` | Hanya dokumen publik boleh dikirim ke provider AI. Dipakai saat provider belum disetujui. |
| `internal` | Sampai `internal`. |
| `restricted` / `secret` | Hanya setelah persetujuan keamanan dan legal. |

Chunk di atas batas ini tidak pernah dikirim ke provider, walaupun pengguna berwenang
membacanya. UI menjelaskan bahwa jawaban dibatasi kebijakan AI yang berlaku, dan pengguna
tetap diarahkan untuk membaca dokumennya langsung. Lihat ADR-0003.

## Audit

Catat ke `audit_log`:

- Setiap akses dokumen `restricted` atau `secret`, beserta jalur akses
  (katalog / search / chat / tautan langsung).
- Setiap keputusan publikasi dan penolakan review.
- Setiap perubahan role, clearance, unit, dan klasifikasi dokumen.
- Setiap aksi admin.

Jangan menyimpan isi dokumen di dalam audit log. Simpan referensi saja.

## Test wajib

| Test | Memastikan |
| --- | --- |
| `tests/rbac/catalog-visibility.spec.ts` | Katalog dan API tidak membocorkan dokumen di luar izin |
| `tests/rbac/search-visibility.spec.ts` | Kedua jalur retrieval terfilter di dalam SQL |
| `tests/rbac/ai-retrieval-leak.spec.ts` | AI chat tidak mengembalikan isi, judul, maupun snippet dokumen terlarang. **Test pertama yang ditulis di proyek ini** |
| `tests/rbac/ai-max-classification.spec.ts` | Guard klasifikasi provider ditegakkan |
| `tests/rbac/audit-log.spec.ts` | Akses sensitif terekam |
| `tests/rbac/reviewer-scope.spec.ts` | Reviewer tidak bisa mempublikasikan di luar kategorinya |
