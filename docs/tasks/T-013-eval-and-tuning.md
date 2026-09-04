# T-013 — Evaluasi jawaban dan penyetelan

Hari 6 · orang A · perkiraan setengah hari

## Tujuan

Membuktikan dengan angka bahwa jawaban AI layak dipakai, bukan dengan kesan setelah mencoba tiga pertanyaan.

## Hubungannya dengan hari 4

Mutu retrieval sudah diukur di T-009 hari 4 lewat `scripts/eval-retrieval.ts`. Task ini mengukur lapisan di atasnya: apakah jawaban yang disusun dari dokumen yang benar juga benar, bersitasi, dan tahu kapan harus diam.

Kalau hit@5 hari 4 sudah memenuhi ambang, kegagalan di sini hampir selalu soal prompt atau jumlah potongan — bukan soal retrieval. Pemisahan ini yang membuat penyetelan bisa terarah.

## Prasyarat

T-011 selesai dan korpus sudah final.

## Baca dulu

`docs/eval/README.md`

## Berkas yang disentuh

`docs/eval/questions.jsonl` · `scripts/eval.ts` · `package.json`

## Langkah

1. Set 10 pertanyaan sudah ditulis bersamaan dengan korpus di T-008. Komposisinya:
   - 6 pertanyaan yang jawabannya jelas ada di korpus
   - 2 pertanyaan yang jawabannya **tidak** ada — harapannya abstain
   - 1 pertanyaan yang menyentuh dokumen terbatas — dijalankan sebagai dua role, hasilnya harus berbeda
   - 1 pertanyaan lanjutan yang tidak mandiri
2. `pnpm eval` menjalankan semuanya dan melaporkan: hit@5, tingkat abstain pada yang seharusnya abstain, jumlah klaim tanpa sitasi, waktu rata-rata.
3. Setel bila di bawah ambang. Urutan penyetelan, dari yang paling murah:
   - jumlah potongan yang dikirim ke prompt
   - kalimat instruksi di prompt
   - ambang relevansi untuk abstain
   - ukuran potongan dan tumpang tindih (mahal — butuh reindeks)
   Ubah **satu** hal, jalankan ulang, catat hasilnya. Mengubah dua hal sekaligus membuat hasilnya tidak bisa dibaca.
4. Catat angka akhir di `docs/eval/README.md`.

## Kriteria terima

hit@5 ≥ 0,85 · abstain ≥ 90% pada pertanyaan di luar korpus · 0 klaim tanpa sitasi · pertanyaan berbeda role menghasilkan jawaban berbeda.

## Kalau angkanya tidak tercapai

Laporkan apa adanya beserta angka sebenarnya. **Jangan mengganti pertanyaan supaya lulus.** Set evaluasi yang disesuaikan agar lulus tidak mengukur apa pun, dan kebohongan itu akan ketahuan saat orang lain mencoba.

## Di luar ruang lingkup

Evaluasi otomatis di CI, penilaian oleh LLM, uji beban.
