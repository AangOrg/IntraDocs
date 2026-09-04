# T-005: RBAC + `visibleDocumentsFilter` + test kebocoran

- Pemilik: Orang A · Minggu 1 · Estimasi: 1,5 hari

> **Task paling penting di proyek ini.** Kalau ini salah, seluruh produk menjadi kewajiban
> hukum, bukan aset. Kerjakan dengan tenang dan jangan dipercepat.

## Tujuan

Satu fungsi yang menegakkan izin di dalam SQL, dipakai oleh setiap jalur baca, dan dijaga
oleh test kebocoran.

## Konteks

Baca `docs/rbac-matrix.md` seluruhnya, lalu
`docs/adr/0004-rbac-and-permission-aware-retrieval.md` dan
`docs/adr/0011-visibilitas-dokumen-di-luar-izin.md`.

## Lingkup

- `lib/rbac/visible-documents.ts` — `visibleDocumentsFilter(user): SQL`, mengimplementasikan
  aturan visibilitas persis seperti pada `docs/rbac-matrix.md`.
- `lib/rbac/can.ts` — `can(user, action, resource)`, satu fungsi dengan tipe action yang
  sempit. Tanda tangan ini ditetapkan di `docs/context-pack.md`; jangan menggantinya dengan
  sekumpulan fungsi terpisah seperti `canPublish()`.
- Helper perbandingan klasifikasi (`public < internal < restricted < secret`) sebagai fungsi
  murni, dengan unit test. Klasifikasi tertinggi per pengguna diturunkan dari `user.role`,
  bukan dari kolom clearance.
- Guard `AI_MAX_CLASSIFICATION` sebagai fungsi terpisah dan bisa diuji sendiri.
- Terapkan pada endpoint katalog dan dokumen yang sudah ada.
- Test suite di `tests/rbac/` sesuai daftar di `docs/rbac-matrix.md`.

## Acceptance criteria

- [ ] Filter dihasilkan sebagai SQL dan diterapkan **di dalam** query, bukan setelahnya
- [ ] Matriks test mencakup 4 role × 4 klasifikasi × (kategori dalam cakupan / di luar cakupan)
- [ ] Dokumen yang tidak terlihat mengembalikan `404`, bukan `403`, dan judulnya tidak muncul
      di katalog, hasil pencarian, maupun jumlah hasil pencarian (ADR-0011)
- [ ] Reviewer bisa melihat draft pada kategorinya, dan tidak pada kategori lain
- [ ] Kontributor selalu bisa melihat draft miliknya sendiri
- [ ] Tidak ada jalur pintas untuk `admin`; `admin` lolos karena aturannya, bukan karena dilewatkan

### Ketujuh test wajib — semuanya lulus sebelum task ini ditutup

Daftar ini disalin dari `docs/rbac-matrix.md` agar sesi eksekusi tidak perlu membuka berkas
kelima. Kalau keduanya berbeda, `docs/rbac-matrix.md` yang benar.

- [ ] `tests/rbac/ai-retrieval-leak.spec.ts` — potongan dokumen di luar izin tidak pernah
      masuk konteks LLM. **Ditulis pertama di proyek ini**, sebelum UI chat ada
- [ ] `tests/rbac/catalog-visibility.spec.ts` — katalog per role berisi tepat dokumen yang
      boleh dilihat
- [ ] `tests/rbac/search-visibility.spec.ts` — kedua jalur pencarian menyaring dengan filter
      yang sama
- [ ] `tests/rbac/category-scope.spec.ts` — reviewer dengan cakupan sempit tidak menerima
      dokumen terbatas dari kategori lain, lewat katalog, pencarian, maupun AI
- [ ] `tests/rbac/ai-max-classification.spec.ts` — batas klasifikasi jalur AI dihormati,
      diuji untuk keempat nilai
- [ ] `tests/rbac/audit-log.spec.ts` — pembacaan dokumen `restricted` dan `secret` tercatat
- [ ] `tests/rbac/inactive-user.spec.ts` — pengguna nonaktif tidak bisa masuk. Butuh pengguna
      keenam pada fixture T-003

## Batasan

- **Nol logic izin di dalam prompt LLM.**
- **Nol penyaringan setelah query** sebagai satu-satunya mekanisme.
- Jangan menduplikasi aturan visibilitas di file lain. Satu sumber kebenaran.
- Penulisan ulang pertanyaan lanjutan tidak boleh mengubah identitas penanya.

## Cara menguji

Seed enam pengguna sesuai fixture T-003 — lima pengguna pada `docs/rbac-matrix.md` ditambah
satu pengguna nonaktif — plus dokumen di setiap level klasifikasi dan dua unit berbeda. Untuk
setiap pengguna, ambil katalog, jalankan search, dan tanya AI, lalu bandingkan dengan daftar
yang diharapkan. Bila satu saja tidak cocok, berhenti dan perbaiki sebelum melanjutkan task
lain.
