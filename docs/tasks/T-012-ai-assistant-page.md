# T-012 — Halaman AI Assistant

Hari 5 · orang B · perkiraan sehari penuh

## Tujuan

Layar 9 mockup. Ini layar yang akan dilihat orang saat demo, jadi kerapiannya penting.

## Prasyarat

T-002. T-011 boleh paralel — sepakati bentuk event SSE di pagi hari, lalu kerjakan masing-masing.

## Baca dulu

`docs/ui-inventory.md` · `docs/mockup-alignment.md` bagian layar 9

## Berkas yang disentuh

`app/ai-assistant/page.tsx` · `components/chat/*` · `components/ui/CitationChip.tsx`

## Langkah

1. Tata letak tiga bagian: daftar percakapan di kiri, percakapan di tengah, kotak tulis di bawah.
2. Pil "Jawaban dibatasi hak akses Anda" di atas. Tombol "Percakapan baru".
3. Selektor "Ruang lingkup jawaban": seluruh knowledge base, kategori tertentu, dokumen yang sedang dibuka. Kalau waktu mepet, sisakan opsi pertama saja dan sembunyikan selektornya — jangan menampilkan pilihan yang tidak berfungsi.
4. Konsumsi SSE. Tampilkan blok "Sumber jawaban — N dokumen" dari event `sources` **sebelum** teks mulai mengalir. Ini bukan detail kosmetik: sumber yang muncul lebih dulu adalah alasan jawaban terasa kredibel.
5. Penanda sitasi `[1]` `[2]` di dalam teks, bisa diklik, menggulir ke kartu sumber yang sesuai.
6. Kartu sumber menampilkan judul dokumen, lokasi di dalamnya, versi, dan lencana Terverifikasi.
7. Aksi di bawah jawaban: Membantu (jempol atas/bawah) yang menulis ke `ai_query.feedback`, dan Salin tautan.
8. Kotak tulis menerima pertanyaan lanjutan dan mengirim `conversationId`.
9. Keadaan wajib: sedang berpikir, gagal, dan **abstain**. Tampilan abstain jangan terlihat seperti error — ia jawaban yang benar, dan cara menampilkannya menentukan apakah orang mempercayai sistemnya.

## Kriteria terima

- Bertanya lalu bertanya lanjutan berjalan mulus dalam satu percakapan.
- Sitasi bisa diklik dan mengarah ke dokumen yang benar.
- Jawaban abstain tampil rapi dan menyarankan langkah berikutnya.
- Terbaca di lebar 375 px.

## Test

Satu test komponen untuk perenderan sitasi. Sisanya manual — test UI di sini tidak sepadan biayanya.

## Di luar ruang lingkup

Ekspor jawaban, chip saran pertanyaan, lampirkan dokumen, bandingkan versi, berbagi percakapan.
