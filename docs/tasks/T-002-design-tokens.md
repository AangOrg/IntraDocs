# T-002: Token desain dan AppShell

Pemilik B. Implementasi setelah T-001a tersedia; inspeksi visual boleh paralel.

## Baca tambahan

`docs/ui-inventory.md`; bila perlu mockup root `intradocs-mockup_1.html` hanya rentang CSS atau screen yang relevan. Jangan memakai path design/mockup yang tidak ada.

## Subtask

- T-002a: token Tailwind, radius/shadow, Inter self-hosted melalui next/font, sprite ikon dari mockup, halaman dev token.
- T-002b: AppShell topbar 56px/sidebar 240px, navigasi responsif, Badge dan state dasar untuk dipakai ulang.

## Lingkup

Semua nilai desain mengikuti ui-inventory/mockup. Tidak memakai warna keras di komponen atau menambah icon library. Primitif lain dibuat ketika task pemakainya membutuhkan, bukan satu pustaka besar sekaligus.

Sidebar menutup di bawah md, dapat dibuka/ditutup dengan keyboard, fokus kembali ke pemicu. Rute nyata hanya /, /katalog, /cari, /unggah, /ai-assistant dan pembaca dokumen; tautan admin yang belum ada tidak ditampilkan sebagai fitur aktif.

## Kriteria terima

- [ ] Token/ikon dibandingkan dengan rentang mockup yang tepat; tidak mendesain ulang.
- [ ] AppShell terbaca pada 375 px dan desktop; navigasi keyboard berfungsi.
- [ ] Badge klasifikasi memenuhi kontras WCAG AA dan istilah UI benar.
- [ ] Memuat/kosong/gagal tersedia; resource tersembunyi menuju 404, bukan PermissionDeniedState.
- [ ] Halaman dev token hanya tersedia di development, bukan menu produk produksi.
- [ ] Tidak ada PWA/service worker atau dependensi UI baru tanpa persetujuan.

Bukti: screenshot viewport desktop/375 px dan test komponen perilaku sidebar.
