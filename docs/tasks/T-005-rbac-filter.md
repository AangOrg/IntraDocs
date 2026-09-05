# T-005: Filter SQL dan tujuh test izin

Pemilik A. Prasyarat T-003/T-004. Semua jalur baca produk menunggu parent task ini selesai.

## Baca tambahan

`docs/rbac-matrix.md`, ADR-0004/0009/0011/0012 hanya bila perlu alasan keputusan.

## Subtask sebelum fitur search/AI

- T-005a: visibleDocumentsFilter(user): SQL, can(user, action, resource), perbandingan klasifikasi, ai-retrieval-leak.spec.ts. Ini test keamanan pertama; tidak melarang test scaffold/migrasi sebelumnya.
- T-005b: catalog-visibility.spec.ts, search-visibility.spec.ts, category-scope.spec.ts.
- T-005c: ai-max-classification.spec.ts, audit-log.spec.ts, melengkapi inactive-user.spec.ts milik T-004.

Semua nama test berada di tests/rbac/. Batas per PR 8 berkas/400 baris termasuk helper. Jangan memuat tujuh test beserta semua modul dalam satu PR jika melampaui batas.

## Batas pembuktian

T-005 menguji fragmen izin pada query DB nyata terhadap fixture, full-text dan query jarak vektor memakai embedding tetap sintetis. Adapter konteks memakai spy provider untuk memastikan hasil SQL tak berizin tidak dikirim. Belum mengklaim endpoint search/chat ada. T-006/007/009/011 menambah pengujian integrasi endpoint pada suite yang sama ketika endpoint dibuat. Tidak memakai filter array sebagai pengganti bukti WHERE.

## Kriteria terima

- [ ] Ketujuh berkas test wajib lulus sebelum parent task ditutup.
- [ ] Matriks empat role/empat klasifikasi/status draft-published/cakupan NULL-kosong-dalam-luar sesuai matriks. Admin tidak bypass fungsi.
- [ ] Filter dijalankan sebelum LIMIT, ranking, total, dan pengiriman konteks.
- [ ] Draft tidak masuk jalur search/RAG; pemilik draft tetap tunduk maksimum klasifikasi.
- [ ] Scope hanya mempersempit izin; perubahan kategori/klasifikasi tidak membocorkan chunk lama.
- [ ] AI_MAX_CLASSIFICATION diuji empat tingkat; error provider tidak menyebabkan bypass.
- [ ] Audit pembacaan restricted/secret menyimpan referensi, bukan isi.
- [ ] Fajar ditolak, Viewer Demo berhasil; variasi contributor/reviewer/admin nonaktif di fixture sementara juga ditolak. Sesi lama mengikuti guard T-004.

Nol izin di prompt; nol penyaringan setelah query sebagai satu-satunya pengaman. Detail aturan tidak disalin lagi ke spec ini.
