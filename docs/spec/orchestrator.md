# Spesifikasi: Orchestrator

Status sesi: berjalan (sesi kukatanya, dimulai 2026-11-13).

## Tempat catatan

| Catatan | Tempat |
|---|---|
| Istilah | `GLOSSARY.md` (satu konteks: Orchestrator) |
| Keputusan besar | `docs/adr/` (dibuat saat ADR pertama disetujui) |
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

- Orchestrator Utama dan Orchestrator Divisi hanya mengoordinasi: memecah pekerjaan, memilih lapisan di bawahnya, dan tidak mengerjakan tugas teknis sendiri.
- Orchestrator juga menjadi teman diskusi Pemilik saat sebuah lapisan terasa berat atau bloated dan perlu dipecah:
  - Orchestrator Utama dapat memecah Orchestrator Divisi yang bloated.
  - Orchestrator Divisi dapat memecah Agen Spesialis yang bloated.

### Persetujuan Pemilik

Tindakan berikut wajib menunggu persetujuan Pemilik lewat Delegator sebelum dijalankan:

1. Deploy ke production (Agen 11 Release/DevOps).
2. Perubahan skema database production (Agen 05 Database).
3. Menulis ke Odoo production (Agen 08 Odoo Integration).
4. Merge ke branch utama.

Tindakan lain boleh berjalan otomatis.

### Runtime

- Agen dijalankan dengan Claude Agent SDK di Node.js.
- Hosting: belum ditentukan; kandidat VPS, Vercel, atau lokal. Bukan Replit.

## Daftar keputusan

| No | Keputusan | Kutipan jawaban Pemilik | Status |
|---|---|---|---|
| R1-Q1 | Fokus sekarang: Orchestrator (dari ide My Second Brain + Orchestrator) | "Saya ingin membuat My Second Brain dan Orchestartor. Saat ini saya ingin membuat orchestartor" | disepakati |
| R1-Q2 | Target sesi: spesifikasi siap implementasi | "C" | disepakati |
| R1-Q3 | Catatan di repo stepsans/mysecondbrain | "https://github.com/stepsans/mysecondbrain.git" | disepakati |
| R2-Q1 | Lingkup: Orchestrator Utama + Orchestrator Divisi Software Factory; agen spesialis dan Digital Growth ditunda | "A" | disepakati |
| R2-Q2 | Nama lapisan: Delegator → Orchestrator Utama → Orchestrator Divisi → Agen Spesialis; satu Orchestrator Utama per Perusahaan; mulai dari satu Perusahaan | "Betul. Delegator >> Orchestrator Utama >> Orchestratro Divisi >> Agen Spesialis. Maxipro nama perusahaan. Setiap perusahaan mempunyai orchestrator utama. Tapi saya mau membangun untuk 1 perusahaan dulu" | disepakati |
| R2-Q3 | Orchestrator hanya memecah/mengoordinasi, plus teman diskusi untuk memecah lapisan yang bloated | "Iya betul, hanya memecah. dan menjadi teman diskusi apabila terasa berat dan bloated harus bikin agent dan pecah agent. Orchestrator utama bisa memecah orchestrator divisi yang bloated. Orchestrator divisi bisa pecah agent yang bloated." | disepakati |
| R2-Q4 | Empat titik persetujuan Pemilik (lihat Aturan domain) | "ok pakai daftar di atas saya ngikut." | disepakati |
| R2-Q5 | Runtime: Claude Agent SDK di Node.js | "A, tidak di replit tetapi bisa di VPS/Vercel/Lokal. Saya masih belum tahu" | disepakati |
| R2-Q5 | Hosting (VPS / Vercel / lokal) | "A, tidak di replit tetapi bisa di VPS/Vercel/Lokal. Saya masih belum tahu" | terbuka |
| R2-Q6 | Struktur catatan: GLOSSARY.md, docs/adr/, docs/spec/orchestrator.md, di branch baru | "Iya saya setuju" | disepakati |

## Alternatif yang ditolak

- R2-Q1: pola generik tanpa divisi (b) dan semua lapisan plus 21 agen sekaligus (c). Pemilik tidak memberi alasan.
- R2-Q5: n8n (b), LangGraph/CrewAI (c), menunda (d). Pemilik tidak memberi alasan. Hosting di Replit juga ditolak, tanpa alasan.

## Catatan inferensi

- Lingkup R1-Q2 (siap implementasi) kemungkinan butuh lebih dari 5 round (inferensi agen).
