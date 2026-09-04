# Papan Task

Satu task satu branch `feat/T-0XX-slug`, satu PR, di bawah 400 baris diff. Spec ditulis paling lambat sehari sebelum dikerjakan.

Orang A memegang data, izin, dan retrieval. Orang B memegang antarmuka. Pembagian ini menjaga agar dua orang jarang menyentuh berkas yang sama.

## Sprint enam hari

| Hari | Task | Orang | Keluaran |
| --- | --- | --- | --- |
| 1 | T-001 kerangka proyek dan deploy | A | Repo jalan, CI hijau, URL Vercel hidup |
| 1 | T-002 design token dan `AppShell` responsif | B | Token dari mockup, shell menutup di ponsel |
| 1 | T-003 skema sebelas tabel dan seed | A | Migrasi terpasang, lima pengguna, dua puluh dokumen contoh |
| 2 | T-004 autentikasi akun lokal | A | Masuk dan keluar, sesi berisi role dan cakupan |
| 2 | T-005 filter izin dan test kebocoran | A | `visibleDocumentsFilter` dan seluruh `tests/rbac/` hijau |
| 2 | T-007 katalog dan filter kategori | B | Layar 3 jalan dengan data nyata |
| 3 | T-006 viewer, daftar heading, form sunting | B | Layar 4 jalan, publikasi memicu embedding |
| 3 | T-008 dokumen sintetis | B | Dua puluh sampai dua puluh lima berkas Markdown |
| 3 | T-015 pemeriksaan konten sensitif | A | Peringatan pola kredensial saat publikasi |
| 4 | T-009 hybrid search dan RRF | A | Identifier persis dan parafrase, keduanya tepat |
| 4 | T-010 halaman hasil pencarian | B | Layar 2 jalan |
| 5 | T-011 endpoint RAG, sitasi, abstain | A | SSE mengalir, sitasi menunjuk bagian dokumen |
| 5 | T-012 halaman AI Assistant | B | Layar 9 jalan, percakapan berlanjut |
| 6 | T-013 eval dan tuning | A | `pnpm eval` mencetak angka, kebocoran nol |
| 6 | T-014 polish, README, skrip demo | B | Bisa didemokan tanpa penjelasan |

## Perubahan dari pemeriksaan mockup layar 9-11

Rinciannya di `docs/mockup-alignment.md`. Tidak ada task baru yang besar; sebagian besar diserap task yang sudah ada.

| Task | Yang bertambah | Kenapa sekarang |
| --- | --- | --- |
| T-002 | Sidebar menutup di bawah titik potong | Menyisipkan perilaku responsif ke enam halaman jadi jauh lebih mahal |
| T-003 | `user.category_scope`, `user.is_active`, tabel `conversation` dan `message` | Skema menjadi sebelas tabel. Menambah dimensi izin setelah test hijau berarti menulis ulang test dan seed |
| T-005 | `tests/rbac/category-scope.spec.ts` dan `inactive-user.spec.ts` | Ikut sekali jalan dengan test kebocoran |
| T-009, T-010 | Satu baris log per kueri | Satu `INSERT`. Tanpanya dashboard fase 2 tidak punya data |
| T-011 | Penulisan ulang pertanyaan lanjutan, parameter ruang lingkup | Pertanyaan lanjutan hampir pasti ditanyakan saat demo |
| T-012 | Riwayat percakapan, selektor ruang lingkup, umpan balik jempol | Layar 9 adalah halaman inti, bukan pelengkap |
| T-015 | Task baru, kecil | Persyaratan kepatuhan di layar 11, sekaligus pengaman terhadap aturan kita sendiri |

## Hari yang paling menentukan

**Hari 2.** Kalau filter izin belum benar dan test kebocoran belum hijau di akhir hari 2, jangan lanjut ke hari 3. Seluruh nilai produk ini bertumpu pada satu klausa `WHERE`, dan memperbaikinya setelah empat hari kode menumpuk di atasnya jauh lebih mahal.

**Hari 5 adalah hari terpadat.** Endpoint RAG dan halaman chat dikerjakan bersamaan, dan keduanya bertambah isi setelah pemeriksaan mockup. Kalau hari 5 tersendat, yang dipotong adalah penyimpanan riwayat percakapan lebih dulu, bukan sitasi dan bukan jalur abstain. Urutan potong lengkap ada di `docs/scope-mvp.md`.

## Spec yang belum ditulis

T-009 sampai T-015 ditulis sehari sebelum dikerjakan. Menulis semuanya sekarang berarti menulis rencana untuk keadaan yang belum diketahui.
