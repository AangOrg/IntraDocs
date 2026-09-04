# T-011 — Endpoint RAG

Hari 5 · orang A · perkiraan sehari penuh

Ini task paling berisiko di seluruh sprint. Kalau ada yang meleset, meleset di sini.

## Tujuan

Endpoint chat yang menjawab dalam bahasa Indonesia, selalu menyertakan sumber, dan mengatakan tidak tahu ketika memang tidak ada jawabannya.

## Prasyarat

T-009 selesai. Retrieval dipakai ulang, tidak ditulis ulang.

## Baca dulu

ADR-0010 · ADR-0003 · `docs/api-contract.md`

## Berkas yang disentuh

`app/api/chat/route.ts` · `lib/ai/rag.ts` · `lib/ai/prompt.ts` · `lib/ai/condense.ts` · `tests/rbac/ai-retrieval-leak.spec.ts` (sudah ada, tambahkan kasus)

## Langkah

1. `POST /api/chat` menerima `conversationId` opsional, `question`, dan `scope` opsional. `runtime = 'nodejs'`, `maxDuration = 60`.
2. **Kondensasi.** Kalau percakapan sudah punya giliran sebelumnya, panggil LLM sekali untuk menulis ulang pertanyaan jadi mandiri. Contoh nyata dari mockup: "Kalau perangkat authenticator-nya hilang bagaimana?" harus jadi pertanyaan yang berdiri sendiri sebelum dicari. Batasi ke 3 giliran terakhir. Kalau tidak ada giliran sebelumnya, lewati langkah ini seluruhnya — jangan buang satu panggilan LLM untuk pertanyaan pertama.
3. Ambil potongan lewat retrieval T-009, dibatasi `visibleDocumentsFilter` dan `AI_MAX_CLASSIFICATION`.
4. Kalau potongan teratas di bawah ambang relevansi, **jangan panggil LLM sama sekali**. Kirim jawaban abstain yang sudah ditentukan. Ini menghemat biaya sekaligus menutup jalur mengarang yang paling umum.
5. Susun prompt: instruksi bahasa Indonesia, potongan bernomor beserta judul dokumen dan `heading_path`, aturan bahwa setiap klaim wajib memakai penanda `[n]`, dan aturan bahwa kalau konteks tidak memuat jawabannya maka jawab tidak menemukan.
6. Alirkan SSE dengan urutan `sources` → `token` → `done`, dan `error` bila gagal. `sources` dikirim **lebih dulu** supaya UI bisa menampilkan sumber sebelum teks selesai.
7. Simpan ke `conversation` dan `message`. Kalau waktu habis, bagian ini yang pertama dipotong — endpoint tetap berfungsi tanpa penyimpanan.
8. Catat `ai_query`: pertanyaan, jumlah dokumen dirujuk, menjawab atau abstain, role.

## Kriteria terima

- Pertanyaan yang jawabannya ada di korpus menghasilkan jawaban dengan minimal satu sitasi yang benar-benar mengarah ke dokumen yang dipakai.
- Pertanyaan di luar korpus menghasilkan abstain, bukan karangan.
- Pertanyaan lanjutan yang tidak mandiri tetap mengambil dokumen yang benar.
- Viewer dan admin mendapat jawaban berbeda untuk pertanyaan yang menyentuh dokumen terbatas.
- Token pertama muncul di bawah 3 detik.

## Test

Tambahkan ke `tests/rbac/ai-retrieval-leak.spec.ts`: isi dokumen terbatas tidak pernah muncul di potongan yang dikirim ke LLM untuk pengguna yang tidak berhak. **Periksa muatan prompt-nya, bukan hanya teks jawabannya.** Jawaban yang kebetulan tidak menyebut isi rahasia tetap kebocoran kalau isinya sempat masuk prompt.

## Di luar ruang lingkup

Tool calling, agent, multi-hop retrieval, reranker, caching jawaban, streaming sitasi per token.
