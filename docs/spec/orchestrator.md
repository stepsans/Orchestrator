# Spesifikasi: Orchestrator

Status sesi: berjalan (sesi kukatanya, dimulai 2026-11-13).

## Tempat catatan

| Catatan | Tempat |
|---|---|
| Istilah | `GLOSSARY.md` (satu konteks: Orchestrator) |
| Keputusan besar | `docs/adr/` |
| Aturan domain dan spesifikasi | dokumen ini |
| Daftar keputusan | dokumen ini |

## Lingkup

- Target sesi: spesifikasi siap implementasi.
- Yang dirinci: Orchestrator Utama dan Orchestrator Divisi "Software Factory" — perutean, serah tugas, status, dan penerimaan hasil.
- Di luar lingkup sesi ini: detail internal tiap Agen Spesialis (01–13), Orchestrator Divisi "Digital Growth", Delegator, My Second Brain.
- Satu Perusahaan dulu: Maxipro.

## Hierarki

```
Pemilik → Delegator → Orchestrator Utama (per Perusahaan) → Orchestrator Divisi → Agen Spesialis
```

Orchestrator Divisi yang direncanakan:

- **Software Factory**: 01 Triage/Intake, 02 Product/Business Analyst, 03 Architecture, 04 Backend/API, 05 Database, 06 Web/UI, 07 Mobile, 08 Odoo Integration, 09 QA/Testing, 10 Security, 11 Release/DevOps, 12 Observability/SRE, 13 Documentation.
- **Digital Growth** (ditunda): 01 Technical SEO & Website Optimization, 02 SEO & AEO Content Strategist, 03 GEO & AI Visibility Specialist, 04 Digital PR & Authority Building, 05 Search Intelligence & Performance Analyst, 06 Product Knowledge & Media Intelligence, 07 WordPress Operations & Content Governance, 08 Paid Ads & Performance Marketing.

## Aturan domain

### Peran orchestrator

- Orchestrator Utama dan Orchestrator Divisi hanya mengoordinasi: memecah pekerjaan, memilih lapisan di bawahnya, dan tidak mengerjakan tugas teknis sendiri. Lihat ADR 0001.
- Orchestrator juga menjadi teman diskusi saat sebuah lapisan terasa berat atau bloated dan perlu dipecah:
  - Orchestrator Utama dapat memecah Orchestrator Divisi yang bloated.
  - Orchestrator Divisi dapat memecah Agen Spesialis yang bloated.

### Pemecahan lapisan yang bloated

- Gejala bloated (pemicu diskusi, bukan pemicu otomatis; angka bisa disetel nanti):
  1. Instruksi/jobdesk agen melebihi ±2.000 kata.
  2. Satu agen menangani lebih dari ±5 jenis tugas berbeda.
  3. Tingkat gagal atau revisi agen terus naik.
- Hasil yang error atau tidak sesuai keinginan harus dievaluasi lagi.
- Pemecahan diputuskan lewat diskusi antara Orchestrator Utama dan Orchestrator Divisi; **tidak** butuh persetujuan Pemilik.
- Setelah agen baru bertugas, Orchestrator Utama mengirim Laporan ke Pemilik: agen apa, bertugas apa.
- `terbuka`: pemecahan menghasilkan perubahan definisi agen; ini bertabrakan dengan aturan "merge ke branch utama butuh persetujuan" (lihat R4).

### Persetujuan Pemilik

Tindakan berikut wajib menunggu Permintaan Persetujuan ke Pemilik lewat Delegator sebelum dijalankan:

1. Deploy ke production (Agen 11 Release/DevOps).
2. Perubahan skema database production (Agen 05 Database).
3. Menulis ke Odoo production (Agen 08 Odoo Integration).
4. Merge ke branch utama.

Tindakan lain boleh berjalan otomatis.

### Tugas dan status

Satu Perintah menjadi satu Tugas; Tugas dipecah menjadi Sub-tugas.

Status Tugas:

```
diterima → diurai → dikerjakan → menunggu persetujuan → selesai / gagal / dibatalkan
```

### Kontrak Delegator ↔ Orchestrator Utama

Hanya tiga jenis pesan:

| Pesan | Arah |
|---|---|
| Perintah | Delegator → Orchestrator Utama |
| Laporan | Orchestrator Utama → Delegator |
| Permintaan Persetujuan | Orchestrator Utama → Delegator (jawaban: setuju / tolak) |

Kanal WhatsApp/Telegram adalah urusan Delegator.

### Runtime dan penyimpanan

- Agen dijalankan dengan Claude Agent SDK di Node.js.
- v1: satu proses — Orchestrator Utama memanggil Orchestrator Divisi dan Agen Spesialis sebagai subagent.
- Penyimpanan status Tugas: PostgreSQL (pilihan agen atas penyerahan Pemilik; lihat R3-Q4).
- Hosting: ditunda; kandidat VPS, Vercel, atau lokal. Bukan Replit.

## Daftar keputusan

| No | Keputusan | Kutipan jawaban Pemilik | Status |
|---|---|---|---|
| R1-Q1 | Fokus sekarang: Orchestrator (dari ide My Second Brain + Orchestrator) | "Saya ingin membuat My Second Brain dan Orchestartor. Saat ini saya ingin membuat orchestartor" | disepakati |
| R1-Q2 | Target sesi: spesifikasi siap implementasi | "C" | disepakati |
| R1-Q3 | Catatan di repo stepsans/mysecondbrain | "https://github.com/stepsans/mysecondbrain.git" | disepakati |
| R2-Q1 | Lingkup: Orchestrator Utama + Orchestrator Divisi Software Factory; agen spesialis dan Digital Growth ditunda | "A" | disepakati |
| R2-Q2 | Nama lapisan: Delegator → Orchestrator Utama → Orchestrator Divisi → Agen Spesialis; satu Orchestrator Utama per Perusahaan; mulai dari satu Perusahaan | "Betul. Delegator >> Orchestrator Utama >> Orchestratro Divisi >> Agen Spesialis. Maxipro nama perusahaan. Setiap perusahaan mempunyai orchestrator utama. Tapi saya mau membangun untuk 1 perusahaan dulu" | disepakati |
| R2-Q3 | Orchestrator hanya memecah/mengoordinasi, plus teman diskusi untuk memecah lapisan yang bloated | "Iya betul, hanya memecah. dan menjadi teman diskusi apabila terasa berat dan bloated harus bikin agent dan pecah agent. Orchestrator utama bisa memecah orchestrator divisi yang bloated. Orchestrator divisi bisa pecah agent yang bloated." | disepakati |
| R2-Q4 | Empat titik persetujuan Pemilik | "ok pakai daftar di atas saya ngikut." | disepakati |
| R2-Q5 | Runtime: Claude Agent SDK di Node.js | "A, tidak di replit tetapi bisa di VPS/Vercel/Lokal. Saya masih belum tahu" | disepakati |
| R2-Q6 | Struktur catatan: GLOSSARY.md, docs/adr/, docs/spec/orchestrator.md, di branch baru | "Iya saya setuju" | disepakati |
| R3-Q1 | Pemecahan agen diputuskan antar-orchestrator tanpa persetujuan Pemilik; Pemilik cukup diberi info agen baru dan tugasnya | "ya, saya selalu setuju ke orchestrator. Biarkan mereka berdiskusi sendiri antara orchestrator utama dan orchestrator divisi. Jadi tidak perlu menanyakan persetujuan saya, yang penting kasih info aja kalo ada agent baru bertugas apa dll." | disepakati |
| R3-Q2 | Tiga gejala bloated sebagai pemicu diskusi; hasil error/tidak sesuai dievaluasi lagi | "pakai ketiga gejala itu sebagai pemicu diskusi, bukan pemicu otomatis. Angkanya bisa disetel nanti. Kalau error dan hasil tidak sesuai diinginkan harus dievaluasi lagi." | disepakati |
| R3-Q3 | Istilah Tugas/Sub-tugas dan alur status | "Setuju" | disepakati |
| R3-Q4 | Pilihan penyimpanan diserahkan ke agen | "Saya tidak tau, coba tanya ke Bos Arga. Mungkin dia lebih mengerti. Saya ikut aja. Kasih saya rekomendasinya" | disepakati |
| R3-Q5 | v1 satu proses, subagent Claude Agent SDK | "A Satu proses: Orchestrator Utama memanggil Orchestrator Divisi dan agen sebagai subagent di Claude Agent SDK." | disepakati |
| R3-Q6 | Kontrak tiga jenis pesan | "Ya pakai tiga jenis pesan ini" | disepakati |
| R3-Q7 | Tulis ADR 0001 orchestrator hanya koordinasi | "Ya" | disepakati |
| R3-Q8 | Hosting | "ya ditunda" | ditunda |

## Alternatif yang ditolak

- R2-Q1: pola generik tanpa divisi (b) dan semua lapisan plus 21 agen sekaligus (c). Pemilik tidak memberi alasan.
- R2-Q5: n8n (b), LangGraph/CrewAI (c), menunda (d). Pemilik tidak memberi alasan. Hosting di Replit juga ditolak, tanpa alasan.
- R3-Q1: usulan agen agar pemecahan agen menjadi titik persetujuan ke-5. Alasan Pemilik: "saya selalu setuju ke orchestrator".

## Catatan inferensi

- Lingkup R1-Q2 (siap implementasi) kemungkinan butuh lebih dari 5 round (inferensi agen).
- R3-Q4: pilihan agen adalah PostgreSQL — sesuai stack Pemilik dan aman untuk penulisan bersamaan oleh beberapa agen (inferensi agen). Pemilik menyebut pilihan ini bisa ditanyakan ke Bos Arga; belum ditinjau.
