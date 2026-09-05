# Kesesuaian mockup — keputusan aktif 11 layar

Sumber intradocs-mockup_1.html, pemeriksaan layar 1–11 pada 4–5 September 2026. Riwayat diskusi ada di PR; berkas ini menyimpan hasil aktif, bukan menyalin kronologi. Rute aplikasi mengikuti ui-inventory (mockup /help-center dan /search dipetakan ke / dan /cari). Scope > ADR > context-pack > matriks ini.

## Layar 1 — Halaman Muka

Masuk: hero pencarian, tombol Tanya AI ke /ai-assistant, enam kartu kategori, jumlah dokumen sesudah izin, daftar published terbaru, banner AI dan footer Internal. Tidak ada chip topik populer atau paling banyak dibaca: log kueri belum merupakan statistik pemakaian yang matang dan document_view tidak dibuat. Jangan mengisi angka karangan.

## Layar 2 — Hasil Pencarian

Masuk: judul/snippet, highlight teks aman, kategori, pagination/total/durasi hasil nyata dan Lanjutkan di chat membawa q. Tidak ada kotak .ai-answer atau dokumen berjudul terkunci. Pil Terbatas — akses via permintaan tidak dibangun menurut ADR-0011: dokumen di luar izin tidak muncul sama sekali, termasuk hitungan.

Keyword/vector search boleh memakai embed(), tidak chat(); fallback keyword saat embedding gagal. Ini tidak secara teknis melarang komponen AI di UI yang sama, tetapi MVP sengaja memilih satu halaman jawaban untuk menghindari pekerjaan ganda. Filter hanya kategori; format/label/status bukan fitur wajib MVP.

## Layar 3 — Katalog

Masuk: tabel metadata dokumen, filter kategori/cursor di URL, versi current, updatedAt dan klasifikasi. Terbatas adalah klasifikasi, Kedaluwarsa nilai turunan; bukan status workflow. Draft hanya terlihat melalui aturan khusus matriks. Tidak ada grid toggle, favorit, riwayat baca, ukuran berkas asli, jumlah halaman PDF, popularitas atau rollback.

## Layar 4 — Pembaca Dokumen

Masuk: DocNav hierarkis, isi Markdown tersanitasi, daftar isi/anchor, metadata pemilik/versi/kategori/klasifikasi, peringatan kedaluwarsa, Tanya AI tentang halaman ini dengan scope document. Heading_path, nomor versi, dan anchor adalah field berbeda, tidak saling menggantikan.

Tidak ada UI riwayat/compare/rollback, favorit, unduh berkas asli, pengingat terjadwal, atau feedback dokumen. Sitasi versi lama yang bukan current ditandai sumber berubah sesuai kontrak, tidak menampilkan versi terbaru seolah versi yang dirujuk. Jejak approval tidak dipalsukan.

## Layar 5 — Unggah Knowledge

MVP tetap menerima md/txt melalui pembacaan teks sederhana dan satu form/pratinjau, bukan seed-only. Judul, kategori, label, klasifikasi, pemilik sesuai aksi, periode tinjau ulang dan editor masuk T-006.

Wizard empat langkah, PDF/DOC/DOCX/XLSX/PPTX/HTML/OCR/ZIP, batas 50 MB, subkategori, usulan metadata AI dan deteksi dokumen serupa ditunda. Tidak membuat tabel file/job. Batas teks MVP di kontrak berbeda dari batas berkas mockup dan dijelaskan sebagai penyederhanaan, bukan kesetaraan penuh.

## Layar 6 — Approval Admin

Seluruh UI/alur approval Fase 2: antrean dua tahap, approve/request_changes/reject, @mention, notifikasi dan perbandingan versi.

Pemindaian pola sensitif tetap MVP, dipindahkan ke publikasi langsung T-006b. Tidak mengklaim regex mendeteksi semua rahasia. Tumpang tindih 38%/analisis AI template tidak dibangun. Hanya published yang diindeks; kegagalan embedding tidak menerbitkan state setengah jadi.

## Layar 7 — Kategori & Label

UI admin ditunda. Enam kategori satu tingkat dan label melalui seed; parent_id dipersiapkan. Tidak ada drag/drop taksonomi, 42 kategori, 68 label, label otomatis, atau mesin aturan.

Aturan kategori Keamanan → Security/Admin pada mockup bukan model izin tambahan yang otomatis tersedia. MVP memakai klasifikasi dan scope pengguna menurut ADR-0012; tidak mengklaim semantik aturan kategori arbitrer sepenuhnya sama. Kedaluwarsa dihitung dari tanggal/periode, bukan mesin pengubah enum.

## Layar 8 — Pengguna & RBAC

Lima role mockup dipetakan ke empat: Super Admin/Admin Knowledge digabung admin, sisanya reviewer/contributor/viewer. UI pengelolaan/invite/AD/role kustom ditunda; model izin tetap dibangun.

Maya Lestari di mockup adalah Admin Knowledge bercakupan sempit. MVP admin global tidak merepresentasikan pembatasan itu: ini deviasi sadar, bukan kemampuan tersembunyi lewat category_scope. Model role-only tanpa clearance ditetapkan ADR-0012; bukan karena ADR-0004 melarang banyak atribut.

Fajar viewer Finance (mitra) nonaktif mengikuti mockup, cakupan SOP & Proses dipetakan ke SOP & Proses Bisnis. Viewer Demo adalah akun sintetis aktif tambahan. Enam akun total, sumber identitas tunggal matriks RBAC; varian nonaktif role lain hanya fixture test sementara.

## Layar 9 — AI Assistant

Masuk: halaman penuh /ai-assistant, riwayat pribadi, multi-turn, scope all/category/document, sumber sebelum token, sitasi berversi/heading/tautan, feedback jawaban, latency nyata, pesan batas hak akses, Percakapan baru. Route tunggal POST /api/ai/ask.

Meta mengembalikan conversationId; scope berubah memulai percakapan baru. Riwayat kehilangan izin ditolak sebelum kondensasi/display. Nomor halaman/Sheet Excel tidak didukung md/txt. Ekspor jawaban, chip saran, lampiran, dan berbagi percakapan ditunda.

## Layar 10 — Dashboard

UI ditunda; data sejak endpoint tersedia: ai_query untuk setiap search/ask, mode/latency/result_count/abstain/error/role; audit_log untuk akses sensitif. Topik populer berasal dari query, paling banyak dibaca memerlukan log pembacaan tersendiri—bukan metrik yang sama. Waktu approval belum dapat dihitung karena alurnya belum ada.

## Layar 11 — Arsitektur dan Roadmap

Next.js menyatukan portal/API/dokumen/AI. Postgres menyatukan metadata/full-text/pgvector; tidak ada gateway/vector DB/search engine terpisah. OIDC, object storage, queue, OCR, notifikasi, PWA/offline dan konsol admin ditunda. Mobile web responsif tetap wajib.

Alur yang dibuktikan: input teks → validasi/scan → publish/index → baca/search/AI → log. Approval/pengayaan AI/pengingat otomatis belum dipenuhi. Model provider lokal adalah jalur portabilitas, bukan klaim deployment perusahaan sudah ada. Semua demo memakai dokumen sintetis.

## Kejujuran demo

Angka 1.284 dokumen/318 pengguna/96% akurasi adalah placeholder; bahkan jumlah per role 312 berbeda dari header, dan total dokumen dicampur dengan terverifikasi. Jangan menyalinnya sebagai fakta.

Sebut deviasi: empat role/global admin, tidak ada judul terkunci, satu halaman AI, input hanya md/txt, tanpa approval/SSO/dashboard/favorit/popularitas. Penyederhanaan bukan persetujuan mentor otomatis; gunakan matriks ini untuk membahas requirement yang masih pasca-MVP.

Rilis mengikuti gerbang scope, bukan jadwal enam hari. Tidak ada fitur wajib yang dipotong hanya karena ditulis sebagai kandidat pemotongan pada riwayat lama.
