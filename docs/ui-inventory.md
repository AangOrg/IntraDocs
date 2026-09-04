# Inventaris UI — 11 layar mockup

Sumber: `intradocs-mockup_1.html`. Array `SCREENS` di akhir berkas adalah daftar resmi.

| # | Layar | Rute | Status MVP |
|---|---|---|---|
| 1 | Halaman Muka | `/` | Dibangun, versi ringkas |
| 2 | Hasil Pencarian | `/cari` | Dibangun |
| 3 | Katalog Dokumen | `/katalog` | Dibangun, filter kategori saja |
| 4 | Baca Dokumen | `/{kategori}/{slug}` | Dibangun |
| 5 | Unggah Knowledge | `/unggah` | Sebagian — form sederhana, bukan wizard 4 langkah |
| 6 | Approval Admin | `/admin/approval` | Ditunda |
| 7 | Kategori & Label | `/admin/kategori-label` | Ditunda |
| 8 | Pengguna & RBAC | `/admin/pengguna` | Ditunda — tapi modelnya dibangun penuh |
| 9 | AI Assistant | `/ai-assistant` | Dibangun |
| 10 | Dashboard | `/admin/dashboard` | Ditunda — datanya sudah dikumpulkan sejak hari 1 |
| 11 | Arsitektur & Roadmap | — | Bukan layar aplikasi, materi presentasi |

Layar 8 dan 10 ditunda tampilannya tapi **tidak** ditunda datanya. Layar 8 menentukan bentuk tabel `user`; layar 10 menentukan apa yang harus dicatat `ai_query`. Menunda pengumpulan datanya berarti dasbornya kosong berbulan-bulan setelah dibuat.

## Komponen bersama — dibuat lebih dulu

`AppShell` (topbar 56px, sidebar 240px, responsif) · `SearchInput` · `Pill` / `Tag` · `ClassificationBadge` · `StatusBadge` · `FileTypeBadge` · `Card` · `DataTable` · `DocNav` · `CitationChip` · `MarkdownViewer` · `EmptyState` · `ErrorState` · `PermissionDeniedState`

Tiga state terakhir dibuat di hari yang sama dengan komponennya, bukan di hari pemolesan. Halaman tanpa state kosong akan terlihat rusak justru saat demo, karena demo sering menyentuh jalur yang datanya sedikit.

## Token desain

Disalin apa adanya dari CSS variabel mockup ke `tailwind.config.ts`.

Biru: `--blue-50 #EFF6FF` `-100 #DBEAFE` `-200 #BFDBFE` `-500 #3B82F6` `-600 #2563EB` `-700 #1D4ED8` `-900 #1E3A8A`
Netral: `--ink #0F172A` `--ink-2 #1E293B` `--muted #64748B` `--muted-2 #94A3B8` `--line #E2E8F0` `--line-2 #F1F5F9` `--bg #F8FAFC`
Semantik: `--green #059669` / `#ECFDF5` · `--amber #B45309` / `#FFFBEB` · `--red #DC2626` / `#FEF2F2` · `--violet #7C3AED` / `#F5F3FF`
Radius: `--r 10px` `--r-lg 14px`. Bayangan: `--sh-sm` `--sh` `--sh-md` `--sh-lg`. Huruf: Inter.
Warna jenis berkas: pdf `#DC2626` · doc `#2563EB` · md `#0F172A` · xls `#059669` · ppt `#EA580C` · txt `#64748B`

## Aturan responsif

Sidebar menutup di bawah titik potong `md`; topbar tetap. Tidak ada manifest, service worker, atau mode luring — itu pekerjaan tersendiri yang tidak membuktikan apa pun minggu ini.

## Istilah UI yang harus persis seperti mockup

"Publik / Internal / Terbatas / Rahasia" · "Terverifikasi" · "Menunggu Approval" · "Ruang lingkup jawaban" · "Sumber jawaban" · "Jawaban dibatasi hak akses Anda" · "Jawaban selalu menyertakan sumber" · "Percakapan baru"
