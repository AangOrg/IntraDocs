---
description: Review perubahan yang belum di-merge terhadap aturan IntraDocs
---

Review perubahan pada branch ini terhadap `main`.

Baca `AGENTS.md`, `docs/rbac-matrix.md`, dan `docs/scope-mvp.md` lebih dahulu, lalu periksa
diff-nya.

Periksa dengan urutan ini — urutannya sesuai besarnya akibat kalau terlewat:

1. **Kebocoran izin.** Apakah ada query dokumen, chunk, atau hasil search yang tidak melewati
   `visibleDocumentsFilter`? Apakah ada endpoint yang mengembalikan 403 padahal seharusnya 404
   untuk dokumen yang tidak terlihat? Apakah ada logic izin yang berpindah ke prompt LLM?
2. **Kebocoran rahasia.** Kunci API, connection string, atau token di dalam kode, test, atau
   berkas contoh.
3. **Kebenaran.** Apakah kodenya benar-benar melakukan apa yang diklaim deskripsi PR? Apakah
   ada penanganan error yang menelan kegagalan tanpa jejak?
4. **Scope.** Apakah ada yang dikerjakan di luar spec task? Apakah ada fitur fase 2 yang
   menyelinap masuk?
5. **Ketaatan aturan.** `any` di TypeScript, warna hardcoded, state kosong/error/tanpa izin
   yang belum ada, logic bisnis yang bocor ke komponen React.
6. **Test.** Apakah test yang ada benar-benar bisa gagal kalau kodenya salah? Test yang selalu
   lolos lebih buruk daripada tidak ada test, karena memberi rasa aman yang keliru.

Aturan penyampaian:

- Kelompokkan temuan menjadi **harus diperbaiki** dan **sebaiknya diperbaiki**. Jangan
  campur.
- Untuk setiap temuan, sebutkan berkas, baris, dan akibat nyatanya. Bukan sekadar preferensi.
- Kalau tidak ada temuan yang harus diperbaiki, katakan begitu dengan jelas. Jangan mencari
  masalah demi terlihat teliti.
- Jangan memperbaiki apa pun. Review saja.
