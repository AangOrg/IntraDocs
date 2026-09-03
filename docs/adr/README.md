# Architecture Decision Records

Setiap keputusan yang **mahal untuk dibatalkan** ditulis di sini. Tujuannya bukan formalitas:
tanpa ADR, alasan sebuah keputusan hilang dalam dua minggu, lalu keputusan yang sama
diperdebatkan ulang, atau lebih buruk — dibatalkan oleh agent AI yang tidak tahu konteksnya.

## Kapan menulis ADR

Tulis ADR bila keputusan memenuhi salah satu dari ini:

- Mengubah skema database atau bentuk data.
- Menambah atau mengganti dependensi infrastruktur.
- Menyentuh keamanan, izin, atau kepatuhan.
- Membuat kita terikat pada satu vendor.
- Sudah pernah diperdebatkan lebih dari sepuluh menit.

Jangan menulis ADR untuk pilihan gaya kode atau hal yang bisa diubah dalam satu jam.

## Aturan

- Status: `Diterima`, `Diusulkan`, `Ditolak`, atau `Digantikan oleh ADR-XXXX`.
- **ADR tidak diedit setelah diterima.** Kalau keputusan berubah, buat ADR baru yang
  menggantikannya dan ubah status yang lama.
- Setiap ADR wajib punya bagian **Konsekuensi** dan **Cara membatalkan**. ADR tanpa dua
  bagian itu hanyalah opini.
- Sebutkan nomor ADR di deskripsi PR bila PR tersebut mengimplementasikannya.

## Indeks

| ADR | Judul | Status |
| --- | --- | --- |
| [0001](0001-tech-stack.md) | Tech stack | Diterima |
| [0002](0002-content-source-of-truth.md) | Postgres sebagai source of truth konten | Diterima |
| [0003](0003-ai-provider-and-data-classification.md) | Abstraksi provider AI & batas klasifikasi data | Diterima |
| [0004](0004-rbac-and-permission-aware-retrieval.md) | RBAC & permission-aware retrieval | Diterima |
| [0005](0005-deployment-and-portability.md) | Strategi deploy & portabilitas | Diterima — bagian Docker ditunda oleh ADR-0007 |
| [0006](0006-embedding-dimension.md) | Standardisasi dimensi embedding pada 1024 | Diterima |
| [0007](0007-mvp-scope-reduction.md) | Pemotongan scope untuk MVP satu minggu | Diterima |

ADR-0004 adalah satu-satunya yang **tidak boleh** dibatalkan tanpa ADR pengganti yang
eksplisit. Sisanya bisa ditinjau ulang bila konteks berubah.

## Template

```markdown
# ADR-XXXX: Judul

- Status: Diusulkan
- Tanggal: YYYY-MM-DD
- Pengambil keputusan: 

## Konteks

Apa yang memaksa kita memilih. Batasan yang nyata, bukan preferensi.

## Keputusan

Apa yang kita pilih, dinyatakan dengan tegas.

## Alternatif yang dipertimbangkan

| Opsi | Kelebihan | Kekurangan | Kenapa tidak dipilih |
| --- | --- | --- | --- |

## Konsekuensi

Apa yang menjadi lebih mudah. Apa yang menjadi lebih sulit. Apa yang kita terima sebagai
risiko.

## Cara membatalkan

Langkah konkret dan estimasi biaya untuk berpindah dari keputusan ini.
```
