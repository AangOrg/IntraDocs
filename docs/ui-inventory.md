# Inventaris UI

Mockup `intradocs-mockup_1.html` adalah **spesifikasi visual**, bukan wireframe: taksonomi,
klasifikasi, dan alur approval sudah dipikirkan di dalamnya. Jangan merancang ulang dari nol.

## Langkah pertama sebelum menulis komponen

Mockup mendefinisikan design system lengkap sebagai CSS variable (`--blue-600`, `--ink`,
`--muted`, `--line`, `--r`, `--sh-md`, dan lain-lain). **Ekstrak semuanya menjadi design token
di `tailwind.config.ts` sebagai task T-002, sebelum komponen apa pun dibuat.**

Alasan: kalau token belum ada, setiap agent AI akan mengarang warna dan spacing sendiri di
setiap file. Itu penyebab utama UI terlihat seperti hasil tempelan. Setelah token ada, aturan
"nol hex hardcoded" di `AGENTS.md` bisa ditegakkan.

Ikon di mockup memakai pola SVG sprite (`<use href="#ic-*">`). Pertahankan pola itu — tidak
perlu icon library.

## Peta screen

"Terverifikasi" berarti isinya sudah dibaca langsung dari file mockup. Yang belum, tolong
dikonfirmasi saat mengerjakan task terkait dan perbarui tabel ini.

| # | Screen | Terverifikasi | Prioritas | Task | Pemilik |
| --- | --- | --- | --- | --- | --- |
| s1 | Help Center (hero search, kategori, paling dibaca, banner AI) | ya | P1 | T-007 | B |
| s2 | Hasil pencarian (filter, kotak jawaban AI + sitasi, kartu hasil) | ya | P1 | T-011 | B |
| s3 | (perlu dikonfirmasi — kemungkinan doc viewer) | belum | P1 | T-006 | B |
| s4 | (perlu dikonfirmasi — kemungkinan AI chat penuh) | belum | P1 | T-013 | B |
| s5 | Unggah Knowledge (wizard 4 langkah, metadata, klasifikasi) | ya | P1 | T-010 | B |
| s6 | Konsol approval admin | ya | P2 | T-012 | B |
| s7 | (perlu dikonfirmasi — kemungkinan admin user/RBAC) | belum | P2 | — | B |
| s8 | (perlu dikonfirmasi — kemungkinan analytics) | belum | P3 / v1.1 | — | — |

## Detail yang sudah dikonfirmasi dari mockup

**Kategori** (6): Infrastruktur & Jaringan, Keamanan Informasi, Aplikasi Internal,
SOP & Proses Bisnis, Onboarding & SDM, Data & Integrasi.

**Label**: Identity, Runbook, Helpdesk, SOP, Kritikal, Onboarding, Migrasi — bebas dan
boleh lebih dari satu per dokumen.

**Filter pada halaman pencarian**: kategori (dengan jumlah), label, format berkas,
terakhir diperbarui, status.

**Metadata dokumen**: judul, ringkasan, kategori, sub-kategori, label, klasifikasi keamanan,
pemilik dokumen, periode tinjau ulang, reviewer/approver.

**Elemen kepercayaan**: badge Terverifikasi, nomor versi (`v3.1`), tanggal update, pemilik,
badge "Terbatas — akses via permintaan", sitasi bernomor pada jawaban AI.

**Role yang muncul di mockup**: Contributor, Admin Knowledge, dan approver kedua
(Manajer Infrastruktur) untuk dokumen berlabel Kritikal.

## Komponen bersama yang perlu dibangun lebih dulu

`AppShell` (topbar + sidebar) · `SearchInput` · `Pill` / `Tag` · `ClassificationBadge` ·
`Card` · `Steps` · `FileRow` · `EmptyState` · `ErrorState` · `PermissionDeniedState` ·
`CitationChip` · `MarkdownViewer`.

Bangun ini di T-002/T-006 sebelum menyalin screen satu per satu. Menyalin screen tanpa
komponen bersama menghasilkan duplikasi yang mahal untuk dirapikan.

## Yang sengaja tidak ditiru dari mockup di v1.0

Angka statistik pada hero ("1.284 dokumen", "96% akurasi AI") adalah placeholder. Tampilkan
angka nyata atau jangan tampilkan sama sekali. **Jangan pernah menampilkan angka akurasi AI
yang tidak diukur oleh eval harness.**

Deteksi duplikasi, approval berjenjang, OCR, dan dukungan XLSX/PPTX/ZIP muncul di mockup
tetapi merupakan non-goal v1.0. Lihat `docs/prd.md` bagian 5.
