# Orchestrator

Rancangan sistem agen AI milik Pemilik (Maxipro): **Delegator → Orchestrator Utama → Orchestrator Divisi → Agen Spesialis**. Fokus v1: **Orchestrator Software Factory** yang nantinya membuat banyak software siap produksi. Sebagian software hasilnya terhubung ke Maxichat atau Maxione, sebagian tidak. Maxichat dan Maxione sendiri **tidak disentuh** di v1.

Status: **perancangan selesai (6 sesi), belum ada kode.** Langkah berikutnya: fase F0 (Fondasi).

## Cara membaca

| Berkas | Isi |
|---|---|
| [`GLOSSARY.md`](GLOSSARY.md) | Kamus istilah (Pemilik, Delegator, Orchestrator Utama, Tugas, Kriteria Penerimaan, Rencana Proyek, Stack Dasar, Sesi Diskusi, Maxione, dan lainnya) |
| [`docs/spec/orchestrator.md`](docs/spec/orchestrator.md) | **Sumber kebenaran**: semua aturan dan **daftar keputusan** lengkap dengan kutipan persis jawaban Pemilik |
| [`docs/spec/implementasi-v1.md`](docs/spec/implementasi-v1.md) | Rancangan teknis: arsitektur, state machine, skema database, kontrak pesan dan output agen, guardrail, pengujian, hambatan, fase F0–F6, perkembangan |
| [`docs/adr/`](docs/adr/) | Keputusan besar: 0001 orchestrator hanya koordinasi, 0002 satu proses, 0003 orkestrasi diatur kode |
| [`docs/riwayat-sesi.md`](docs/riwayat-sesi.md) | Cerita kronologis sesi 1–6: kenapa tiap keputusan diambil |
| [`docs/sumber.md`](docs/sumber.md) | Sumber fakta teknis yang dirujuk |
| [`agents/maxipro/software-factory/_draf-pemilik/`](agents/maxipro/software-factory/_draf-pemilik/) | Teks asli draf ke-13 kartu agen dari Pemilik + daftar koreksi yang wajib diterapkan |
| [`docs/halaman/orchestrator-maxipro.html`](docs/halaman/orchestrator-maxipro.html) | Halaman ringkasan visual (versi setelah sesi 3) |

## Ringkasan desain

- **Delegator** (bot Telegram, akun pribadi Pemilik): bisa diajak diskusi kritis atau langsung `/jalan`.
- **Orchestrator** hanya mengoordinasi; alur diatur kode, setiap agen dipanggil terpisah lewat colokan `AgentRunner` (Claude Agent SDK atau gateway seperti Magpie).
- **13 Agen Spesialis** Software Factory, semuanya di v1.
- **7 titik persetujuan** Pemilik: deploy production, skema DB production, tulis Odoo, merge (setelah rilis pertama), lewat batas biaya, arsitektur besar, Rencana Proyek software baru. Tindakan berisiko hanya lewat `ops-gateway`; PIN untuk deploy dan migrasi production.
- **Stack Dasar** software baru: React + Vite, Express 5, OpenAPI + Orval, Zod, Drizzle, PostgreSQL, pnpm, Node.js 24 LTS. Hosting default Vercel, VPS bila perlu.
- Orchestrator v1 berjalan **lokal** dengan `docker-compose.yml`, bisa pindah ke Mac 24/7 atau VPS.

## Yang sengaja ditunda

- Cara login ke penyedia AI (awal F1, setelah Magpie dicek).
- Akun Vercel dan domain (F3).
- K5 akun Odoo read-only, K6 deploy ke tag, K7 `SAFE_MODE` Maxichat (sebelum menyentuh Proyek nyata; Pemilik diingatkan saat F4 lulus).
- Divisi Digital Growth (sesi tersendiri).
