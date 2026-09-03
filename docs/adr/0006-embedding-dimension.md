# ADR-0006: Standardisasi dimensi embedding pada 1024

- Status: Diterima
- Tanggal: 2026-09-03

## Konteks

Provider AI belum ditentukan (ADR-0003). Setiap model embedding punya dimensi berbeda:
1536 untuk `text-embedding-3-small`, 3072 untuk `-large`, 1024 untuk `bge-m3`, 1024 untuk
`multilingual-e5-large`.

Kolom `vector(n)` di pgvector punya dimensi tetap. Kalau kita memilih 1536 sekarang lalu
pindah ke model lokal 1024, konsekuensinya bukan sekadar reindex — tapi **migrasi skema pada
tabel yang paling besar**, di tengah proyek yang sudah mepet.

## Keputusan

**Semua embedding distandardisasi pada 1024 dimensi**, apa pun providernya.

| Model | Cara mencapai 1024 |
| --- | --- |
| `bge-m3` | 1024 native |
| `multilingual-e5-large` | 1024 native |
| Cohere `embed-multilingual-v3` | 1024 native |
| OpenAI `text-embedding-3-small`/`-large` | parameter `dimensions: 1024` (Matryoshka) |

Skema:

```sql
embedding      vector(1024) NOT NULL,
embedding_model text        NOT NULL,   -- misal 'bge-m3'
content_hash    text        NOT NULL    -- SHA-256, agar tidak embed ulang tanpa perlu
```

Kolom `embedding_model` memungkinkan reindex bertahap dan perbandingan antar model.
`content_hash` mencegah pekerjaan embedding yang sia-sia ketika dokumen diedit sedikit.

1024 dipilih karena merupakan **titik temu** dari semua kandidat kuat yang mendukung bahasa
Indonesia — satu-satunya dimensi yang bisa dipenuhi model lokal maupun cloud tanpa kompromi.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| 1536 (default OpenAI) | Model multibahasa lokal terbaik tidak mencapai 1536; mengunci ke cloud |
| 3072 | Penyimpanan dan waktu query paling besar tanpa keuntungan berarti untuk korpus ini |
| Kolom per model | Menggandakan skema dan kode indexing |
| Simpan sebagai `jsonb` dan hitung di aplikasi | Kehilangan index pgvector; sangat lambat |

Catatan bahasa: korpus utamanya berbahasa Indonesia. Model multibahasa (`bge-m3`,
`multilingual-e5-large`) berkinerja lebih baik di sini dibandingkan model yang dominan
berbahasa Inggris — dan keduanya 1024 secara native. Keputusan ini sekaligus keputusan
kualitas, bukan hanya keputusan skema.

## Konsekuensi

**Lebih mudah:** ganti provider = ubah env + `pnpm reindex`. Tanpa migrasi skema. Ini yang
membuat ADR-0003 benar-benar bisa dijalankan, bukan sekadar niat baik.

**Lebih sulit:** pada model OpenAI, `dimensions: 1024` sedikit menurunkan kualitas
dibandingkan dimensi penuh. Selisihnya kecil dan sepadan dengan portabilitas yang didapat.

**Risiko yang diterima:** bila kelak ada model jauh lebih baik yang tidak mendukung 1024,
kita menghadapi satu migrasi skema — tepat pekerjaan yang keputusan ini tunda, dengan
kemungkinan terjadi yang jauh lebih kecil.

## Cara membatalkan

Buat kolom `vector(n)` baru, isi lewat job reindex bertahap sambil membaca dari kolom lama,
tukar setelah selesai, lalu hapus kolom lama. Kolom `embedding_model` sudah dirancang untuk
mendukung transisi seperti ini tanpa downtime.
