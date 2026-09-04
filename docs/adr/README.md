# Catatan Keputusan Arsitektur

Satu berkas per keputusan yang mahal untuk dibalik. Format: konteks, keputusan, konsekuensi, cara membatalkan.

Kalau sebuah keputusan berubah, jangan sunting ADR lama. Tulis ADR baru yang menyatakan penggantinya.

| No | Keputusan | Status |
| --- | --- | --- |
| [0001](0001-tech-stack.md) | Tumpukan teknologi | Diterima |
| [0002](0002-content-source-of-truth.md) | Sumber kebenaran konten: Postgres | Diterima |
| [0003](0003-ai-provider-and-data-classification.md) | Penyedia AI dan klasifikasi data | Diterima |
| [0004](0004-rbac-and-permission-aware-retrieval.md) | RBAC dan retrieval sadar izin | Diterima |
| [0005](0005-deployment-and-portability.md) | Deploy dan portabilitas | Diterima |
| [0006](0006-embedding-dimension.md) | Dimensi embedding 1024 | Diterima |
| [0007](0007-mvp-scope-reduction.md) | Pemotongan scope ke MVP satu minggu | Diterima |
| [0008](0008-environments-and-cicd.md) | Lingkungan dan CI/CD | Diterima |
| [0009](0009-rbac-cakupan-kategori.md) | RBAC dua dimensi: cakupan kategori | Diterima |
| [0010](0010-chat-multi-turn.md) | Percakapan berlanjut pada AI Assistant | Diterima |

ADR-0009 dan ADR-0010 lahir dari pemeriksaan ulang terhadap layar 9-11 mockup. Latar lengkapnya ada di `docs/mockup-alignment.md`.
