# AGENTS.md — aturan kerja di repo ini

Berkas ini yang pertama dibaca setiap sesi. Pendek dengan sengaja.

## Jangan baca seluruh folder docs/

Repo ini punya banyak dokumen perencanaan. **Membacanya semua akan merusak kualitas kerjamu**, bukan meningkatkannya. Setiap task sudah punya daftar bacaan sendiri.

Urutan baca per sesi — maksimal empat berkas:

1. `docs/STATUS.md` — hari ini hari ke berapa, apa yang sudah jadi
2. `AGENTS.md` — berkas ini
3. `docs/context-pack.md` — fakta stabil, nama identifier, batasan
4. `docs/tasks/T-0XX-*.md` — spec task yang sedang dikerjakan

Spec task akan menyebut sendiri ADR atau dokumen tambahan yang perlu dibuka. Buka yang disebut saja. Daftar lengkapnya di `docs/eksekusi.md`.

**Jangan pernah membuka `intradocs-mockup_1.html` secara utuh.** Berkasnya ~2.010 baris dan akan memakan sebagian besar konteksmu. Baca per rentang baris.

## Kalau dokumen saling bertentangan

Urutan kekuatan: `docs/scope-mvp.md` > ADR bernomor > `docs/context-pack.md` > sisanya. Kalau menemukan pertentangan, ikuti yang lebih kuat dan **tulis catatan di deskripsi PR**. Jangan memperbaiki dokumen lain di tengah task kode.

## Satu task, satu branch, satu PR

- Branch: `feat/T-0XX-slug`. Tidak pernah menulis ke `main`.
- Selesaikan satu task, buka PR, berhenti. **Jangan lanjut ke task berikutnya di sesi yang sama** walaupun terasa masih sanggup.
- PR di bawah 400 baris perubahan. Kalau lebih besar, task-nya salah dipecah — laporkan, jangan diteruskan.
- Jangan merge PR dan jangan menghapus branch tanpa diminta eksplisit.

## Keputusan sudah diambil

ADR di `docs/adr/` adalah keputusan tertutup. Kalau tidak setuju, tulis alasannya di deskripsi PR dan tetap ikuti ADR-nya. Jangan mengganti pustaka, menambah dependensi, atau mengubah arah teknis di tengah task. Menambah dependensi baru butuh persetujuan pemilik repo.

## Konvensi kode

- TypeScript strict. Tidak ada `any`, tidak ada `@ts-ignore`.
- Identifier, komentar kode, dan pesan commit: **bahasa Inggris**. Teks yang dilihat pengguna: **bahasa Indonesia**.
- Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`.
- Semua route handler `export const runtime = 'nodejs'`.
- Query basis data lewat Drizzle. Migrasi lewat `drizzle-kit generate` lalu commit SQL-nya. **Jangan pernah `drizzle-kit push` ke basis data produksi.**
- Tidak ada penulisan filesystem lokal, tidak ada SDK spesifik vendor, tidak ada Edge runtime.

## Aturan keamanan yang tidak bisa ditawar

1. Setiap pengambilan dokumen memanggil `visibleDocumentsFilter(user)`. Tidak ada jalur kedua. Filter izin tidak pernah ditaruh di prompt LLM.
2. Dokumen yang tidak boleh dilihat mengembalikan **404, bukan 403**.
3. Semua isi dokumen bersifat sintetis dan fiktif. Jangan pernah memasukkan data Telkom asli, kredensial, atau data pribadi.
4. Markdown selalu dibersihkan lewat `rehype-sanitize` sebelum dirender.

## Selesai berarti

Sebuah task selesai kalau semuanya benar:

- `pnpm typecheck && pnpm lint && pnpm test && pnpm build` hijau di lokal
- Kriteria terima di spec task terpenuhi, satu per satu
- Kalau task menyentuh jalur baca dokumen: ada test yang membuktikan izin tetap rapat
- `docs/STATUS.md` diperbarui di PR yang sama
- PR dibuka dengan templat yang ada, berisi ringkasan dan cara mengujinya

## Berhenti dan bertanya kalau

Spec bertentangan dengan kode yang sudah ada · task butuh dependensi baru · task ternyata lebih dari 400 baris · butuh kredensial yang tidak ada · harus menyentuh berkas di luar yang disebut spec.
