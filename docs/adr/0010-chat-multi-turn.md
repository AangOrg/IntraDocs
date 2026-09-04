# ADR-0010: Percakapan berlanjut pada AI Assistant

- Status: Diterima
- Tanggal: 4 September 2026
- Memperjelas: ADR-0004 dan keputusan "RAG satu langkah"

## Konteks

Layar 9 memperlihatkan AI Assistant sebagai halaman penuh dengan riwayat percakapan di sisi kiri, dan percakapan contohnya berisi pertanyaan lanjutan yang tidak berdiri sendiri: "Kalau perangkat authenticator-nya hilang bagaimana?" Pertanyaan itu tidak dapat dipahami tanpa pertanyaan sebelumnya tentang reset password, dan retrieval atas teks itu apa adanya akan mengembalikan dokumen yang salah.

Rencana sebelumnya menyebut "RAG satu langkah" dan menempatkan halaman chat terpisah pada urutan potong nomor dua. Dua-duanya salah kalibrasi. Halaman chat adalah layar inti produk, dan pertanyaan lanjutan hampir pasti ditanyakan saat demo karena begitulah orang bertanya.

## Keputusan

Tetap satu panggilan retrieval per pertanyaan, tetapi pertanyaan dinormalkan lebih dulu bila ada riwayat.

1. Bila percakapan punya riwayat, satu panggilan LLM pendek menulis ulang pertanyaan lanjutan menjadi pertanyaan mandiri.
2. Pertanyaan mandiri itulah yang masuk ke hybrid search.
3. Konteks yang dikirim ke LLM penjawab berisi potongan dokumen ditambah maksimal dua putaran percakapan terakhir.

Yang tetap tidak dilakukan: tool calling, retrieval berulang, dan agent yang memutuskan sendiri kapan mencari lagi. Alurnya tetap deterministik untuk pertanyaan yang sama, sehingga tetap dapat dievaluasi. Inilah alasan penolakan agentic RAG yang sudah dicatat sebelumnya, dan alasan itu tidak berubah.

## Konsekuensi

- Halaman chat keluar dari daftar potong. Yang boleh dipotong adalah penyimpanan riwayat, bukan chat-nya.
- Riwayat butuh dua tabel kecil, `conversation` dan `message`, masuk ke skema hari 1. Skema MVP menjadi 11 tabel.
- Latensi bertambah satu panggilan LLM pendek, hanya pada pertanyaan lanjutan, tidak pada pertanyaan pertama.
- Penulisan ulang pertanyaan **tidak** boleh melewati filter izin. Retrieval tetap memakai `visibleDocumentsFilter` atas identitas penanya, apa pun bentuk pertanyaan hasil penulisan ulang.
- Eval tetap dijalankan tanpa riwayat, jadi angkanya tetap dapat diperbandingkan antar hari.

## Cara membatalkan

Lewati langkah penulisan ulang dan kirim pertanyaan apa adanya ke retrieval. Kualitas jawaban pada pertanyaan lanjutan turun; tidak ada yang rusak.
