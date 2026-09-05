# T-014: Pemeriksaan rilis dan README

Pemilik B; prasyarat seluruh fitur inti terintegrasi, T-013 menyelesaikan gerbang mutu.

## Baca tambahan

Scope definisi selesai; ui-inventory.

## Lingkup

T-014a memperbaiki celah state/responsif yang terukur; T-014b menulis README dan menjalankan walkthrough penguji baru. Tidak menunda seluruh state UI sampai task ini.

README mencakup setup lokal tanpa Docker, env per fitur, seed, lima akun aktif/Fajar nonaktif, batas demo sintetis, dan skenario lima menit. Jangan menaruh credential layanan nyata.

## Kriteria terima

- [ ] Penguji baru mengikuti README tanpa instruksi dari chat.
- [ ] Demo meliputi reviewer sempit, viewer aktif, Fajar ditolak, pencarian fallback, sitasi/abstain/multi-turn/scope.
- [ ] Semua halaman inti memiliki state/keyboard/375 px dan error tidak menampilkan stack/secret.
- [ ] Dokumen serta riwayat tak berizin 404; aksi terlarang pada resource terlihat sesuai kontrak.
- [ ] Seed ulang aman; upload md/txt dan publish diuji; failure embed tidak merusak versi lama.
- [ ] Semua gerbang scope dan empat pemeriksaan CI lulus pada commit yang ditinjau.
- [ ] Angka demo berasal dari data/eval, bukan mockup. Screenshot dan hasil pemeriksaan dilampirkan.

Tidak membuat tag, merge, atau menghapus branch tanpa permintaan eksplisit. Tidak membuka kembali fitur Fase 2 atas nama polish. STATUS diperbarui oleh chat eksekusi, bukan chat review.
