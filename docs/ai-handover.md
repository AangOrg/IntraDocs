# Melanjutkan proyek di chat, akun, atau workspace lain

Riwayat percakapan **tidak bisa dipindahkan** antar akun atau workspace. Kalau kuota chat
habis, atau berpindah perangkat, atau asistennya lupa konteks di tengah jalan, seluruh
konteks percakapan hilang.

Karena itu satu prinsip berlaku sejak awal:

> **Keadaan proyek disimpan di repo, tidak di dalam percakapan.**

Kalau prinsip ini dipegang, berpindah akun butuh dua menit. Kalau tidak, berpindah akun
berarti menjelaskan ulang seluruh proyek dari nol — dan asisten yang baru akan mengarang
asumsi yang bertentangan dengan keputusan yang sudah diambil.

## Apa yang hilang dan apa yang tidak

| Hilang saat pindah | Tidak hilang |
| --- | --- |
| Riwayat percakapan | Semua berkas di repo |
| Halaman Notion di workspace lama | ADR dan alasan di baliknya |
| Koneksi GitHub (harus disambungkan ulang) | Keadaan harian di `docs/STATUS.md` |
| Konteks implisit yang tidak pernah dituliskan | Spec task dan acceptance criteria |

Kolom kiri adalah yang tidak boleh dijadikan tempat menyimpan apa pun yang penting.

## Prosedur pindah

1. Buka Notion dengan akun atau workspace yang baru.
2. Sambungkan GitHub pada akun tersebut, dengan akses ke `AangOrg/IntraDocs`.
3. Tempel **prompt bootstrap** di bawah ini ke chat baru.
4. Biarkan asisten membaca berkasnya dan meringkas keadaan. **Periksa ringkasannya** — kalau
   ringkasannya salah, berkas keadaannya yang perlu diperbaiki, bukan diteruskan begitu saja.
5. Lanjutkan bekerja.

## Prompt bootstrap — tempel apa adanya

```
Kamu melanjutkan proyek yang sudah berjalan. Jangan menulis kode sebelum membaca.

Repo: https://github.com/AangOrg/IntraDocs
Proyek: IntraDocs, web dokumentasi internal dengan RBAC dan AI chat bersitasi.
Konteks: tugas magang, dua orang, dikerjakan dengan bantuan agent AI.
Target MVP: Jumat 11 September 2026.

Baca berkas berikut dari repo, dengan urutan ini:
1. docs/STATUS.md       - keadaan hari ini dan task yang sedang berjalan
2. docs/context-pack.md - fakta proyek yang stabil
3. docs/scope-mvp.md    - apa yang masuk MVP dan apa yang tidak; berkas ini mengikat
4. AGENTS.md            - aturan menulis kode di repo ini
5. docs/adr/README.md   - indeks keputusan; baca ADR yang relevan dengan task berjalan

Batasan yang tidak boleh dilanggar:
- Jangan pernah push ke main. Satu task satu branch feat/T-00X-slug, lalu buka PR.
- Jangan pernah merge PR atau menghapus branch tanpa permintaan eksplisit dari saya.
- Filter izin selalu di SQL lewat visibleDocumentsFilter, tidak pernah di dalam prompt LLM.
- Seluruh isi dokumen bersifat sintetis. Jangan pernah memakai data Telkom yang asli.
- Bahasa Indonesia untuk teks UI dan dokumen. Bahasa Inggris untuk identifier kode dan
  commit message.
- Kalau sebuah keputusan sudah tercatat di ADR, ikuti. Kalau menurutmu keputusan itu salah,
  katakan sebelum menulis kode, jangan diam-diam menyimpang.

Setelah membaca, ringkas dalam lima baris: hari sprint keberapa, apa yang sudah selesai, apa
task berikutnya, apa yang sedang rusak atau tertunda, dan apa yang kamu butuhkan dari saya.
Tunggu konfirmasi saya sebelum mulai menulis kode.
```

## Kewajiban harian — satu-satunya yang membuat ini berhasil

Di akhir setiap hari sprint, perbarui `docs/STATUS.md` dan commit. Butuh dua menit.

```bash
git add docs/STATUS.md && git commit -m "docs: update status hari N" && git push
```

Ini terlihat seperti formalitas sampai suatu saat kuota habis di tengah hari 4. Setelah itu,
ini terasa seperti dua menit terbaik yang pernah dipakai di proyek ini.

Aturan praktisnya: **kalau sebuah informasi hanya ada di dalam chat, informasi itu belum
ada.** Keputusan masuk ke ADR, keadaan masuk ke `STATUS.md`, rencana masuk ke `scope-mvp.md`.
