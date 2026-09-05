# T-011: Endpoint AI bersitasi dan riwayat aman

Pemilik A; prasyarat T-009 lulus gerbang integrasi, T-004/T-005, pipeline T-006b.

## Baca tambahan

API bagian AI, matriks riwayat, ADR-0010/0012.

## Subtask berurutan

- T-011a: ownership percakapan/pesan, izin ulang sumber, endpoints riwayat dan test lintas pengguna.
- T-011b: AiProvider.chat, kondensasi maksimal dua putaran, retrieval berscope, threshold/abstain dan spy prompt.
- T-011c: POST /api/ai/ask SSE, feedback/logging, integrasi end-to-end provider terkontrol.

Route hanya app/api/ai/ask/route.ts. Body/scope/meta/sources/token/done/error persis kontrak; runtime nodejs, uji batas durasi deployment. Jangan membuat alias /api/chat.

## Kriteria terima

- [ ] Awal percakapan mengembalikan conversationId melalui event meta sebelum sources/token; lanjutan memakai ID yang sama.
- [ ] User B tak dapat membaca/memakai conversationId/aiQueryId milik A; 404 generik.
- [ ] Perubahan role/cakupan, penonaktifan, sumber dihapus/diarsipkan, atau batas AI diperketat menolak riwayat sebelum kondensasi maupun respons. Tidak cukup menyembunyikan citation chip.
- [ ] Scope all/category/document mempersempit query; id dokumen tak berizin 404. Pertanyaan dokumen terbuka benar-benar memakai scope, bukan hanya menyalin judul ke prompt.
- [ ] Retrieval tidak mengembalikan draft/chunk versi lama; konteks selalu memakai identitas asli.
- [ ] Tanpa konteks cukup, generasi jawaban dilewati dan abstain dikirim. Kondensasi yang diperlukan sebelumnya tidak disebut nol panggilan LLM.
- [ ] Semua klaim faktual didukung sitasi; citation punya versi, headingPath, anchor/href tervalidasi.
- [ ] Histori memakai maksimal dua putaran; teks dokumen diperlakukan data, tidak instruksi.
- [ ] Error sebelum/di tengah stream ditangani tanpa done palsu; retry tidak menggandakan pesan user/requestId.
- [ ] ai_query kind=ask/log audit sumber ditulis; retrieval internal tidak membuat log search tambahan.

Tambahkan kasus ke tests/rbac/ai-retrieval-leak.spec.ts dan suite riwayat: periksa payload ke provider, bukan jawaban saja. Pengukuran mutu nyata milik T-013, bukan klaim dari stub provider.
