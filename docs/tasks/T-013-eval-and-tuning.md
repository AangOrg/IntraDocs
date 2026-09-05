# T-013: Evaluasi jawaban dan gerbang rilis

Pemilik A; prasyarat T-011/T-012 terintegrasi dan korpus T-008 dibekukan.

## Baca tambahan

`docs/eval/README.md` untuk metode; `docs/scope-mvp.md` untuk angka.

## Lingkup

T-009 mengukur retrieval tanpa generasi. Task ini menambahkan scripts/eval.ts dan pnpm eval untuk generasi, dukungan sitasi, abstain, dan latensi nyata. pnpm eval --retrieval tetap tersedia.

Sepuluh baseline = delapan answerable termasuk dua parafrase + dua outside-corpus. RBAC lintas role, ownership, perubahan izin, dan multi-turn adalah suite integrasi terpisah; jangan menghitung jawaban lanjutan tanpa riwayat sebagai kegagalan retrieval baseline.

## Kriteria terima

- [ ] Hit@5, abstain recall, klaim tanpa dukungan sitasi, rbac_leak, dan p95 dilaporkan dengan denominator/metode/model/commit/lingkungan.
- [ ] Semua gerbang rilis scope lulus; gerbang lanjut integrasi bukan izin rilis.
- [ ] Rubrik sitasi diperiksa manusia per klaim; keberadaan token [1] saja bukan bukti sumber mendukung.
- [ ] Pengukuran latency menyebut jumlah sampel, mode, error, cold/warm; tidak mengklaim generalisasi dari sepuluh pertanyaan.
- [ ] Ubah satu parameter, ukur sebelum/sesudah; jangan mengganti baseline supaya lulus.
- [ ] Bukti hasil dicatat sebagai artefak/PR, bukan menimpa metode evaluasi dengan angka tanpa tanggal.

Jika gagal: laporkan, perbaiki, ulangi; tanggal bergeser. Tidak ada LLM judge atau evaluasi provider berbayar otomatis pada tiap CI. Pecah harness dan tuning menjadi subtask bila melewati batas PR.
