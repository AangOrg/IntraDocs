# T-013 — Evaluasi dan penyetelan

Hari 6 · orang A · perkiraan setengah hari

## Tujuan

Membuktikan dengan angka bahwa jawaban AI layak dipakai, bukan dengan kesan setelah mencoba tiga pertanyaan.

## Prasyarat

T-011 selesai dan korpus sudah final.

## Baca dulu

`docs/eval/README.md`

## Berkas yang disentuh

`docs/eval/questions.jsonl` · `scripts/eval.ts` · `package.json`

## Langkah

1. Tulis 10 pertanyaan dalam format `{"q":…,"expect_docs":[…],"expect":…}`. Komposisinya penting:
   - 6 pertanyaan yang jawabannya jelas ada di korpus
   - 2 pertanyaan yang jawabannya **tidak** ada — harapannya abstain
   - 1 pertanyaan yang menyentuh dokumen terbatas — dijalankan sebagai dua role, hasilnya harus berbeda
   - 1 pertanyaan lanjutan yang tidak mandiri
2. `pnpm eval` menjalankan semuanya dan melaporkan: hit@5, tingkat abstain pada yang seharusnya abstain, jumlah klaim tanpa sitasi, waktu rata-rata.
3. Setel bila di bawah ambang. Urutan penyetelan, dari yang paling murah:
   - ukuran potongan dan tumpang tindih
   - jumlah potongan yang dikirim ke prompt
   - bobot RRF antara kata kunci dan vektor
   - kalimat instruksi di prompt
   Ubah **satu** hal, jalankan ulang, catat hasilnya. Mengubah dua hal sekaligus membuat hasilnya tidak bisa dibaca.
4. Catat angka akhir di `docs/eval/README.md`.

## Kriteria terima

hit@5 ≥ 0,85 · abstain ≥ 90% pada pertanyaan di luar korpus · 0 klaim tanpa sitasi · pertanyaan berbeda role menghasilkan jawaban berbeda.

## Kalau angkanya tidak tercapai

Laporkan apa adanya beserta angka sebenarnya. **Jangan mengganti pertanyaan supaya lulus.** Set evaluasi yang disesuaikan agar lulus tidak mengukur apa pun, dan kebohongan itu akan ketahuan saat orang lain mencoba.

## Di luar ruang lingkup

Evaluasi otomatis di CI, penilaian oleh LLM, uji beban.
