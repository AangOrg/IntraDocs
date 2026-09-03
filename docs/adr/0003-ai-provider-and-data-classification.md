# ADR-0003: Abstraksi provider AI & batas klasifikasi data

- Status: Diterima
- Tanggal: 2026-09-03

## Konteks

Produk ini menjanjikan AI chat di atas dokumentasi internal. Tetapi pada saat perencanaan,
**belum ada jawaban resmi** apakah dokumen internal boleh diproses oleh LLM cloud, dan
apakah tersedia GPU on-prem.

Menunggu jawaban akan membuang minggu pertama. Menebak jawabannya berisiko: bila kita
membangun dengan asumsi cloud lalu ditolak keamanan, fitur AI harus ditulis ulang di minggu
terakhir. Bila kita berasumsi on-prem lalu GPU tidak tersedia, kita berhenti total.

Ada risiko yang lebih besar lagi: mengunggah dokumen internal Telkom yang asli ke API publik
sebelum ada persetujuan adalah insiden data, bukan sekadar kesalahan teknis.

## Keputusan

Tiga bagian.

**1. Satu antarmuka provider.** Semua pemanggilan model melewati `lib/ai/provider.ts`:

```ts
export interface AiProvider {
  embed(texts: string[]): Promise<number[][]>   // selalu 1024 dimensi, lihat ADR-0006
  chat(messages: Message[], opts): AsyncIterable<string>
}
```

Provider dipilih lewat env: `AI_PROVIDER=openai|azure|ollama|vllm|bedrock`. Tidak ada SDK
vendor yang diimpor di luar file ini. Konsekuensinya, ganti provider = ganti env + reindex,
bukan refactor.

**2. Batas klasifikasi yang ditegakkan kode.** Env `AI_MAX_CLASSIFICATION` bernilai
`public | internal | restricted | secret`. Lapisan retrieval **menolak** mengirim chunk di
atas batas itu ke provider, walaupun pengguna berwenang membacanya.

Saat ini nilainya `public`. Naikkan hanya setelah ada persetujuan tertulis.

Ketika jawaban terpotong oleh batas ini, UI mengatakan dengan jujur bahwa jawaban dibatasi
kebijakan AI yang berlaku, lalu tetap menampilkan dokumen yang relevan agar pengguna bisa
membacanya sendiri. Produk tetap berguna; hanya lapisan generatifnya yang dibatasi.

**3. Aturan keras data.** **Jangan pernah mengunggah dokumen internal Telkom yang asli ke
environment mana pun yang terhubung ke API publik.** Pengembangan dan demo memakai **20–30
dokumen sintetis** bergaya mockup (`SOP-IT-014 Manajemen Identitas`, `Kebijakan Password &
Autentikasi`, `Runbook VPN`). Dokumen sintetis ini sekaligus menjadi eval set.

## Alternatif yang dipertimbangkan

| Opsi | Kenapa tidak dipilih |
| --- | --- |
| Menunggu keputusan keamanan lebih dulu | Membuang 25% dari total waktu proyek |
| Langsung pakai OpenAI dan mengurus izin nanti | Risiko insiden data pada dokumen asli |
| Hanya on-prem sejak awal | GPU belum dipastikan tersedia; iterasi jadi lambat |
| LangChain sebagai lapisan abstraksi | Abstraksi berat untuk antarmuka dua fungsi |

## Konsekuensi

**Lebih mudah:** pengembangan berjalan hari ini tanpa keputusan kebijakan; keputusan
keamanan tidak lagi memblokir; demo aman ditunjukkan ke siapa pun.

**Lebih sulit:** kualitas jawaban harus diverifikasi ulang setelah provider final dipilih
(karena itu eval harness wajib ada, lihat `docs/eval/README.md`); dokumen sintetis harus
ditulis dengan sungguh-sungguh agar demo terasa nyata.

**Risiko yang diterima:** model lokal 7B–14B menghasilkan jawaban yang lebih lemah daripada
model cloud terbaik. Dengan retrieval yang baik dan kewajiban sitasi, ini masih memadai
untuk tanya-jawab dokumentasi — dan lebih baik daripada tidak punya fitur AI sama sekali.

## Cara membatalkan

Ubah `AI_PROVIDER`, jalankan `pnpm reindex`, lalu jalankan `pnpm eval` untuk membandingkan
angka sebelum dan sesudah. Tidak ada migrasi skema karena dimensi embedding sudah
distandardisasi (ADR-0006).
