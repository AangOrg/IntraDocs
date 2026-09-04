## Apa yang berubah

<!-- Ringkas dalam 1-3 baris. Tautkan ID task, misal: T-005 -->

Task: T-

## Kenapa

<!-- Masalah yang diselesaikan. Sebutkan nomor ADR bila PR ini mengimplementasikannya. -->

## Cara menguji

<!-- Langkah konkret. Reviewer harus bisa mengikutinya tanpa bertanya. -->

1. 
2. 
3. 

## Checklist

- [ ] Acceptance criteria pada task spec sudah terpenuhi semuanya
- [ ] `pnpm typecheck && pnpm lint && pnpm test` hijau
- [ ] Ada test untuk logic berisiko (izin, parsing, scoring)
- [ ] Sudah diuji manual mengikuti langkah di atas
- [ ] Tidak ada `console.log`, kode mati, atau `TODO` yang tertinggal
- [ ] Tidak ada secret, kredensial, atau data internal asli di dalam diff
- [ ] Dokumen terkait diperbarui (ADR / context-pack / task board)

## Checklist RBAC & AI

<!-- Wajib diisi bila PR menyentuh izin, retrieval, atau AI. Hapus bila tidak relevan. -->

- [ ] Setiap jalur baca baru memakai `visibleDocumentsFilter(user)`
- [ ] Tidak ada logic izin di dalam prompt LLM
- [ ] Tidak ada penyaringan izin setelah query sebagai satu-satunya mekanisme
- [ ] `tests/rbac/` masih lulus, dan test baru ditambahkan untuk jalur baru
- [ ] Guard `AI_MAX_CLASSIFICATION` masih ditegakkan
- [ ] `pnpm eval` dijalankan; angka sebelum/sesudah dilampirkan di bawah

<!--
hit@5:            sebelum ___  sesudah ___
citation_rate:    sebelum ___  sesudah ___
abstain_precision: sebelum ___  sesudah ___
rbac_leak:        harus 0
-->

## Dampak performa

<!-- Isi bila menyentuh query, bundle, atau rendering. Sertakan EXPLAIN ANALYZE atau ukuran bundle. -->

## Catatan untuk reviewer

<!-- Bagian yang perlu perhatian khusus, keputusan yang masih ragu, atau utang teknis yang sengaja diambil. -->
