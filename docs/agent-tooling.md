# Alat agent AI — untuk mengerjakan proyek, dan di dalam produk

Dua hal yang sering tertukar, dan konsekuensinya berbeda jauh:

- **Bagian A** — alat AI yang kita pakai **untuk membangun** IntraDocs.
- **Bagian B** — apakah **produknya sendiri** perlu tool calling dan agent.

Jawaban untuk keduanya tidak sama. Bagian A: pakai secukupnya, dan itu sangat membantu.
Bagian B: jangan, belum.

---

# Bagian A — Alat untuk membangun

## Yang dipakai

| Alat | Fungsi | Biaya setup |
| --- | --- | --- |
| `AGENTS.md` + `CLAUDE.md` | Aturan repo yang otomatis terbaca agent | Sudah ada |
| `docs/context-pack.md` | Konteks proyek yang stabil, ditempel ke chat baru | Sudah ada |
| `docs/tasks/T-00X.md` | Spec per task: acceptance criteria + batasan | Sudah ada |
| `/task` slash command | Memuat konteks + task lalu mulai bekerja, tanpa menjelaskan ulang | 5 menit |
| `/review` slash command | Review diff terhadap checklist RBAC dan `AGENTS.md` | 5 menit |
| Subagent `reviewer` | Review dengan konteks terpisah, sehingga tidak membela kodenya sendiri | 10 menit |
| Copilot code review pada PR | Mata kedua yang gratis untuk hal-hal mekanis | Sekali klik |

Total setup: kurang dari setengah jam, dan berlaku untuk seluruh sprint.

**Kenapa subagent `reviewer` layak dibuat padahal ada `/review`:** model yang baru saja
menulis sebuah kode adalah reviewer terburuk untuk kode itu. Ia sudah yakin kodenya benar.
Subagent berjalan dengan konteks bersih dan tidak punya keterikatan pada keputusan
sebelumnya, jadi lebih sering menemukan masalah nyata.

## Yang sengaja tidak dipakai

| Tidak dipakai | Alasan |
| --- | --- |
| Orkestrasi multi-agent (agent yang mengatur agent) | Menambah kegagalan yang tidak bisa didebug. Dua orang bisa mengoordinasi diri sendiri |
| Sepuluh subagent berperan | Perawatan prompt melebihi manfaatnya. Dua sudah cukup |
| MCP server kustom untuk pengembangan | Slash command dan berkas repo sudah menutup kebutuhannya |
| Hook auto-commit atau auto-push | Menghapus satu-satunya titik di mana manusia melihat perubahan |
| Agent yang boleh merge PR | Tidak, dalam keadaan apa pun |
| Test yang ditulis agent tanpa dibaca | Test yang tidak dibaca hanya mengunci bug yang sudah ada |

## Pembagian model — supaya usage tidak jebol

Aturan ekonomi yang sederhana: **model termurah yang cukup untuk pekerjaan itu.**

| Pekerjaan | Model | Alasan |
| --- | --- | --- |
| Boilerplate, komponen dari mockup, seed, penamaan | Kelas murah/cepat | Ada spec dan ada mockup; keputusannya sedikit |
| Menulis 20–25 dokumen sintetis | Kelas murah/cepat | Volume tinggi, risiko rendah |
| Filter RBAC, alur retrieval, kontrak jawaban AI | Kelas atas | Salah di sini mahal dan sulit terlihat |
| Review diff sebelum merge | Kelas atas | Justru di sinilah nilai model terbaik paling terasa |
| Debug yang sudah dua kali gagal | Kelas atas, konteks bersih | Melanjutkan percakapan yang sudah bingung akan mengulang kebingungan |

Empat kebiasaan yang menghemat usage lebih banyak daripada memilih model:

1. **Mulai chat baru per task.** Konteks panjang membuat model lebih lambat, lebih mahal, dan
   lebih mudah lupa aturan awal.
2. **Tempel spec, jangan jelaskan.** Menjelaskan ulang proyek setiap kali adalah pemborosan
   terbesar yang paling tidak disadari.
3. **Kalau dua kali percobaan gagal, jangan percobaan ketiga di chat yang sama.** Mulai
   bersih dengan potongan masalah yang lebih kecil.
4. **Baca diff-nya.** Ini bukan soal usage, tapi soal apakah hasilnya bisa dipertahankan.

## Skill opsional yang layak dibuat, satu saja

Di hari 3, **penulis dokumen sintetis**: satu instruksi yang memuat gaya rumah — frontmatter,
struktur bernomor seperti mockup s4, tingkat formalitas dokumen internal, penomoran SOP.
Alasannya bukan kecepatan, tetapi **konsistensi**: eval hanya bermakna kalau korpusnya
konsisten. Dua puluh lima dokumen yang gayanya berbeda-beda akan mengacaukan pengukuran
retrieval.

Selain itu, tidak perlu membangun pustaka skill. Untuk sprint enam hari, itu perkakas yang
lebih banyak dirawat daripada dipakai.

---

# Bagian B — Apakah produknya perlu agent dan tool calling?

## Keputusan: MVP memakai RAG satu langkah, bukan agent

Alur MVP tetap: pertanyaan → retrieval hybrid → satu panggilan LLM dengan konteks → jawaban
bersitasi. Tanpa loop, tanpa tool calling, tanpa perencanaan multi-langkah.

Lima alasan, berurutan dari yang paling menentukan:

1. **Bisa dievaluasi.** RAG satu langkah bersifat deterministik untuk pertanyaan yang sama,
   jadi `pnpm eval` mengukur perubahan nyata. Agent yang mengambil jalur berbeda setiap kali
   dijalankan membuat angka eval kehilangan arti — dan tanpa angka, tidak ada cara membuktikan
   kualitas.
2. **Latensi.** Satu langkah menghasilkan token pertama dalam hitungan detik. Loop agent
   dengan tiga sampai lima panggilan mudah menjadi dua puluh detik atau lebih. Untuk pertanyaan
   internal sehari-hari, itu selisih antara dipakai dan ditinggalkan.
3. **Sitasi bisa diverifikasi.** Kalau konteksnya satu himpunan chunk yang tetap, setiap klaim
   bisa dilacak ke sumbernya. Kalau modelnya mengambil ulang beberapa kali, hubungan antara
   jawaban dan sumber menjadi kabur — dan sitasi yang tidak bisa diverifikasi lebih buruk
   daripada tanpa sitasi, karena tampak kredibel tanpa benar-benar kredibel.
4. **Permukaan izin.** Setiap tool tambahan adalah satu jalur lagi yang harus dibuktikan tidak
   bocor. Satu jalur retrieval berarti satu tempat yang perlu diuji.
5. **Biaya.** Tiga sampai lima kali panggilan per pertanyaan, untuk manfaat yang belum
   terbukti dibutuhkan.

## Kapan tool calling layak ditambahkan

Setelah eval menunjukkan **kegagalan yang bentuknya spesifik**, bukan karena arsitekturnya
terdengar lebih canggih. Dua pemicu yang jelas:

- **Pertanyaan multi-hop gagal** ("bandingkan SLA gangguan jaringan dengan SLA insiden
keamanan") — satu retrieval tidak cukup karena jawabannya ada di dua dokumen. Tambahkan tool
`search_documents` yang boleh dipanggil paling banyak dua kali, dengan tetap memakai
`visibleDocumentsFilter` yang sama.
- **Pertanyaan sangat pendek atau penuh singkatan gagal** — tambahkan satu langkah penulisan
ulang kueri. Ini lebih murah daripada agent penuh dan biasanya sudah cukup.

Keduanya aditif. Tidak ada yang perlu dibongkar untuk menambahkannya nanti, dan itulah
sebabnya menundanya aman.

## Satu ide fase 3 yang murah dan berdampak besar

Ekspos IntraDocs sebagai **MCP server**, sehingga pegawai bisa menanyakan dokumentasi internal
langsung dari Claude, Cursor, atau editor mereka tanpa membuka web-nya.

Secara teknis kecil: satu endpoint yang memakai kembali `visibleDocumentsFilter` dan alur
retrieval yang sudah ada. Yang sulit bukan retrieval-nya, tetapi **autentikasinya** — klien
MCP perlu identitas pengguna supaya filter izin tetap berlaku, dan token statis akan
melanggar seluruh model RBAC yang kita bangun.

Jangan dikerjakan sebelum RBAC terbukti kokoh di web. Tapi layak disebutkan saat demo sebagai
arah lanjutan, karena inilah yang membedakannya dari sekadar situs dokumentasi.
