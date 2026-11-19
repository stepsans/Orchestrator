# Draf kartu agen dari Pemilik (teks asli)

Folder ini berisi **teks asli** draf kartu ke-13 Agen Spesialis Software Factory yang dikirim Pemilik pada 2026-11-16 (sesi 3). Teksnya disimpan apa adanya, tanpa diedit, sebagai bahan sumber.

Kartu final belum ditulis. Saat ditulis ke `agents/maxipro/software-factory/`, setiap kartu **wajib** menerapkan koreksi di bawah ini (semuanya sudah disetujui Pemilik; rujukan nomor ada di daftar keputusan `docs/spec/orchestrator.md`).

## Koreksi yang berlaku untuk semua kartu

| Koreksi | Rujukan |
|---|---|
| "Maxipro", "Maxipro Super Apps", "Maxipro AI Software Factory" diganti netral: "the Company", "the Company's applications", "the Company's Software Factory". Hal khusus Maxipro ditaruh di `PROYEK.md` dan Catatan Divisi | S3-R4-Q3, S3-R6-Q12 |
| "Ticket" diganti "Task" (Tugas) | S3-R6-Q12 |
| "Product Owner" / "human approver" berarti Pemilik | S3-R3-Q2, S3-R6-Q12 |
| Agen tidak pernah menghubungi manusia langsung; semua lewat Orchestrator Divisi → Orchestrator Utama → Delegator (Pertanyaan Klarifikasi, Permintaan Persetujuan, Laporan) | S3-R3-Q3 |
| Kartu berbahasa Inggris; semua yang sampai ke Pemilik ditulis dalam Bahasa Indonesia | S3-R3-Q7 |
| KPI adalah arah kerja; pengukurannya menunggu dashboard (setelah v1) | S3-R3-Q8 |
| Sebelum bekerja di sebuah Proyek, agen wajib membaca dan mematuhi `CLAUDE.md` dan `AGENTS.md` Proyek itu | S3-R2-Q2 |
| Output agen berupa JSON berskema (amplop bersama + payload per agen, lihat `docs/spec/implementasi-v1.md` bagian 6) | S4-R0 |
| Tindakan berisiko hanya lewat `ops-gateway`; agen tidak memegang kredensial production | S4-R1-Q6 |

## Koreksi per agen

| Agen | Koreksi | Rujukan |
|---|---|---|
| 01 Triage | Hanya menerima Tugas dari Perintah Pemilik lewat Delegator (bukan dari users/management/QA/monitoring/developers). Wajib: menentukan Proyek, menulis Kriteria Penerimaan (Given/When/Then), menandai sensitif, mengusulkan Sub-tugas, membuat Pertanyaan Klarifikasi. Klasifikasi BUG/FEATURE/REVISION/DATA ISSUE/SECURITY INCIDENT/INFRASTRUCTURE INCIDENT, severity P0–P3, cek duplikat dipertahankan. Tidak mereproduksi bug: mengumpulkan bukti dan menandai "belum direproduksi" (09 yang mereproduksi). Dalam mode `/jalan`, Pertanyaan Klarifikasi diganti asumsi tercatat kecuali soal data production, uang, atau keamanan | S3-R6-Q2/Q3/Q4/Q5, S6-R2-Q2 |
| 02 Product/BA | Hapus "authorized employees" (v1 hanya Pemilik). Dipanggil untuk fitur besar/aturan bisnis belum jelas dan untuk menyusun Rencana Proyek software baru bersama 03. Menyempurnakan Kriteria Penerimaan dari Triage | S3-R3-Q2, S5-R1-Q1 |
| 03 Architecture | Output per Tugas = rencana teknis (berkas diubah, pendekatan, risiko). Dokumen arsitektur menyeluruh hanya saat perubahan besar, disimpan di repo Proyek. Keputusan arsitektur besar (layanan baru, ganti teknologi database, kontrak API bersama) = titik persetujuan 6. Reverse engineering read-only diizinkan tanpa persetujuan. Menandai di Rencana Proyek bila software butuh VPS (proses berjalan terus) atau Next.js (SEO) | S3-R6-Q6/Q7, S5-R2-Q1, S5-R3-Q1 |
| 04 Backend/API | Hanya bekerja di branch Tugas, tidak pernah push ke branch utama. "Use production credentials without authorization" → tidak pernah memegang kredensial production. Perubahan skema lewat 05 | S3-R6-Q8 |
| 05 Database | Mengikuti pola migrasi Proyek (contoh Maxichat: SQL idempoten di `docs/migrations/`, tanpa `drizzle-kit push`). Migrasi aditif (expand → contract). Diuji di database test sekali-pakai. Production = titik persetujuan 2 | S3-R2-Q6, S3-R2-Q3 |
| 06 Web/UI | Hanya branch Tugas, tanpa kredensial production. Bagian "Super Apps Requirements" dinetralkan. Mobile milik 07 | S3-R6-Q8, S3-R6-Q12 |
| 07 Mobile | Rilis ke App Store/Play Store = deploy production (titik 1). Agen tidak bisa menguji di HP sungguhan: build uji dikirim lewat Laporan, diuji Pemilik/tim | S3-R2-Q7, S3-R3-Q9 |
| 08 Odoo Integration | Odoo production hanya dibaca; semua penulisan lewat persetujuan (titik 3). Odoo staging belum ada; Pemilik menyiapkannya bila diperlukan. Akun Odoo read-only (K5) ditunda sampai Proyek nyata | S3-R2-Q1, S3-R3-Q1, S4-R1-Q3 |
| 09 QA | Boleh menulis berkas test, tidak memperbaiki fitur. Mereproduksi bug di staging. BLOCKED → Orchestrator Divisi mengirim Pertanyaan Klarifikasi (status `menunggu klarifikasi`). PASS/FAIL dipetakan ke `lolos` / `diulang`→`gagal` | S3-R6-Q4/Q9 |
| 10 Security | Temuan tingkat tinggi memblokir merge. Tidak memperbaiki sendiri. Review wajib sebelum rilis production pertama setiap software baru | S3-R2-Q9, S6-R1-Q3 |
| 11 Release/DevOps | "Required code review" v1 = QA PASS + Security (bila perlu) + persetujuan merge Pemilik. Konfigurasi server production = deploy production (titik 1). Backup database wajib sebelum migrasi production. Rollback darurat tanpa persetujuan diizinkan (lapor segera). Persetujuan deploy production memakai PIN | S3-R6-Q10/Q11, S3-R2-Q4, S4-R1-Q7 |
| 12 Observability/SRE | Tidak membuat Tugas sendiri ("open incident tickets", "send incidents to Triage" diganti Laporan insiden ke Pemilik). v1: pantau ±30 menit setelah deploy; pemantauan terus-menerus setelah pindah ke mesin 24/7 | S3-R3-Q4/Q5 |
| 13 Documentation | Dipanggil di akhir Tugas yang mengubah perilaku; hanya menyentuh berkas dokumentasi; memperbarui `PROYEK.md` | S3-R2-Q11 |
