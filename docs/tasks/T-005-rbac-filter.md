# T-005: RBAC + `visibleDocumentsFilter` + test kebocoran

- Pemilik: Orang A · Minggu 1 · Estimasi: 1,5 hari

> **Task paling penting di proyek ini.** Kalau ini salah, seluruh produk menjadi kewajiban
> hukum, bukan aset. Kerjakan dengan tenang dan jangan dipercepat.

## Tujuan

Satu fungsi yang menegakkan izin di dalam SQL, dipakai oleh setiap jalur baca, dan dijaga
oleh test kebocoran.

## Konteks

Baca `docs/rbac-matrix.md` seluruhnya, lalu
`docs/adr/0004-rbac-and-permission-aware-retrieval.md`.

## Lingkup

- `lib/rbac/visible-documents.ts` — `visibleDocumentsFilter(user): SQL`, mengimplementasikan
  aturan visibilitas persis seperti pada `docs/rbac-matrix.md`.
- `lib/rbac/can.ts` — pemeriksaan aksi: `canCreateDraft`, `canPublish(categoryId)`,
  `canManageUsers`, dan seterusnya.
- Helper perbandingan klasifikasi (`public < internal < restricted < secret`) sebagai fungsi
  murni, dengan unit test.
- Guard `AI_MAX_CLASSIFICATION` sebagai fungsi terpisah dan bisa diuji sendiri.
- Terapkan pada endpoint katalog dan dokumen yang sudah ada.
- Test suite di `tests/rbac/` sesuai daftar di `docs/rbac-matrix.md`.

## Acceptance criteria

- [ ] Filter dihasilkan sebagai SQL dan diterapkan **di dalam** query, bukan setelahnya
- [ ] `tests/rbac/ai-retrieval-leak.spec.ts` ada dan lulus, walaupun UI chat belum dibuat
- [ ] Matriks test mencakup 4 role × 4 klasifikasi × (unit sama / unit berbeda)
- [ ] Dokumen yang tidak terlihat mengembalikan `404`, bukan `403` — tidak membocorkan
      keberadaannya
- [ ] Reviewer bisa melihat draft pada kategorinya, dan tidak pada kategori lain
- [ ] Kontributor selalu bisa melihat draft miliknya sendiri
- [ ] Test guard `AI_MAX_CLASSIFICATION` lulus untuk keempat nilai
- [ ] Akses `restricted`/`secret` tercatat di `audit_log`

## Batasan

- **Nol logic izin di dalam prompt LLM.**
- **Nol penyaringan setelah query** sebagai satu-satunya mekanisme.
- Jangan menduplikasi aturan visibilitas di file lain. Satu sumber kebenaran.

## Cara menguji

Seed enam pengguna yang mencakup kombinasi role dan clearance, plus dokumen di setiap level
klasifikasi dan dua unit berbeda. Untuk setiap pengguna, ambil katalog, jalankan search, dan
tanya AI — lalu bandingkan dengan daftar yang diharapkan. Bila satu saja tidak cocok,
berhenti dan perbaiki sebelum melanjutkan task lain.
