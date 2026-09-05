# Eval harness — metode, bukan angka pemasaran

Scope adalah sumber ambang mutu. Baseline kecil membuktikan regresi pada korpus sintetis, bukan akurasi umum model. Tidak mengambil pertanyaan helpdesk/chat internal atau data perusahaan sungguhan.

## Data

T-008 membuat docs/eval/questions.jsonl: tepat 10 kasus sintetis, 8 answerable (termasuk 2 parafrase) dan 2 outside-corpus. questions.example.jsonl hanya contoh format, bukan hasil eval atau korpus final.

Field: id unik, q, actor (nama akun seed aktif), expect_docs (slug dokumen yang diizinkan), should_abstain boolean, expected_points (array pokok jawaban). Tidak ada clearance dari input; harness memuat aktor/role/scope terkini dari fixture DB. Fajar bukan aktor query berhasil.

Semua expect_docs harus resolve ke dokumen fixture yang tepat. Untuk abstain daftar kosong. Jangan mengganti pertanyaan/ground truth agar model lulus. Tambahan kasus gagal disimpan sebagai suite regresi tambahan dan dilaporkan terpisah dari baseline beku.

## Dua lapisan

- T-009: pnpm eval --retrieval → scripts/eval-retrieval.ts, retrieval nyata tanpa generasi; boleh memanggil embedding. Laporkan top-5 slug dan hit setiap kasus.
- T-013: pnpm eval → scripts/eval.ts, jawaban nyata + pemeriksaan manusia terhadap dukungan klaim/sitasi/abstain. Opsi --retrieval tetap memilih mode sebelumnya.
- Keamanan: pnpm test mencakup query/guard serta integrasi endpoint, spy payload provider, ownership, perubahan izin, sesi nonaktif, dan multi-turn. Tidak bergantung pada model kebetulan tidak mengucapkan rahasia.

## Definisi pengukuran

| Metrik | Pembilang / penyebut atau metode |
| --- | --- |
| hit@5 | Kasus answerable dengan setidaknya satu expect_doc di top-5 / 8 |
| Abstain recall | Kasus wajib abstain yang benar-benar abstain / 2 |
| Unsupported claims | Jumlah klaim faktual tanpa sitasi yang benar-benar mendukungnya; review manusia |
| RBAC leak | Jumlah kasus yang membocorkan judul/snippet/teks/metadata/riwayat ke respons atau provider |
| p95 search/first token | Distribusi durasi endpoint dan waktu menuju token pertama, bukan waktu sources/meta |

Dengan denominator kecil, satu kegagalan sangat berarti: hit@5 bergerak 1/8; abstain 1/2. Ini bukan confidence interval akurasi produksi. Laporkan false abstain pada kasus answerable juga, bukan hanya keberhasilan abstain.

Latency: minimal 30 request terukur per endpoint, campuran pertanyaan tetap; pisahkan cold/warm, first/follow-up, hybrid/keyword. Laporkan n, error/timeout, lingkungan/region, model dan commit. Jangan menghapus request lambat diam-diam. Tidak ada klaim p95 dari satu panggilan. Abstain tanpa token dicatat terpisah sebagai latency respons, bukan token pertama cepat palsu.

## Gerbang dan bukti

Ambang integrasi/rilis ada di docs/scope-mvp.md. Gagal integrasi menghentikan T-011; lolos integrasi belum berarti lolos rilis. Satu kebocoran memblokir merge/rilis yang terdampak.

Lampirkan hasil per kasus, denominator, model, tanggal, commit, biaya/panggilan bila diketahui, serta penilaian sitasi manusia di artefak PR. Jangan menimpa dokumen metode ini dengan angka tanpa provenance.

Tidak menjalankan generasi berbayar pada setiap CI. Jalankan ketika retrieval/prompt/provider berubah atau saat gerbang rilis; perubahan UI murni tidak perlu eval LLM penuh. Satu parameter diubah per eksperimen.
