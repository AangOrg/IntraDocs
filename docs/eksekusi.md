# Protokol eksekusi dan QC hemat konteks

Baseline ADR-0012 berlaku setelah seluruh rangkaian PR dokumentasi digabung. Tidak memulai kode dari sebagian stack yang kontraknya belum lengkap. STATUS tetap milik chat eksekusi; review tidak mengubahnya.

## Unit kerja

Satu subtask terkecil = satu sesi = satu branch = satu PR. Spec parent memuat subtasknya. Maksimal 8 berkas kode dan 400 baris total diff termasuk migrasi; kalau perkiraan lebih besar, pecah spec sebelum coding, bukan menyembunyikan generated diff. Parent task selesai setelah semua subtask/integrasinya lulus.

## Bacaan minimum

Awal sesi eksekusi: STATUS, AGENTS, context-pack, satu spec aktif. Tambahan hanya bagian yang ditunjuk spec. Untuk review: nomor PR + head commit + diff + kriteria terima + hasil test; matriks RBAC hanya bila jalur izin terdampak. Jangan memuat ulang HTML atau seluruh dokumen per pertanyaan.

Gunakan checkout lokal pada commit tetap untuk agent eksekusi: file dibaca sekali, edit sebagai patch, git diff untuk review. GitHub MCP dipakai untuk status/PR/komentar, bukan transport seluruh repo berulang. Perubahan kontrak memerlukan pembacaan dependennya; perubahan kosmetik tidak.

## Serah terima maksimal 12 baris

Task/subtask; base/head commit; file berubah; kriteria terpenuhi; perintah test dan hasil; test belum jalan; blocker; keputusan baru/rujukan; langkah berikut. Tidak menyalin riwayat percakapan. Simpan fakta stabil di repo, bukan hanya chat. Tidak menyimpan secret di paket konteks.

## Urutan dan pemilik

Papan task adalah satu-satunya tabel urutan di docs/tasks/README.md. T-003 sebelum task data; T-005 sebelum jalur baca produk; T-006b/T-008 sebelum T-009; T-009 melewati gerbang sebelum T-011. T-012 boleh UI kontrak paralel, tetapi belum selesai sebelum integrasi.

## Pemeriksaan dan penggunaan AI

Jalankan typecheck/lint/test/build lokal dahulu. Kirim error yang relevan, bukan seluruh log. Perbaikan sintaks/format sederhana tidak butuh audit AI besar. Evaluasi generasi hanya untuk perubahan retrieval/prompt/provider atau gerbang rilis; bukan setiap perubahan UI.

QC melaporkan blocker/major/minor + file/lokasi + pelanggaran kriteria + cara reproduksi. Review lanjutan cukup diff sejak commit yang terakhir ditinjau. Boleh komentar PR; tidak memperbaiki kode, merge, atau tag dari chat QC.

Sesi baru ketika berganti task atau konteks penuh, bukan untuk setiap pertanyaan kecil dalam task yang sama. Brainstorming tetap boleh di chat pendek; keputusan yang disetujui ditulis sekali ke ADR/spec, lalu tutup topiknya.

## Mockup

Sumber root intradocs-mockup_1.html, tidak dipindah. Gunakan ui-inventory/matriks alignment dahulu. Cari id screen s1–s11 atau blok :root lalu baca rentang; nomor baris berubah antar commit, jangan menganggap perkiraan lama selalu benar. Jangan buka HTML utuh.

## Penutupan

Chat eksekusi memperbarui STATUS maksimal 15 baris sesuai hasil nyata. Test belum dijalankan tidak ditandai lulus. PR memakai template, menyertakan build dan test izin yang relevan. Dependensi/prasyarat belum tersedia adalah blocker eksplisit.

Dokumen workflow/agent-tooling/ai-handover dan prompt lama adalah referensi, bukan sumber cakupan tambahan. Jangan mengikuti perintah lama membangun Docker/invite/fitur Fase 2. Kuota/tool trial tidak diasumsikan tak terbatas; optimalkan ukuran konteks dan jumlah panggilan sebelum menambah alat.
