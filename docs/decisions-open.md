# Keputusan — semuanya sudah tertutup

Tidak ada lagi keputusan yang menunggu jawaban pihak lain. Semuanya diputuskan sendiri agar sprint bisa jalan, dan setiap keputusan dibuat supaya bisa dibalik dengan biaya kecil.

| Pertanyaan | Keputusan | Tempat | Biaya membalikkan |
|---|---|---|---|
| Di mana LLM dan embedding diproses | Abstraksi penyedia, API terkelola saat sprint, model lokal tinggal ganti env | ADR-0003 | Satu berkas |
| Sumber kebenaran konten: Postgres atau Git | Postgres, dengan skrip ekspor ke Markdown | ADR-0002 | Satu berkas, sudah disiapkan |
| Dimensi embedding | 1024 | ADR-0006 | Reindeks penuh |
| Target deploy | Vercel + Neon, dengan aturan portabilitas ketat | ADR-0005 | Terlokalisasi |
| Autentikasi MVP | Akun lokal, OIDC di balik flag | ADR-0001 | Tambahan, bukan penggantian |
| Berapa lingkungan | Dua: lokal dan produksi, plus preview otomatis per PR | ADR-0008 | — |
| Ruang lingkup MVP | Dipersempit ke irisan vertikal enam hari | ADR-0007 | — |
| Cakupan kategori pada izin | Dimensi kedua di samping role | ADR-0009 | Satu kolom |
| Percakapan berlanjut | Kondensasi pertanyaan, tetap satu retrieval | ADR-0010 | — |
| Tool calling di aplikasi | Tidak untuk MVP | `docs/agent-tooling.md` | — |

## Dua hal yang menunggu tindakan, bukan keputusan

1. Akun Neon dengan `pgvector`.
2. API key penyedia AI.

Keduanya sudah diputuskan bentuknya; tinggal dibuatkan akunnya.

## Kalau mau membuka ulang salah satu keputusan

Tulis ADR baru yang menyatakan ADR lama digantikan. Jangan mengubah ADR lama dan jangan mengubah arah teknis tanpa jejak tertulis — dalam proyek yang dikerjakan lewat banyak sesi AI, keputusan tanpa jejak akan diputuskan ulang berkali-kali dengan hasil berbeda.
