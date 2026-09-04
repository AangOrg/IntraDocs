# Keadaan proyek

Berkas ini diperbarui **di akhir setiap hari sprint**. Ini berkas pertama yang dibaca asisten
AI baru — lihat `docs/ai-handover.md`.

Tulis singkat dan jujur. Berkas keadaan yang optimistis lebih buruk daripada tidak ada.

---

**Terakhir diperbarui:** 2026-09-04, 10:30 WIB
**Hari sprint:** 1 dari 6 (target MVP: Jumat 11 September 2026)

## Sudah selesai

- Mockup HTML dibaca seluruhnya, delapan screen dipetakan ke task (`docs/ui-inventory.md`)
- Dokumen perencanaan lengkap: PRD, arsitektur, kontrak API, matriks RBAC, ADR 0001–0008
- Scope dipotong ke sprint enam hari (ADR-0007), definisi selesai ada di `docs/scope-mvp.md`
- Keputusan terbuka dijawab sendiri (`docs/decisions-open.md`)
- Rencana lingkungan, CI/CD, dan alat agent ditulis (ADR-0008, `docs/environments.md`,
  `docs/agent-tooling.md`)

## Sedang dikerjakan

- PR #1 (`docs/foundation`) terbuka, menunggu review

## Berikutnya

- T-001 scaffold + T-003 skema DB + seed (Orang A)
- T-002 design token + `AppShell` (Orang B)

## Tertunda atau belum siap

- Belum ada kode aplikasi — menunggu persetujuan PR #1
- Akun Neon dengan pgvector belum dibuat
- Kunci API provider AI belum disiapkan
- Branch protection pada `main` belum diaktifkan

## Catatan untuk diri sendiri nanti

- Hari 2 (RBAC + test kebocoran) adalah hari terpenting. Jangan lanjut ke hari 3 kalau test
  RBAC belum hijau.
- Menulis 20–25 dokumen sintetis makan 4–6 jam. Mulai mencicil di hari 3, jangan ditumpuk.
- Urutan potong kalau waktu habis ada di `docs/roadmap.md`. Ikuti daftarnya, jangan
  berimprovisasi saat panik.
