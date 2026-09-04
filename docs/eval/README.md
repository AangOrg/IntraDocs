# Eval harness

## Kenapa ini wajib ada

Tanpa eval, "AI-nya bagus" hanyalah perasaan. Setiap perubahan pada chunking, prompt,
embedding, atau bobot RRF menjadi tebakan, dan kualitas bergerak naik-turun tanpa ada yang
sadar sampai pengguna mengeluh.

Mockup menampilkan angka **96% akurasi jawaban AI** di hero. Angka itu placeholder. Aturan
proyek ini: **jangan pernah menampilkan angka akurasi yang tidak dihasilkan oleh harness
ini.** Kalau angkanya belum diukur, jangan tampilkan sama sekali.

Eval juga yang membuat ADR-0003 bisa dijalankan: ketika provider AI final ditentukan, kita
bisa mengukur apakah kualitasnya naik atau turun, bukan berdebat.

## Format

`docs/eval/questions.jsonl` — satu pertanyaan per baris:

```json
{"q": "bagaimana cara reset password domain?", "expect_docs": ["SOP-IT-014"], "expect": "self-service portal, verifikasi OTP", "role": "viewer", "clearance": "internal"}
```

| Field | Arti |
| --- | --- |
| `q` | Pertanyaan, ditulis seperti pengguna sungguhan menulisnya |
| `expect_docs` | Dokumen yang seharusnya muncul sebagai sitasi |
| `expect` | Kata kunci yang harus ada di jawaban yang benar |
| `role`, `clearance` | Konteks pengguna — supaya eval juga menguji RBAC |
| `should_abstain` | `true` untuk pertanyaan di luar korpus |

## Cara membuat eval set

**Ambil 30 pertanyaan nyata dari helpdesk atau grup chat internal.** Ini pekerjaan
non-coding yang bisa dan sebaiknya dimulai hari ini — tidak perlu menunggu kode siap, dan
nilainya jauh lebih tinggi daripada pertanyaan karangan.

Komposisi yang disarankan:

- 20 pertanyaan yang jawabannya ada di korpus
- 5 pertanyaan parafrase (kata-kata berbeda, dokumen sama) — menguji jalur vector
- 5 pertanyaan yang jawabannya **tidak ada** (`should_abstain: true`) — menguji kejujuran
- Beberapa pertanyaan mengarah ke dokumen `restricted` dengan `role: viewer` — menguji
  bahwa sistem menolak dengan benar

Lima pertanyaan abstain adalah bagian terpenting. Sistem RAG yang tidak pernah berkata
"tidak tahu" akan berhalusinasi, dan halusinasi pada dokumentasi internal lebih buruk
daripada tidak ada jawaban — pengguna akan mengikuti prosedur yang salah.

## Metrik

| Metrik | Target v1.0 | Arti |
| --- | --- | --- |
| `hit@5` | >= 0,85 | Dokumen yang benar ada di lima sitasi teratas |
| `citation_rate` | 1,00 | Tidak ada klaim tanpa sitasi |
| `abstain_precision` | >= 0,90 | Abstain saat memang tidak tahu |
| `rbac_leak` | 0 | Tidak pernah menyitasi dokumen di luar izin. **Tidak boleh gagal** |
| p95 token pertama | < 3 s | Terasa responsif |

## Cara pakai

```bash
pnpm eval                 # jalankan semua, cetak tabel metrik
pnpm eval --only=abstain  # jalankan subset
pnpm eval --compare=main  # bandingkan dengan baseline tersimpan
```

Aturan kerja:

- Jalankan sebelum dan sesudah setiap perubahan pada retrieval atau prompt. Simpan angkanya
  di deskripsi PR.
- `rbac_leak > 0` memblokir merge. Tanpa pengecualian.
- Setiap kali jawaban buruk ditemukan saat pemakaian nyata, **tambahkan ke eval set**.
  Harness ini harus tumbuh, bukan statis.
