# ADR-0008: Dua lingkungan dan CI minimal

- Status: Diterima
- Tanggal: 2026-09-04

## Konteks

Pola umum dev → staging → produksi dirancang untuk tim yang perlu menguji rilis pada
lingkungan mirip produksi sebelum melepasnya ke pengguna. Pola itu menuntut biaya perawatan
tetap: satu database lagi, satu set rahasia lagi, satu tempat lagi yang bisa basi dan
menyesatkan.

Kita berdua, enam hari, dan penggunanya belum ada. Staging terpisah akan menghabiskan waktu
setup dan hampir pasti berakhir sebagai lingkungan yang tidak dipercaya siapa pun.

Sementara itu, Vercel sudah memberi **preview deployment untuk setiap PR** secara otomatis,
tanpa konfigurasi. Setiap PR mendapat URL sendiri yang bisa diklik reviewer.

## Keputusan

**Dua lingkungan, bukan tiga:**

| Lingkungan | Kapan | Database | Siapa yang lihat |
| --- | --- | --- | --- |
| Lokal | Saat menulis kode | Neon branch `dev` (dipakai bersama) | Yang menulis |
| Preview | Otomatis setiap PR | Neon branch `dev` yang sama | Kita berdua saat review |
| Produksi | Merge ke `main` | Neon branch `main` | Pembimbing, penguji, demo |

Preview per PR **menggantikan** staging. Tidak ada lingkungan keempat yang dirawat manual.

**Deploy dikelola Vercel, bukan oleh GitHub Actions.** Tidak ada workflow deploy yang kita
tulis sendiri. CI hanya memverifikasi, tidak melepas.

**CI hanya empat langkah:** `typecheck → lint → test → build`. Sekitar dua menit. Wajib hijau
sebelum merge ke `main`.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| dev → staging → prod penuh | Biaya perawatan tanpa manfaat pada skala ini |
| Neon branch per PR (integrasi Vercel–Neon) | Menarik, tetapi setiap preview lahir dengan database kosong sehingga perlu seed otomatis. Ditunda sampai kita benar-benar saling menimpa data |
| Tanpa CI, andalkan review manusia | Main yang rusak pada hari demo adalah kegagalan yang paling mudah dicegah |
| Deploy lewat GitHub Actions ke Vercel | Menulis ulang sesuatu yang sudah gratis dan lebih baik |
| E2E + Lighthouse CI + coverage gate | Menambah menit CI dan perawatan untuk risiko yang tidak kita hadapi minggu ini |

## Konsekuensi

**Lebih mudah:** nol perawatan lingkungan; setiap PR punya URL yang bisa diklik; umpan balik
CI dua menit.

**Lebih sulit:** karena preview memakai database `dev` yang sama, migrasi yang merusak pada
satu PR akan mengganggu preview PR lain. Itu sebabnya aturan migrasi aditif di
`docs/environments.md` bersifat mengikat, bukan saran.

**Risiko yang diterima:** produksi tidak diuji pada database bersalinan produksi sebelum
merge. Pada skala 25 dokumen sintetis, risikonya bisa diabaikan. Begitu ada konten nyata,
ADR ini perlu ditinjau ulang.

## Cara membatalkan

Menambahkan Neon branch per PR adalah mengaktifkan integrasi Vercel–Neon plus satu langkah
seed di CI — sekitar satu jam. Menambahkan staging terpisah adalah satu proyek Vercel baru
plus satu Neon branch — sekitar dua jam. Keduanya aditif dan tidak menuntut perubahan kode.
