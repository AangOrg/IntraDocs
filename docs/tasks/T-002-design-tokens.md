# T-002: Design token dari mockup

- Pemilik: Orang B · Minggu 0 · Estimasi: 0,5 hari

## Tujuan

Seluruh sistem visual mockup tersedia sebagai token Tailwind, **sebelum** ada komponen yang
ditulis.

## Kenapa ini dikerjakan lebih dahulu

Kalau token belum ada, setiap agent AI akan mengarang warna, radius, dan shadow sendiri di
setiap file. Itu penyebab utama UI yang terlihat seperti tempelan, dan sangat mahal untuk
dirapikan setelah 20 komponen dibuat. Dengan token, aturan "nol hex hardcoded" di
`AGENTS.md` menjadi bisa ditegakkan otomatis.

## Konteks

Baca `design/mockup/intradocs-mockup_1.html`, khususnya blok `:root` di dalam `<style>`.
Baca juga `docs/ui-inventory.md`.

## Lingkup

- Ekstrak semua CSS variable menjadi `tailwind.config.ts`: skala `blue-50..900`, `ink`,
  `ink-2`, `muted`, `muted-2`, `line`, `line-2`, `bg`, `white`, ditambah pasangan semantik
  `green`/`green-bg`, `amber`/`amber-bg`, `red`/`red-bg`, `violet`/`violet-bg`.
- Radius (`r`, `r-lg`) dan shadow (`sh-sm`, `sh`, `sh-md`, `sh-lg`).
- Font Inter melalui `next/font` (self-hosted, bukan CDN).
- Sprite SVG ikon sebagai satu komponen React, mempertahankan pola `<use href="#ic-*">`
  dari mockup.
- Petakan klasifikasi ke warna semantik: Publik → hijau, Internal → biru, Terbatas → amber,
  Rahasia → merah. Konsisten di seluruh aplikasi.
- Satu halaman `/dev/tokens` yang menampilkan semua token dan ikon secara visual.

## Di luar lingkup

Komponen fitur. Hanya token, ikon, dan primitif.

## Acceptance criteria

- [ ] Semua CSS variable dari mockup punya padanan token Tailwind
- [ ] `/dev/tokens` menampilkan seluruh warna, radius, shadow, dan ikon
- [ ] Nol warna hardcoded di luar `tailwind.config.ts`
- [ ] Badge klasifikasi memenuhi kontras WCAG AA
- [ ] Tanpa icon library sebagai dependensi

## Batasan

- Jangan mendesain ulang. Cocokkan mockup.
- Jangan menambah dependensi UI di luar shadcn/ui.

## Cara menguji

Buka `/dev/tokens` dan mockup bersebelahan. Warna dan bentuk harus cocok.
