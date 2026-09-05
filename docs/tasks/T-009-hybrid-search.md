# T-009: Hybrid search dan baseline retrieval

Pemilik A; prasyarat T-005/T-008 dan pipeline embed T-006b.

## Baca tambahan

Arsitektur pencarian, API search, eval README.

## Subtask

- T-009a: query keyword/vector dengan izin, RRF/deduplikasi, test integrasi kedua jalur.
- T-009b: endpoint/logging/fallback/pagination dan scripts/eval-retrieval.ts.

Indeks sudah disediakan T-003: periksa, jangan membuat migrasi indeks identik. Full-text memakai konfigurasi simple dan ts_rank_cd; PostgreSQL standar tidak menyediakan BM25. Vektor memakai embed() T-006b. Ambil 20 kandidat per jalur sesudah filter, RRF k=60, deduplikasi dokumen dan hasil default 10.

GET /api/search tidak memanggil chat(). Timeout/kegagalan embedding memberi fallback keyword, mode eksplisit dan log tepat satu per permintaan; bukan error kosong atau hasil global. Scope/RBAC tidak boleh dilewati saat fallback.

## Kriteria terima

- [ ] Kedua query menerapkan visibleDocumentsFilter, published/current-version dan kategori sebelum LIMIT; total tidak menghitung dokumen tersembunyi.
- [ ] Sorting/cursor stabil dan snippet teks aman. Highlight literal query dikerjakan UI T-010, bukan menerima HTML tak tersanitasi.
- [ ] Integrasi search-visibility.spec.ts menguji tiap jalur, RRF, fallback, perubahan kategori dan chunk versi lama.
- [ ] Tepat satu ai_query kind=search berisi mode, jumlah hasil, role, durasi/error; tidak menggandakan log lewat RAG.
- [ ] pnpm eval --retrieval mengukur hit@5 tanpa generasi dan melaporkan per kasus/model/tanggal.
- [ ] Gerbang lanjut integrasi sesuai scope; gagal berarti berhenti sebelum T-011.
- [ ] Ukur p95 per mode dan lampirkan EXPLAIN ANALYZE query nyata; jangan mengklaim target tercapai dari satu request.

Di luar scope: reranker, typo correction, filter tambahan, generation answer di /cari.
