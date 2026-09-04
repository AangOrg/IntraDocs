# ADR-0004: RBAC & permission-aware retrieval

- Status: Diterima
- Tanggal: 2026-09-03

## Konteks

Dokumentasi internal punya tingkat kepekaan berbeda. Persyaratannya bukan hanya "ada RBAC
di UI", melainkan bahwa **fitur AI tidak boleh menjadi jalan pintas untuk melewati izin**.

Ini kegagalan paling umum dan paling berbahaya pada aplikasi RAG internal. Polanya selalu
sama: retrieval mengambil dari seluruh korpus, lalu prompt berisi kalimat "jangan ungkapkan
dokumen rahasia". Itu bukan kontrol keamanan — model bisa dibujuk, dan judul serta snippet
saja sudah merupakan kebocoran.

## Keputusan

**Penegakan izin terjadi di dalam query SQL, sebelum data meninggalkan database.**

Satu fungsi menjadi sumber kebenaran:

```ts
// lib/rbac/visible-documents.ts
export function visibleDocumentsFilter(user: SessionUser): SQL
```

Aturan yang mengikat:

1. Fungsi ini **wajib** dipakai oleh katalog, kedua jalur retrieval search (full-text dan
   vector), retrieval RAG, dan setiap endpoint yang mengembalikan dokumen atau chunk.
2. **Tidak boleh** ada instruksi izin di dalam prompt LLM. Prompt bukan lapisan keamanan.
3. **Tidak boleh** penyaringan setelah query di layer aplikasi sebagai satu-satunya mekanisme.
4. Tabel `chunk` menyimpan salinan `classification` dan `owner_unit` agar predikat yang sama
   bisa dijalankan pada chunk tanpa join.
5. Chunk yang tidak boleh dilihat pengguna tidak pernah masuk ke konteks LLM, sehingga tidak
   mungkin bocor lewat parafrase.

Model: empat role (`viewer`, `contributor`, `reviewer`, `admin`) dikalikan empat klasifikasi
(`public < internal < restricted < secret`), ditambah cakupan unit dan grant eksplisit.
Detail lengkap ada di `docs/rbac-matrix.md`.

**`tests/rbac/ai-retrieval-leak.spec.ts` adalah test pertama yang ditulis dalam proyek ini**,
sebelum UI chat dibuat.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| Filter setelah retrieval di aplikasi | Data sensitif sudah keluar dari DB; mudah terlewat di jalur baru |
| Instruksi izin di dalam prompt | Bukan kontrol keamanan sama sekali |
| Index vector terpisah per level klasifikasi | Menggandakan penyimpanan dan kompleksitas reindex tanpa manfaat tambahan |
| Postgres Row Level Security | Elegan, tetapi menyulitkan debug dan menyebarkan logic ke dua tempat |

## Konsekuensi

**Lebih mudah:** satu tempat untuk diaudit dan satu tempat untuk diperbaiki; setiap jalur
baca baru otomatis mewarisi aturan yang sama.

**Lebih sulit:** query retrieval menjadi lebih panjang, dan index harus dirancang agar
predikat izin tidak merusak performa vector search. Ini biaya yang layak dibayar.

**Risiko yang diterima:** filter izin dapat mengurangi jumlah kandidat sehingga jawaban
lebih sering abstain untuk pengguna dengan akses terbatas. Itu perilaku yang benar.

## Cara membatalkan

Tidak untuk dibatalkan. Ini persyaratan, bukan preferensi. Perubahan pada model role atau
klasifikasi harus lewat ADR baru yang menggantikan ADR ini.
