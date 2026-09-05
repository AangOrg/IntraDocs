# Papan task — urutan dependensi, bukan deadline

Semua T-001–T-014 mempunyai spec. Parent task besar berisi subtask; satu subtask terkecil satu sesi/PR <=8 berkas kode/400 baris. Hari/minggu pada dokumen historis bukan izin mempercepat melewati gerbang. Tidak ada task T-015: pemindaian sensitif milik T-006b.

| Task | Pemilik | Prasyarat implementasi | Keluaran |
| --- | --- | --- | --- |
| T-001 | A | Infra pemilik untuk deploy | Scaffold, CI, env, deploy tanpa Docker |
| T-002 | B | T-001a | Token, AppShell responsif |
| T-003 | A | T-001a + DB uji | Sebelas tabel domain dan fixture minimal enam akun |
| T-004 | A | T-003 | Auth.js credentials dan guard sesi terkini |
| T-005 | A | T-003/T-004 | Filter SQL dan tujuh suite izin tingkat query/guard |
| T-006 | B/A per subtask | T-002/T-005 + key AI untuk publish | CRUD, viewer, scan, embed/publish/reindex atomik |
| T-007 | B | T-002/T-005 | Halaman muka/katalog, hitungan berizin |
| T-008 | B | Penulisan bebas; integrasi T-003/T-006b | 20–25 dokumen, 10 kasus eval |
| T-009 | A | T-005/T-006b/T-008 | Hybrid/fallback, log, gerbang retrieval |
| T-010 | B | T-002; integrasi T-009 | /cari, kategori, highlight aman |
| T-011 | A | T-009 lulus gerbang | AI berscope, SSE, ownership/riwayat, sitasi |
| T-012 | B | T-002; integrasi T-011 | UI AI/multi-turn/riwayat/feedback |
| T-013 | A | T-011/T-012 + korpus beku | Evaluasi nyata dan gerbang rilis |
| T-014 | B | Fitur inti + T-013 | Walkthrough, README, bukti rilis |

## Koordinasi

- T-006b memiliki provider embed/pipeline; T-009 memakai, T-011 menambahkan chat lewat kontrak yang sama. Tidak membuat provider duplikat.
- T-003 memiliki fixture akun/minimal; T-008 memiliki korpus dan seed penuh. Keduanya memakai tabel seed matriks RBAC.
- T-005 membuktikan query/guard sebelum fitur; setiap task endpoint kemudian menambah integrasi. Tidak mengklaim HTTP search/chat sudah diuji pada tahap sebelum endpoint dibuat.
- T-001/T-003 dapat memerlukan pecahan lebih kecil karena generated file dihitung dalam batas PR. Definisikan sebelum eksekusi.

## Lulus

Checklist spec dan gerbang scope dipenuhi dengan bukti. Tidak ada persetujuan pemotongan otomatis. Status implementasi tetap di docs/STATUS.md dan PR aktual; tabel ini bukan klaim pekerjaan sudah selesai.
