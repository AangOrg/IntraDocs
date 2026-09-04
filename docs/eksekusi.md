# Protokol eksekusi — supaya kualitas tidak turun di tengah jalan

Masalah yang dicegah berkas ini: sesi AI yang terlalu panjang mulai melupakan aturan di awal, mencampur task, dan menghasilkan kode yang terlihat benar tapi melanggar keputusan yang sudah diambil. Gejalanya muncul pelan dan biasanya baru ketahuan saat integrasi.

## Aturan inti

**Satu task = satu sesi chat = satu branch = satu PR.**

Sesi baru untuk task baru, walaupun sesi lama masih terasa segar. Konteks yang sudah penuh dengan detail task sebelumnya adalah konteks yang lebih buruk untuk task berikutnya, bukan lebih baik.

## Anggaran bacaan per task

Selalu: `docs/STATUS.md` · `AGENTS.md` · `docs/context-pack.md` · spec task-nya.
Tambahan per task — hanya ini, tidak lebih:

| Task | Tambahan yang dibaca |
|---|---|
| T-001 Scaffold + deploy | `docs/environments.md` |
| T-002 Token + AppShell | `docs/ui-inventory.md` |
| T-003 Skema + seed | `docs/architecture.md` (bagian basis data), ADR-0006, ADR-0009 |
| T-004 Auth | ADR-0001 (bagian auth) |
| T-005 RBAC + test kebocoran | `docs/rbac-matrix.md`, ADR-0004, ADR-0009 |
| T-006 Viewer + editor | `docs/ui-inventory.md`, `docs/api-contract.md` |
| T-007 Katalog | `docs/ui-inventory.md` |
| T-008 Korpus sintetis | `docs/scope-mvp.md` (bagian korpus) |
| T-009 Pencarian hybrid | `docs/architecture.md` (dua jalur data), ADR-0006 |
| T-010 Halaman pencarian | `docs/ui-inventory.md`, `docs/api-contract.md` |
| T-011 Endpoint RAG | ADR-0010, ADR-0003, `docs/api-contract.md` |
| T-012 Halaman AI Assistant | `docs/ui-inventory.md`, `docs/mockup-alignment.md` (bagian layar 9) |
| T-013 Evaluasi | `docs/eval/README.md` |
| T-014 Pemolesan | `docs/ui-inventory.md` |

Berkas yang **tidak** dibaca saat menulis kode: `docs/prd.md`, `docs/roadmap.md`, `docs/decisions-open.md`, `docs/agent-tooling.md`, `docs/ai-handover.md`, `docs/workflow.md`. Semuanya dokumen perencanaan untuk manusia, bukan bahan kerja agent.

## Membaca mockup tanpa menghabiskan konteks

`intradocs-mockup_1.html` berukuran ~178 KB / ~2.010 baris. Membacanya utuh akan memakan sebagian besar jendela konteks dan menyisakan sedikit untuk berpikir.

- Token warna, huruf, dan kerangka shell: sekitar **baris 1–120**.
- Layar 9 AI Assistant: sekitar **baris 1.650–1.830**.
- Layar 10 Dashboard: sekitar **baris 1.780–1.900**.
- Layar 11 Arsitektur: sekitar **baris 1.880–2.010**.
- Layar lain: cari judulnya dulu dengan pencarian kode, lalu baca rentang ~200 baris di sekitarnya.

Untuk sebagian besar task, `docs/ui-inventory.md` sudah cukup dan mockup tidak perlu dibuka sama sekali.

## Batas ukuran satu langkah

- Maksimal **8 berkas kode** disentuh per PR. Lebih dari itu, task-nya salah dipecah.
- Maksimal **400 baris perubahan** per PR.
- Tulis test di PR yang sama dengan kodenya, bukan menyusul.
- Jangan pernah minta satu sesi menghasilkan skema, API, dan UI sekaligus. Itu resep untuk kode yang tidak ada yang benar-benar membacanya.

## Urutan yang tidak boleh dibalik

1. **T-003 sebelum apa pun yang menyentuh data.** Skema dulu.
2. **T-005 sebelum T-007, T-009, T-011.** Filter izin harus ada sebelum ada jalur baca yang bisa membocorkannya. Test kebocoran ditulis **sebelum** fitur pencarian, bukan sesudah.
3. **T-008 sebelum T-009.** Tidak ada gunanya menyetel pencarian tanpa korpus.
4. **T-009 sebelum T-011.** RAG memakai ulang retrieval yang sama.

Selebihnya boleh dikerjakan paralel oleh dua orang.

## Yang ditulis balik setiap akhir task

Perbarui `docs/STATUS.md` di PR yang sama, maksimal 15 baris:

- Hari sprint ke berapa
- Task yang baru selesai dan nomor PR-nya
- Task berikutnya
- Apa yang rusak atau ditunda, kalau ada
- Apa yang dibutuhkan dari pemilik repo

**Kalau sebuah informasi hanya ada di dalam chat, informasi itu belum ada.** Chat akan hilang. Repo tidak.

## Tanda konteks mulai membusuk

Berhenti dan mulai sesi baru kalau salah satu terjadi:

- Agent mengusulkan pustaka yang ada di daftar tolak `docs/context-pack.md`
- Agent menulis ulang berkas yang sudah selesai di task sebelumnya tanpa diminta
- Agent lupa memanggil `visibleDocumentsFilter` di jalur baca baru
- Agent mulai mengarang nama kolom yang tidak ada di skema
- Jawaban jadi panjang dan umum, bukan spesifik ke berkas yang sedang disentuh

Semuanya gejala yang sama: jendela konteks sudah lebih banyak berisi sejarah daripada tugas.

## Ritme harian yang disarankan

Pagi: buka sesi, kerjakan satu task, buka PR, tutup sesi.
Siang: review PR pasangan kerja pakai `/review`, perbaiki, tutup sesi.
Sore: sesi ketiga untuk task kedua hari itu.

Tiga sesi pendek lebih baik daripada satu sesi panjang, walaupun total tokennya sama.
