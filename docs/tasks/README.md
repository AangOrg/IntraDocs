# Task specs

Satu file per task. Satu task per branch. Satu branch per PR.

## Kenapa task ditulis sebagai spec, bukan sebagai judul

Kualitas kode yang dihasilkan agent AI hampir seluruhnya ditentukan oleh kualitas
instruksinya. "Buatkan halaman upload" menghasilkan kode karangan. Spec dengan file yang
disebutkan, acceptance criteria yang bisa diuji, dan batasan yang eksplisit menghasilkan
kode yang bisa langsung di-review.

Menulis spec juga memaksa kita memikirkan masalahnya lebih dahulu — yang jauh lebih murah
daripada menemukannya saat review.

## Format

Setiap task punya bagian: **Tujuan**, **Konteks** (file yang perlu dibaca), **Lingkup**
(dan yang di luar lingkup), **Acceptance criteria** (bisa dicentang), **Batasan**, dan
**Cara menguji**.

## Cara memakai bersama agent

1. Tempel `docs/context-pack.md` + file task ke agent.
2. Minta agent membaca file yang disebut di **Konteks** sebelum menulis kode.
3. Minta agent bekerja di branch `feat/T-00X-slug` lalu membuka PR.
4. Review diff dengan model kelas atas, bukan dengan mata saja.

## Papan task

| ID | Task | Minggu | Pemilik | Bergantung pada |
| --- | --- | --- | --- | --- |
| T-001 | Repo setup, CI, docker-compose | 0 | A | — |
| T-002 | Design token dari mockup | 0 | B | T-001 |
| T-003 | Skema DB + migrasi | 0 | A | T-001 |
| T-004 | Auth: akun lokal + invite | 1 | A | T-003 |
| T-005 | RBAC + `visibleDocumentsFilter` + test kebocoran | 1 | A | T-003, T-004 |
| T-006 | CRUD dokumen + viewer Markdown | 1 | B | T-003, T-005 |
| T-007 | Katalog, kategori, label, Help Center | 1 | B | T-006 |
| T-008 | Seed 20–30 dokumen sintetis | 1 | B | T-003 |
| T-009 | Upload + object storage + job konversi | 2 | A | T-003 |
| T-010 | Wizard unggah 4 langkah | 2 | B | T-009 |
| T-011 | Hybrid search + RRF + halaman hasil | 2 | A+B | T-005, T-008 |
| T-012 | Alur review → publish + audit log | 2 | A | T-006 |
| T-013 | Chunk + embed + endpoint RAG | 3 | A | T-011 |
| T-014 | UI chat + sitasi + abstain | 3 | B | T-013 |
| T-015 | Eval harness | 3 | A | T-013 |
| T-016 | Rate limit + observability + health | 3 | A | — |

T-009 sampai T-016 ditulis pada akhir minggu sebelumnya, ketika bentuk kodenya sudah
diketahui. Menulis semuanya sekarang akan menghasilkan spec yang harus dibuang.
