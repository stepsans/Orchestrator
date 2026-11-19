# Riwayat sesi perancangan

Catatan kronologis semua percakapan perancangan Orchestrator, dari pemasangan skill sampai sesi 6. Keputusan resmi beserta kutipan persis jawaban Pemilik ada di **Daftar keputusan** `docs/spec/orchestrator.md`; dokumen ini menceritakan alurnya supaya pembaca baru paham *kenapa* keputusan itu diambil.

Semua sesi memakai skill **kukatanya** (wawancara bernomor dengan rekomendasi, maksimal 5 round per lingkup, jawaban "ya" yang mencakup beberapa nomor dipecah, catatan di glossary / ADR / satu dokumen spesifikasi). Sumber skill: https://github.com/madearga/kukatanya-skill.

---

## 0. Persiapan (2026-11-13)

- Skill kukatanya dipasang dari repo `madearga/kukatanya-skill`.
- Ide awal Pemilik: membangun **My Second Brain** dan **Orchestrator**; yang dikerjakan dulu: Orchestrator.
- Rancangan awal Pemilik (dua cabang di bawah "Orchestrator Maxipro"):
  - **Software Factory**: 01 Triage/Intake, 02 Product/Business Analyst, 03 Architecture, 04 Backend/API, 05 Database, 06 Web/UI, 07 Mobile, 08 Odoo Integration, 09 QA/Testing, 10 Security, 11 Release/DevOps, 12 Observability/SRE, 13 Documentation.
  - **Digital Growth**: 01 Technical SEO & Website Optimization, 02 SEO & AEO Content Strategist, 03 GEO & AI Visibility Specialist, 04 Digital PR & Authority Building, 05 Search Intelligence & Performance Analyst, 06 Product Knowledge & Media Intelligence, 07 WordPress Operations & Content Governance, 08 Paid Ads & Performance Marketing.

## 1. Sesi 1 (2026-11-13), 10 round

Lingkup: Orchestrator Utama + Orchestrator Divisi Software Factory; target "spesifikasi siap implementasi".

Garis besar yang disepakati:
- **Nama lapisan**: Delegator → Orchestrator Utama (satu per Perusahaan, mulai Maxipro) → Orchestrator Divisi → Agen Spesialis.
- **Orchestrator hanya mengoordinasi** (ADR 0001) dan menjadi teman diskusi untuk memecah lapisan yang bloated; pemecahan diputuskan antar-orchestrator tanpa persetujuan Pemilik, cukup dilaporkan dengan alasannya.
- Tiga gejala bloated (instruksi > ±2.000 kata, > ±5 jenis tugas, tingkat gagal naik) sebagai pemicu diskusi, bukan otomatis.
- **Titik persetujuan** awal: deploy production, skema DB production, tulis Odoo production, merge ke branch utama (kecuali `agents/`), lalu batas keras biaya.
- **Tugas / Sub-tugas** dan status; **Kriteria Penerimaan** ditulis Triage; maksimal 2 kali ulang.
- Software Factory: selalu mulai dari 01 Triage, pemilihan agen dinamis, 09 QA wajib sebelum merge, 10 Security untuk perubahan sensitif.
- **Kontrak Delegator ↔ Orchestrator Utama**: Perintah, Laporan, Permintaan Persetujuan, lalu Pertanyaan Klarifikasi (pesan ke-4).
- Runtime Claude Agent SDK di Node.js, satu proses (ADR 0002); PostgreSQL; kode di `src/`, kartu di `agents/`, daftar Proyek di `proyek/`.
- Antrean satu per satu per divisi, paralel antar-divisi; batas biaya per Tugas; rahasia di environment variable; bot Telegram sebagai pengganti Delegator.
- Sesi pertama sempat ditutup di round 5, lalu Pemilik memilih lanjut sampai round 10.

## 2. Sesi 2 (2026-11-16), 5 round: menutup yang ditunda

- Pemilik bertanya kenapa banyak yang ditunda; penyebabnya: fakta belum ada, pilihan Pemilik sendiri, dan batas lingkup.
- Repo Proyek diverifikasi: Maxichat = `stepsans/MaxiChat.App` (bukan `stepsans/MaxiChat`), Maxione = `stepsans/maxione`.
- Struktur folder diberi level Perusahaan (`agents/maxipro/...`, `proyek/maxipro/...`); setiap Perusahaan punya divisi sendiri.
- Hosting: Pemilik ingin mencoba Vercel → dibahas batas waktu fungsi Vercel → hybrid → akhirnya **lokal di komputer Pemilik**, dikemas `docker-compose.yml`, bisa pindah ke Mac 24/7 atau VPS. Bot Telegram long polling di mesin yang sama. Dashboard menyusul.
- Kartu agen disimpan di folder `agents/` sendiri (bukan `.claude/agents/`).
- Brief 6 agen v1 (01, 03, 04, 06, 09, 11).
- Selama di lokal: agen bekerja selama komputer menyala; Tugas terputus dilanjutkan otomatis.

## 3. Sesi 3 (2026-11-16), 10 round: semua 13 agen di v1

- Pemilik menanyakan kenapa 7 agen "menyusul" → diputuskan **ke-13 agen dibangun bersama di v1**.
- Fakta dari repo Maxichat: mobile Expo ~54 / React Native 0.81.5; migrasi SQL idempoten di `docs/migrations/`; test `*.test.ts` vs `*.dbtest.ts` dengan `TEST_DATABASE_URL`; `pnpm run verify` sebagai gerbang rilis; `CLAUDE.md` dan `AGENTS.md` sendiri.
- Fakta dari repo Maxione: aplikasi internal kantor Maxipro, production di Hostinger VPS (maxione.click), backup harian.
- Kedua repo **tidak punya staging** → **staging lokal** di Docker, tanpa kredensial production pihak ketiga, Odoo hanya dibaca, database test.
- Pemilik mengirim **draf lengkap ke-13 kartu agen** (teks asli di `agents/maxipro/software-factory/_draf-pemilik/`); setiap selisih dengan brief diperdebatkan dan dikoreksi (lihat README di folder itu).
- Titik persetujuan ke-6: keputusan arsitektur besar. Rollback darurat tanpa persetujuan. P0 menjeda Tugas berjalan.
- Istilah **Maxione**: Super Apps Maxipro; Maxichat aplikasi sendiri multi-tenant yang terhubung (setiap user Maxione punya Maxichat, tidak sebaliknya). Kartu agen ditulis netral ("the Company").

## 4. Spesifikasi implementasi (2026-11-16)

Pemilik meminta spesifikasi rinci, hambatan, penerapan yang tahan uji, dan perkembangan → `docs/spec/implementasi-v1.md`. Temuan cek ulang:
- Dokumentasi Agent SDK: subagent tidak bisa memanggil subagent lain; perilaku nesting CLI berubah antar versi.
- Agent SDK untuk produk harus memakai API key, bukan login langganan claude.ai.
- Agent SDK mendukung structured outputs (JSON Schema).
- `deploy.sh` Maxichat/Maxione belum mendukung deploy ke tag; Maxichat butuh mode aman untuk staging.

## 5. Sesi 4 (2026-11-17), 8 round: keputusan K1–K9

- Fokus ditegaskan: **membangun Orchestrator**; Maxichat dan Maxione hanya konteks, tidak disentuh.
- K1 orkestrasi diatur kode, tiap agen dipanggil `query()` sendiri (ADR 0003); K2 status `dijeda`; K3 golden Tugas sebelum kartu berubah; K4 `ops-gateway`; K8 anggaran (US$50/bulan, bisa diatur); K9 PIN. K5/K6/K7 ditunda sebagai syarat sebelum Proyek nyata (Pemilik minta diingatkan saat Orchestrator jadi).
- Repo latihan `orchestrator-sandbox`.
- Pemilik ingin model tidak harus Claude (Codex, Magpie) dan tidak selalu API key → **colokan `AgentRunner`** multi-penyedia; cara login dipilih di awal F1. Kode Orchestrator dibangun dengan Claude Code memakai akun Pro.
- Magpie (menurut Pemilik): gateway lokal untuk AI coding agent dan catatan lintas-platform (Claude & ChatGPT).

## 6. Sesi 5 (2026-11-17 s.d. 2026-11-19), 4 round: membuat software baru dari nol

- **Rencana Proyek** untuk software baru; titik persetujuan ke-7; repo baru dibuat otomatis.
- Hosting software baru: **Vercel** default + PostgreSQL terkelola, VPS untuk proses berjalan terus, Dockerfile sejak awal; preview deploy sebagai staging.
- **Stack Dasar** dibahas mendalam (Next.js vs stack Maxichat/Maxione vs Hono): dipilih stack yang sama dengan Maxichat/Maxione (React + Vite, Express 5, OpenAPI + Orval, Zod, Drizzle, PostgreSQL, pnpm, Node.js 24 LTS), Hono/Next.js boleh per proyek. Template di `orchestrator-sandbox`.
- Lampiran Telegram (screenshot, berkas, voice note dikonfirmasi jadi teks); kunci rahasia dimasukkan Pemilik sendiri.

## 7. Sesi 6 (2026-11-19), 3 round: Delegator diskusi dan rilis

- **Delegator v1 bisa diajak diskusi** dan bertanya kritis seperti skill kukatanya; mode langsung `/jalan`; Sesi Diskusi tersimpan (`/lanjut`); dibangun di F1.
- Merge otomatis untuk software yang belum rilis; daftar "siap produksi" sebelum rilis pertama; bug dari pengguna diteruskan Pemilik.

## 8. Artefak lain

- Halaman HTML ringkasan yang di-share: `docs/halaman/orchestrator-maxipro.html` (versi setelah sesi 3; belum memuat sesi 4–6).

## 9. Langkah berikutnya (per 2026-11-19)

1. Menulis 13 kartu agen final (draf Pemilik + koreksi) dan review Pemilik.
2. 20 contoh Perintah nyata dari Pemilik untuk golden Tugas.
3. Fase F0 (Fondasi).
4. Satu ide software nyata untuk menguji Rencana Proyek di F3.
