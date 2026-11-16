# Spesifikasi: Orchestrator

Status sesi: selesai (sesi 1: 2026-11-13; sesi 2: 2026-11-16). Nomor pertanyaan sesi 2 memakai awalan S2-. Sesi 3 (2026-11-16, awalan S3-): semua 13 agen Software Factory masuk v1; menunggu konfirmasi akhir.

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
- Pemecahan diputuskan lewat diskusi antara Orchestrator Utama dan Orchestrator Divisi; **tidak** butuh persetujuan Pemilik. Ini berlaku untuk memecah Agen Spesialis maupun memecah Orchestrator Divisi.
- Setelah diskusi matang, Orchestrator Utama mengirim Laporan ke Pemilik berisi hasil diskusi dan alasan pemecahan: agen apa, bertugas apa.
- Definisi Agen disimpan di repo, di folder `agents/`. Perubahan yang **hanya** menyentuh `agents/` di-merge otomatis lalu dilaporkan; merge yang menyentuh berkas lain tetap butuh persetujuan Pemilik.

### Evaluasi hasil

1. Saat Tugas diurai, 01 Triage menulis Kriteria Penerimaan.
2. Orchestrator Divisi mencocokkan hasil tiap Sub-tugas dengan Kriteria Penerimaan.
3. Jika tidak cocok, Sub-tugas dikembalikan ke agen dengan umpan balik, maksimal 2 kali ulang (angka usulan, bisa disetel).
4. Jika masih gagal, Tugas berstatus `gagal` dan Pemilik menerima Laporan berisi penyebabnya.
5. Setiap kegagalan dihitung ke gejala bloated ke-3.

### Alur di Software Factory

- Setiap Tugas selalu dimulai dari 01 Triage.
- Sesudahnya Orchestrator Divisi memilih Agen Spesialis yang dibutuhkan secara dinamis (bukan pipeline tetap).
- 09 QA selalu dijalankan sebelum merge.
- 10 Security wajib untuk perubahan yang menyentuh autentikasi, data pelanggan, atau kredensial.

### Perintah di luar divisi yang ada

- Perintah untuk bidang yang belum punya Orchestrator Divisi disimpan sebagai Tugas berstatus `diterima`, tidak dipaksakan ke divisi lain.
- Membuat Orchestrator Divisi baru:
  1. Pemilik mengirim Perintah lewat Delegator (misalnya "buat divisi Digital Growth").
  2. Orchestrator Utama merancang Orchestrator Divisi baru beserta Agen Spesialisnya, sebagai perubahan di `agents/` (langsung berlaku).
  3. Orchestrator Utama mengirim Laporan ke Pemilik.
  4. Tugas berstatus `diterima` yang menunggu divisi itu otomatis dilanjutkan.

### Proyek

- Daftar Proyek disimpan di repo ini, dikelompokkan per Orchestrator Divisi, satu folder per Proyek. Contoh: `proyek/maxipro/software-factory/maxichat/`, `proyek/maxipro/software-factory/maxione/`. Setiap Proyek aplikasi baru mendapat folder sendiri. Nama folder divisi sama dengan di `agents/`.
- Tiap folder Proyek berisi `PROYEK.md`: URL repo dan branch utama, lingkungan (staging/production) dan URL-nya, stack, perintah menjalankan test, cara deploy.
- Repo Proyek: Maxichat = `stepsans/MaxiChat.App`; Maxione = `stepsans/maxione` (keduanya terverifikasi ada di akun GitHub Pemilik, 2026-11-16). Repo `stepsans/MaxiChat` tidak dipakai.
- 01 Triage menentukan Proyek yang dimaksud sebuah Perintah; bila tidak jelas, Orchestrator Utama mengirim Pertanyaan Klarifikasi lewat Delegator.

### Data di PostgreSQL

| Tabel | Kolom utama |
|---|---|
| `tugas` | id, perusahaan, divisi, proyek, perintah asli, Kriteria Penerimaan, prioritas (P0–P3), status, waktu dibuat/diubah |
| `sub_tugas` | id, tugas_id, agen, input, hasil, status, jumlah ulang, umpan balik |
| `permintaan_persetujuan` | id, tugas_id, tindakan, alasan, dampak, jawaban (setuju / tolak / belum), waktu diminta/dijawab |
| `laporan` | id, tugas_id, jenis, isi, waktu kirim |
| `pertanyaan_klarifikasi` | id, tugas_id, pertanyaan, jawaban, waktu ditanya/dijawab |

Status Sub-tugas: `antre → dikerjakan → dievaluasi → lolos / diulang / gagal`. Detail kolom disempurnakan saat implementasi.

### Permintaan Persetujuan yang belum dijawab

- Tugas tetap `menunggu persetujuan`; sistem tidak pernah menyetujui otomatis.
- Delegator mengingatkan Pemilik setelah 24 jam, lalu 72 jam (angka usulan).
- Selama menunggu, antrean divisi itu dilepas; Tugas berikutnya boleh mulai. Pekerjaan yang menunggu tersimpan di branch-nya sendiri.

### Definisi Agen

- Satu berkas markdown per agen: `agents/<perusahaan>/<divisi>/<nomor-nama>.md`, contoh `agents/maxipro/software-factory/01-triage.md`.
- Bagian atas: nama, divisi, jobdesk singkat, tool yang boleh dipakai, model. Bagian bawah: instruksi lengkap.
- Orchestrator Divisi: `agents/<perusahaan>/<divisi>/_orchestrator.md`.
- Fakta (dicek 2026-11-16, dokumentasi Claude Agent SDK): subagent bisa didefinisikan secara programatis lewat parameter `agents` di opsi `query()`, atau sebagai berkas markdown ber-YAML frontmatter di `.claude/agents/`.
- Definisi Agen disimpan di folder `agents/` sendiri; program membacanya lalu meneruskannya ke parameter `agents` SDK. Nama kolom frontmatter mengikuti SDK (`name`, `description`, `tools`, `model`) ditambah `perusahaan` dan `divisi`.
- Perubahan yang hanya menyentuh `agents/` tidak lewat 09 QA; diperiksa dengan pengecekan format otomatis (frontmatter valid, kolom wajib lengkap) dan dibaca ulang Orchestrator Utama. Laporan menyertakan ringkasan perubahan.

### Ingatan antar-Tugas

- Setiap Tugas mulai dengan konteks bersih; orchestrator boleh membaca ringkasan beberapa Tugas terakhir dari PostgreSQL bila relevan.
- Tiap Orchestrator Divisi punya Catatan Divisi di `agents/<perusahaan>/<divisi>/_catatan.md` (ikut merge otomatis `agents/`). Pelajaran khusus satu Proyek ditulis di berkas yang sama di bawah judul nama Proyek.

### Rahasia dan kredensial

- Token GitHub, akses Odoo, kunci deploy, dan API key Claude disimpan sebagai environment variable di server, tidak pernah di repo.
- Agen tidak pernah menulis rahasia ke repo, Laporan, atau Catatan Divisi.

### Batas biaya per Tugas

- Batas lunak ±US$2 per Tugas: lewat batas, Pemilik menerima Laporan.
- Batas keras ±US$10 per Tugas: Tugas berhenti dan Permintaan Persetujuan dikirim untuk melanjutkan.
- Angka usulan, disetel setelah ada data pemakaian.

### Isi v1 Software Factory

- Semua 13 Agen Spesialis Software Factory dibangun bersama di v1 (S3-R1-Q1; menggantikan R7-Q5).
- Brief agen 02, 05, 07, 08, 10, 12, 13: disetujui Pemilik beserta draf Definisi Agen lengkap yang ia lampirkan (berbahasa Inggris); koreksi disepakati di S3-R3 (lihat Aturan tambahan sesi 3).

### Fakta Proyek Maxione (dicek 2026-11-16 dari repo `stepsans/maxione`, branch `main`)

- README: aplikasi internal kantor Maxipro; setelah login user masuk ke beranda berisi menu fitur, salah satunya Katalog Produk (login OTP email, data dari Google Sheets, harga per tier sesuai role).
- Production di Hostinger VPS (maxione.click), bukan Replit (`deploy/DEPLOY.md`).
- Stack: monorepo pnpm, Node.js 24, TypeScript 5.9, Express 5, PostgreSQL + Drizzle, Zod, Orval.

### Aturan tambahan sesi 3

- Sebelum bekerja di sebuah Proyek, setiap agen wajib membaca dan mematuhi `CLAUDE.md` dan `AGENTS.md` Proyek itu. Untuk urusan teknis kode, aturan Proyek menang; aturan persetujuan sistem ini tetap berlaku.
- `docker-compose.yml` menyertakan satu database test sekali-pakai yang dikosongkan setiap kali test dijalankan (untuk `*.dbtest.ts` dan uji migrasi).
- Rollback darurat: bila 12 Observability mendeteksi error parah setelah deploy, 11 Release/DevOps boleh langsung rollback ke versi sebelumnya tanpa menunggu persetujuan, lalu segera mengirim Laporan. Pengecualian dari titik persetujuan ke-1.
- Agen 08 hanya membaca dari Odoo production; semua penulisan ke Odoo lewat Permintaan Persetujuan (titik ke-3). Odoo staging belum ada; Pemilik akan menyiapkannya bila diperlukan.
- Hanya Pemilik yang memberi Perintah; "authorized employees" dihapus dari brief 02 untuk v1. "Product Owner" di brief berarti Pemilik.
- Agen tidak pernah menghubungi Pemilik langsung; semua lewat Orchestrator Divisi → Orchestrator Utama → Delegator, sebagai Pertanyaan Klarifikasi, Permintaan Persetujuan, atau Laporan.
- Agen 12 tidak membuat Tugas sendiri: ia mengirim Laporan insiden; insiden parah boleh di-rollback darurat oleh 11; perbaikan kode menunggu Perintah Pemilik.
- Agen 12 di v1 memantau ±30 menit setelah deploy; pemantauan terus-menerus aktif setelah sistem pindah ke perangkat 24/7 atau VPS.
- Kartu agen berbahasa Inggris dengan aturan tambahan: semua yang sampai ke Pemilik ditulis dalam Bahasa Indonesia.
- KPI di kartu agen adalah arah kerja; pengukurannya menunggu dashboard (setelah v1).
- Agen 07 mengirim build uji lewat Laporan; uji di perangkat sungguhan dilakukan Pemilik atau tim.
- Draf brief Pemilik untuk agen 02, 05, 07, 08, 10, 12, 13 disimpan sebagai draf Definisi Agen di `agents/maxipro/software-factory/` dengan koreksi di atas, setelah sesi selesai.
- Koreksi draf 6 agen (S3-R6): Triage hanya menerima Tugas yang berasal dari Perintah Pemilik lewat Delegator; Triage wajib menentukan Proyek, menulis Kriteria Penerimaan, menandai sensitif, mengusulkan Sub-tugas, dan membuat Pertanyaan Klarifikasi; klasifikasi BUG/FEATURE/REVISION/DATA ISSUE/SECURITY INCIDENT/INFRASTRUCTURE INCIDENT, severity P0–P3, dan cek duplikat dipertahankan. Triage mengumpulkan bukti dan menandai "belum direproduksi"; 09 QA yang mereproduksi di staging lokal. 03 menghasilkan rencana teknis per Tugas; dokumen arsitektur menyeluruh hanya diperbarui saat perubahan besar, disimpan di repo Proyek. 04/06 hanya bekerja di branch Tugas dan tidak pernah memegang kredensial production. 09 boleh menulis berkas test; keputusan BLOCKED → Pertanyaan Klarifikasi (`menunggu klarifikasi`). Code review v1 = QA PASS + Security (bila perlu) + persetujuan merge Pemilik; tanpa agen code review. 11 wajib backup database sebelum migrasi production. Semua kartu: "the Company", "Task" (bukan Ticket), "Product Owner" = Pemilik, komunikasi ke manusia lewat Orchestrator.
- Nama "Maxione" / "Maxipro Super Apps" khusus Maxipro; tidak dipakai saat sistem diduplikasi ke Perusahaan lain.
- Hubungan Proyek: Maxione adalah aplikasi utama internal, hanya dipakai Maxipro. Maxichat adalah aplikasi berdiri sendiri dengan banyak tenant, terhubung dengan Maxione. Setiap user Maxione pasti punya Maxichat; setiap tenant Maxichat belum tentu punya Maxione.
- Software internal Maxipro yang baru menjadi menu baru di repo Maxione, bukan repo terpisah.
- Kartu agen ditulis netral ("the Company", "the Company's applications"); hal khusus Maxipro ditaruh di `PROYEK.md` dan Catatan Divisi Maxipro, supaya kartu bisa diduplikasi ke Perusahaan lain tanpa diedit.
- Draf kartu Pemilik untuk agen 01, 03, 04, 06, 09, 11 diterima (2026-11-16); selisihnya dengan brief yang disepakati sedang dibahas (S3-R6).
- Fakta (dicek 2026-11-16): repo Maxichat dan Maxione tidak menyebut lingkungan staging. Maxione: production di VPS Hostinger lewat `deploy/deploy.sh` (git pull, build, restart API), dengan backup harian dari `setup-server.sh`. Maxichat: `./deploy.sh` di VPS.

### Fakta Proyek Maxichat (dicek 2026-11-16 dari repo `stepsans/MaxiChat.App`, branch `main`)

- Monorepo pnpm. Bagian di `artifacts/`: `admin`, `api-server` (Express + drizzle-orm + PostgreSQL), `desktop`, `maxichat-video`, `mobile`, `mockup-sandbox`, `whatsapp-ai`.
- Mobile: Expo `~54.0.27`, React Native `0.81.5` (`artifacts/mobile/package.json`).
- Repo punya `AGENTS.md` dan `CLAUDE.md` sendiri; `AGENTS.md` merujuk pekerjaan berjalan integrasi Odoo di `docs/odoo-integration-brief.md`.
- Maxichat punya halaman Synchronize (platform ↔ MAXIONE, khusus operator), menurut `docs/odoo-integration-brief.md`.
- Odoo: Odoo 18 Community milik Maxipro, di-host di AWS (menurut `docs/odoo-integration-brief.md`). Dokumen itu tidak menyebut Odoo staging.

### Laporan

- Laporan dikirim hanya saat status Tugas berubah, saat ada agen baru, atau saat Tugas gagal. Perubahan per Sub-tugas tidak dilaporkan; detailnya bisa ditanyakan ke Delegator.

### Struktur per Perusahaan

- Folder `agents/` dan `proyek/` punya level Perusahaan: `agents/<perusahaan>/<divisi>/`, `proyek/<perusahaan>/<divisi>/<proyek>/`. Mulai dari `maxipro`.
- Setiap Perusahaan punya Orchestrator Divisi sendiri (misalnya Software Factory sendiri), terpisah dari Perusahaan lain.
- Membuat Perusahaan baru: Pemilik memberi Perintah lewat Delegator → Software Factory Maxipro mengerjakannya sebagai Tugas biasa, menyalin template ke `agents/<perusahaan>/` dan `proyek/<perusahaan>/` → perubahan `agents/` langsung berlaku, perubahan `proyek/` lewat persetujuan merge → Laporan ke Pemilik.

### Delegator dan banyak Perusahaan

- Pemilik bisa brainstorming dengan Delegator; Delegator yang mengubahnya menjadi Perintah ke Orchestrator Utama.
- Satu Delegator untuk semua Perusahaan. Delegator meneruskan Perintah ke Orchestrator Utama milik Perusahaan yang dimaksud; bila Perusahaan tidak jelas, Delegator bertanya ke Pemilik.
- Cara membuat Orchestrator Utama untuk Perusahaan baru (contoh: VJ Labs, Telson Indonesia): ditunda.

### Branch, PR, dan pembatalan

- Setiap Tugas dikerjakan di branch sendiri di repo Proyek (`tugas/<id>-<ringkas>`); hasilnya Pull Request. Merge butuh persetujuan Pemilik.
- Staging = staging lokal: aplikasi Proyek dijalankan di Docker pada mesin yang sama dengan orchestrator, memakai database test. Deploy ke staging lokal otomatis; deploy ke production butuh persetujuan.
- Staging lokal tidak pernah memakai kredensial production pihak ketiga (WhatsApp, Telegram, Xendit dimatikan atau mode test/sandbox); Odoo hanya dibaca; database staging selalu database test, tidak pernah salinan data pelanggan asli.
- Pemilik bisa membatalkan Tugas kapan saja lewat Delegator: status `dibatalkan`, pekerjaan berjalan dihentikan, branch dibiarkan (tidak dihapus, tidak di-merge), Pemilik menerima Laporan.

### Brief Agen Spesialis v1 (Software Factory Maxipro)

| Agen | Dipanggil | Jobdesk | Tool | Batasan | Status |
|---|---|---|---|---|---|
| 01 Triage | Selalu, langkah pertama | Pahami Perintah, tentukan Proyek, tulis Kriteria Penerimaan, tandai sensitif, usulkan Sub-tugas; bila tidak jelas → Pertanyaan Klarifikasi | Baca repo (read-only) | Tidak menulis kode atau mengubah repo | disepakati |
| 03 Architecture | Fitur baru, lintas bagian, perubahan DB, integrasi baru | Rencana teknis: berkas yang diubah, pendekatan, risiko | Baca repo (read-only) | Tidak menulis kode aplikasi | disepakati |
| 04 Backend/API | Pekerjaan backend | Kode Node.js/Express + unit test di branch Tugas; v1 boleh menulis berkas migrasi | Baca/tulis repo di branch Tugas, jalankan install/test | Tidak push ke branch utama, tidak menyentuh DB production, tanpa rahasia di kode; migrasi ke production = titik persetujuan 2 | disepakati |
| 06 Web/UI | Pekerjaan frontend | Kode React + test komponen di branch Tugas; mobile tetap 07 | Sama dengan 04 | Sama dengan 04 | disepakati |
| 09 QA | Selalu sebelum merge | Jalankan test, cocokkan dengan Kriteria Penerimaan, tambah test yang kurang | Baca repo, jalankan test, tulis berkas test | Tidak memperbaiki kode aplikasi; temuan kembali ke 04/06 | disepakati |
| 11 Release/DevOps | Setelah 09 lolos | Deploy staging, siapkan Permintaan Persetujuan merge/production, deploy production setelah disetujui | Akses deploy sesuai `PROYEK.md` | Tidak deploy production tanpa persetujuan; selalu siapkan rollback; catat versi di Laporan | disepakati |

### Perutean Orchestrator Utama (v1)

- Perintah yang jelas soal software, aplikasi, atau Odoo → Software Factory.
- Perintah yang jelas soal bidang lain → Tugas `diterima`.
- Perintah yang meragukan → Pertanyaan Klarifikasi.

### Ketentuan umum v1

- Laporan memakai Bahasa Indonesia.
- Hanya Pemilik yang boleh memberi Perintah.
- Agen bekerja 24 jam. Selama sistem di lokal: agen bekerja selama komputer menyala; Tugas yang terputus dilanjutkan otomatis saat komputer menyala lagi; pengingat Permintaan Persetujuan tetap dihitung dari waktu permintaan dibuat. Aturan 24 jam berlaku lagi setelah pindah ke perangkat 24/7 atau VPS.

### Antrean

- Tugas untuk Orchestrator Divisi yang sama dikerjakan satu per satu (antre).
- Tugas untuk Orchestrator Divisi yang berbeda berjalan paralel.
- Tugas P0 menjeda Tugas yang sedang berjalan di divisi itu; P0 dikerjakan dulu, lalu Tugas yang dijeda dilanjutkan (pekerjaannya aman di branch-nya). P1 tidak menjeda, tetapi menjadi antrean pertama. P0/P1 langsung dilaporkan ke Pemilik.

### Persetujuan Pemilik

Tindakan berikut wajib menunggu Permintaan Persetujuan ke Pemilik lewat Delegator sebelum dijalankan:

1. Deploy ke production (Agen 11 Release/DevOps).
2. Perubahan skema database production (Agen 05 Database).
3. Menulis ke Odoo production (Agen 08 Odoo Integration).
4. Merge ke branch utama (kecuali perubahan yang hanya menyentuh `agents/`).
5. Melanjutkan Tugas yang melewati batas keras biaya.
6. Keputusan arsitektur besar: layanan baru, ganti teknologi database, atau mengubah kontrak API yang dipakai bersama (usulan 03 Architecture).

Pengecualian: rollback darurat oleh 11 (lihat Aturan tambahan sesi 3). Perubahan konfigurasi server production (Nginx, environment variable, service) dianggap deploy production (titik 1).

Tindakan lain boleh berjalan otomatis.

### Tugas dan status

Satu Perintah menjadi satu Tugas; Tugas dipecah menjadi Sub-tugas.

Status Tugas:

```
diterima → diurai → (menunggu klarifikasi) → dikerjakan → menunggu persetujuan → selesai / gagal / dibatalkan
```

- `menunggu klarifikasi` hanya dipakai bila ada Pertanyaan Klarifikasi; selama status ini antrean divisi dilepas.
- Kriteria Penerimaan ikut dikirim dalam Laporan saat Tugas selesai diurai, sebagai informasi (bukan persetujuan); Pemilik bisa mengoreksi kapan saja lewat Delegator.

### Kontrak Delegator ↔ Orchestrator Utama

Empat jenis pesan (tiga disepakati di R3-Q6, Pertanyaan Klarifikasi ditambahkan di R9-Q1):

| Pesan | Arah |
|---|---|
| Perintah | Delegator → Orchestrator Utama |
| Laporan | Orchestrator Utama → Delegator |
| Permintaan Persetujuan | Orchestrator Utama → Delegator (jawaban: setuju / tolak) |
| Pertanyaan Klarifikasi | Orchestrator Utama → Delegator (jawaban: teks bebas) |

Kanal WhatsApp/Telegram adalah urusan Delegator.

Isi pesan:

- **Perintah**: perusahaan, teks perintah, lampiran (opsional), id percakapan Delegator.
- **Laporan**: tugas_id, jenis (progres / hasil / gagal / agen baru), ringkasan, tautan ke hasil (PR, URL staging).
- **Permintaan Persetujuan**: id, tugas_id, tindakan, alasan, dampak, cara membatalkan. Jawaban: setuju / tolak, plus catatan opsional.
- **Pertanyaan Klarifikasi**: id, tugas_id, pertanyaan. Jawaban: teks bebas. (Isi rinci disempurnakan saat implementasi.)

Pengganti Delegator di v1: bot Telegram sederhana yang berjalan di VPS (long polling), yang hanya menjalankan kontrak empat jenis pesan (tanpa brainstorming). Ke depan kanal juga dihubungkan ke WhatsApp.

### Runtime dan penyimpanan

- Agen dijalankan dengan Claude Agent SDK di Node.js.
- v1: satu proses — Orchestrator Utama memanggil Orchestrator Divisi dan Agen Spesialis sebagai subagent. Lihat ADR 0002.
- Kode orchestrator disimpan di repo ini, folder `src/`, berdampingan dengan `agents/`, `proyek/`, dan `docs/`.
- Model AI ditulis di Definisi Agen: orchestrator memakai model Claude yang lebih kuat; Agen Spesialis memakai model yang lebih hemat, kecuali 03 Architecture. Model persis dipilih saat implementasi.
- Penyimpanan status Tugas: PostgreSQL (pilihan agen atas penyerahan Pemilik; lihat R3-Q4).
- Hosting v1: lokal, di komputer Pemilik saat ini. Harus bisa dipindah ke perangkat lokal lain (misalnya Mac yang menyala 24/7) atau ke VPS. Vercel dan Replit tidak dipakai.
- Pengemasan: satu berkas `docker-compose.yml` menjalankan orchestrator, PostgreSQL, dan bot Telegram bersama-sama.
- Cara pindah tempat: clone repo → salin `.env` → restore database (`pg_dump` / `pg_restore`) → `docker compose up`.
- Dashboard: tidak masuk v1; menyusul, dijalankan di tempat yang sama dengan orchestrator.

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
| R3-Q8 | Hosting (dibuka lagi di S2-R1-Q5) | "ya ditunda" | ditunda |
| R4-Q1 | Cara mengatasi bentrokan agen baru vs aturan merge diserahkan ke agen | "Saya kurang mengerti, saya ikut rekomendasimu" | disepakati |
| R4-Q2 | Orchestrator Utama dan Orchestrator Divisi memutuskan pemecahan sendiri; Laporan berisi hasil diskusi matang dan alasan | "Iya Orchestrator Utama dan Orchestrator Divisi berdiskusi sendiri, hasil diskusi yang sudah matang beserta alasan pemecahan report ke saya." | disepakati |
| R4-Q3 | Alur evaluasi hasil dengan Kriteria Penerimaan dan maksimal 2 kali ulang | "Ya pakai alur itu" | disepakati |
| R4-Q4 | Software Factory: selalu mulai dari 01 Triage, pemilihan agen dinamis, 09 QA wajib, 10 Security untuk perubahan sensitif | "ya" | disepakati |
| R4-Q5 | Perintah tanpa divisi disimpan sebagai Tugas `diterima`; divisi dibuat atas perintah Pemilik | "ya disimpan sementara sebagai tugas berstatus diterima, dan saya tinggal perintah ke Orchestrator divisi, dia yang bagian buatin" | disepakati |
| R4-Q6 | Antre per Orchestrator Divisi; paralel antar-divisi | "satu per satu antri. Kecuali beda divisi langsung paralel" | disepakati |
| R5-Q1 | Divisi baru dibuat lewat Perintah ke Delegator → Orchestrator Utama merancang di `agents/` → Laporan → Tugas `diterima` dilanjutkan. Pemilik brainstorming dengan Delegator, Delegator yang menginstruksikan Orchestrator Utama | "Ok saya salah jawaban di sebelumnya. Maksudnya bicara ke delegator untuk brainstorming, delegator yang menginstruksikan ke Orchestrator Utama. Iya saya setuju [...]" | disepakati |
| R5-Q2 | Satu Delegator untuk semua Perusahaan, merutekan per Perusahaan; cara membuat Perusahaan baru ditunda | "Iya Delegator" | disepakati |
| R5-Q2 | Cara membuat Orchestrator Utama untuk Perusahaan baru | "Iya Delegator" | ditunda |
| R6-Q1 | Daftar Proyek per divisi, satu folder per Proyek (`proyek/maxipro/software-factory/<proyek>/`) | "Nanti saya buat link nya, saat ini belum ada. Tapi bayangan saya proyek/softwarefactory nanti maxichat dan maxione masuk ke dalam folder tersebut. Jadi nanti setiap proyek apps yang baru masuk ke dalam folder masing2. Harus rapi." | disepakati |
| R6-Q1 | Link repo tiap Proyek (ditutup di S2-R1-Q1/Q2) | "Nanti saya buat link nya, saat ini belum ada." | ditunda |
| R6-Q2 | 4 tabel PostgreSQL dan status Sub-tugas | "Ya" | disepakati |
| R6-Q3 | Persetujuan tidak dijawab: tidak ada auto-setuju, pengingat 24/72 jam, antrean dilepas | "Ya" | disepakati |
| R6-Q4 | Format Definisi Agen | "Ya" | disepakati |
| R6-Q5 | Konteks bersih per Tugas + Catatan Divisi | "Keduanya [...]" | disepakati |
| R6-Q6 | Laporan hanya saat status Tugas berubah, agen baru, atau gagal | "Ya" | disepakati |
| R7-Q1 | Folder `proyek/maxipro/software-factory/` dan `PROYEK.md` dengan isi yang diusulkan | "iya saya setuju menggunakan pakai nama `software-factory` dan berkas `PROYEK.md` dengan isi di atas." | disepakati |
| R7-Q2 | Catatan Divisi di `agents/<perusahaan>/<divisi>/_catatan.md` | "Ya" | disepakati |
| R7-Q3 | Rahasia sebagai environment variable di server; agen tidak menulis rahasia | "Ya" | disepakati |
| R7-Q4 | Batas biaya lunak/keras per Tugas; batas keras jadi titik persetujuan ke-5 | "Ya" | disepakati |
| R7-Q5 | v1 Software Factory: 6 agen inti (digantikan S3-R1-Q1) | "Ya" | disepakati |
| R8-Q1 | Isi tiga jenis pesan (tanya ulang R7-Q6) | "Setuju" | disepakati |
| R8-Q2 | Kode orchestrator di repo ini, folder `src/` | "Ya" | disepakati |
| R8-Q3 | Pengganti Delegator v1: bot Telegram sederhana; ke depan juga WhatsApp | "Ya, ke depan mau dihubungkan dengan whatsapp juga" | disepakati |
| R8-Q4 | Branch per Tugas, PR, staging otomatis (staging = lokal, S3-R6-Q1), production butuh persetujuan | "Ya" | disepakati |
| R8-Q5 | Pembatalan Tugas: branch dibiarkan, Laporan dikirim | "Ya" | disepakati |
| R8-Q6 | Model lebih kuat untuk orchestrator, lebih hemat untuk Agen Spesialis kecuali 03 | "Ya" | disepakati |
| R8-Q7 | Tulis ADR 0002 v1 satu proses dengan subagent | "Ya" | disepakati |
| R9-Q1 | Tambah Pertanyaan Klarifikasi (pesan ke-4) dan status `menunggu klarifikasi`; antrean dilepas | "Ya" | disepakati |
| R9-Q2 | Kriteria Penerimaan dikirim dalam Laporan setelah diurai, sebagai informasi | "Ya" | disepakati |
| R9-Q3 | Perutean Orchestrator Utama v1 | "Ya" | disepakati |
| R9-Q4 | Laporan berbahasa Indonesia, hanya Pemilik memberi Perintah, agen bekerja 24 jam | "Ya" | disepakati |
| R10-Q1 | Rangkuman akhir benar | "Ya" | disepakati |
| R10-Q2 | Link repo Proyek | "Ya" | ditunda |
| R10-Q2 | Tempat menyimpan Pertanyaan Klarifikasi di PostgreSQL (ditutup di S2-R1-Q6) | "Ya" | ditunda |
| R10-Q2 | Apakah 09 QA juga berlaku untuk perubahan `agents/` (ditutup di S2-R1-Q7) | "Ya" | ditunda |
| S2-R1-Q1 | Repo Maxichat: `stepsans/MaxiChat.App` | "Ya pakai MaxiChat.App" | disepakati |
| S2-R1-Q2 | Repo Maxione: `stepsans/maxione` | "Ya pakai itu" | disepakati |
| S2-R1-Q3 | Level Perusahaan di `agents/` dan `proyek/` | "Iya betul" | disepakati |
| S2-R1-Q4 | Setiap Perusahaan punya Orchestrator Divisi sendiri | "A Setiap perusahaan mempunyai softaware factory sendiri jadi terpisah" | disepakati |
| S2-R1-Q6 | Tabel `pertanyaan_klarifikasi` | "Ya" | disepakati |
| S2-R1-Q7 | Perubahan `agents/`: tanpa 09 QA, cek format + dibaca Orchestrator Utama | "Ya" | disepakati |
| S2-R1-Q9 | Sesi ini juga merinci brief 6 agen v1 | "Ya boleh supaya contextnya tepat. Apakah benar seperti itu?" | disepakati |
| S2-R2-Q1 | Hosting hybrid: Vercel untuk bagian ringan, orchestrator di VPS (digantikan S2-R4-Q2) | "Ya saya pilih B. Coba cek apa plus minus bila dashboard di vercel, tetapi orchestrator di VPS?" | disepakati |
| S2-R3-Q2 | Bot Telegram di VPS | "Bot di VPS" | disepakati |
| S2-R3-Q3 | Definisi Agen di folder `agents/` sendiri | "A" | disepakati |
| S2-R3-Q4 | Cara membuat Perusahaan baru | "Ya" | disepakati |
| S2-R3-Q6 | Brief 11 Release/DevOps | "Ya" | disepakati |
| S2-R3 (catatan) | Bila hybrid menambah rumit, semua di VPS (digantikan S2-R4-Q2) | "Bila lebih stabil dashboard di VPS juga tidak masalah. Saya hanya iseng mencoba di vercel, tetapi bila itu membuat tambah rumit, saya putuskan semuanya ada di VPS." | disepakati |
| S2-R4-Q1 | v1 tanpa dashboard; dashboard menyusul | "B" | disepakati |
| S2-R4-Q2 | Hosting v1 lokal di komputer ini; bisa dipindah ke perangkat lokal lain atau VPS | "Bagaimana untuk sementara di lokal saja. Komputer ini. [...] Saat ini saya putuskan dibangun di lokal. Dan akan ada opsi memindahkan ke Device lokal lain atau VPS" | disepakati |
| S2-R4-Q3 | Satu berkas `docker-compose.yml` | "Ya satu berkas docker-compose.yml. [...]" | disepakati |
| S2-R4-Q4 | Brief 01, 03, 04, 06, 09 | "ya untuk semuanya" | disepakati |
| S2-R5-Q1 | Selama lokal: agen bekerja selama komputer menyala, Tugas dilanjutkan otomatis | "Ya taruh di lokal sini" | disepakati |
| S2-R5-Q2 | Rangkuman sesi 2 benar; 7 agen menyusul dan Digital Growth di sesi berikutnya | "Ya" | disepakati |
| S3-R1-Q1 | Semua 13 agen Software Factory dibangun bersama di v1 | "Saya mau Orchestrator Divisi: Software Factory dibangun bersama di V1. Kenapa harus menyusul?" | disepakati |
| S3-R2-Q1 | Agen 08 hanya membaca Odoo production; penulisan lewat persetujuan | "Apa itu odoo staging jelaskan. saya setuju agen 08 hanya boleh membaca dari Odoo production, dan semua penulisan tetap lewat persetujuan." | disepakati |
| S3-R2-Q1 | Apakah Odoo staging ada (dijawab di S3-R3-Q1) | "Apa itu odoo staging jelaskan." | disepakati |
| S3-R2-Q2 | Agen wajib membaca CLAUDE.md/AGENTS.md Proyek | "Ya" | disepakati |
| S3-R2-Q3 | Database test sekali-pakai di docker-compose | "Ya" | disepakati |
| S3-R2-Q4 | Rollback darurat tanpa persetujuan | "Ya" | disepakati |
| S3-R2-Q5 | Brief 02 Product/Business Analyst (draf terlampir) | "Ya, Terlampir (Benerin bila aku salah)" | disepakati |
| S3-R2-Q6 | Brief 05 Database (draf terlampir) | "Ya, Terlampir (benerin bila aku salah)" | disepakati |
| S3-R2-Q7 | Brief 07 Mobile (draf terlampir) | "Ya" | disepakati |
| S3-R2-Q8 | Brief 08 Odoo Integration (draf terlampir) | "Ya" | disepakati |
| S3-R2-Q9 | Brief 10 Security (draf terlampir) | "Ya" | disepakati |
| S3-R2-Q10 | Brief 12 Observability/SRE (draf terlampir) | "Ya" | disepakati |
| S3-R2-Q11 | Brief 13 Documentation (draf terlampir) | "Ya" | disepakati |
| S3-R3-Q1 | Odoo staging belum ada; 08 hanya membaca; Pemilik menyiapkan staging bila diperlukan | "Belum ada. Hanya membaca. Bila diperlukan saya siapin odoo staging nya" | disepakati |
| S3-R3-Q2 | Hanya Pemilik memberi Perintah; "authorized employees" dihapus dari brief 02 | "Ya ikut rekomendasimu" | disepakati |
| S3-R3-Q3 | Agen tidak menghubungi Pemilik langsung | "Ya" | disepakati |
| S3-R3-Q4 | Agen 12 tidak membuat Tugas sendiri | "Ya" | disepakati |
| S3-R3-Q5 | Agen 12 v1 pantau ±30 menit setelah deploy | "Ya" | disepakati |
| S3-R3-Q6 | Istilah Maxione = Super Apps Maxipro; tidak dipakai untuk Perusahaan lain | "Ya tambahkan. Nanti super Apps ini bernama Maxione. Di dalamnya ada beberapa software maxipro yang saling terhubung. Hanya khusus untuk perusahaan Maxipro. Bila nanti duplikasi ke perusahaan lain, jangan menggunakan Maxipro Super Apps" | disepakati |
| S3-R3-Q7 | Kartu agen bahasa Inggris, output ke Pemilik bahasa Indonesia | "Ya" | disepakati |
| S3-R3-Q8 | KPI sebagai arah kerja, pengukuran menunggu dashboard | "Ya" | disepakati |
| S3-R3-Q9 | Uji perangkat oleh Pemilik/tim | "Ya" | disepakati |
| S3-R3-Q10 | Simpan draf brief sebagai draf Definisi Agen setelah sesi | "Ya" | disepakati |
| S3-R4-Q1 | Maxione aplikasi utama internal Maxipro; Maxichat berdiri sendiri, multi-tenant, terhubung | "Maxione itu apps utama, Maxichat itu apps berdiri sendiri tetapi saling berhubungan. Setiap user di maxione pasti punya maxichat. Setiap tenant di maxichat belum tentu punya maxione. Maxichat banyak tenant. Maxione dipakai hanya perusahaan maxipro." | disepakati |
| S3-R4-Q2 | Software internal baru jadi menu di repo Maxione | "Jadi menu baru di repo Maxione" | disepakati |
| S3-R4-Q3 | Kartu agen netral ("the Company") | "Ya, bisa menduplikasi tanpa kata kata maxipro. Lebih baik menyebut the company" | disepakati |
| S3-R4-Q4 | Draf kartu agen 01, 03, 04, 06, 09, 11 dari Pemilik, didiskusikan dulu (dibahas di S3-R6) | "Saya ada, tetapi takut lebih kacau. Tidak sesuai dengan pemikiranmu. Saya kirim tapi diskusi dulu ya." | disepakati |
| S3-R5-Q1 | Sesi 3 dilanjutkan 5 round lagi (round 6–10) | "A lanjut" | disepakati |
| S3-R6-Q1 | Staging lokal di Docker untuk v1 | "ikut rekomendasimu" | disepakati |
| S3-R6-Q2 | Triage hanya menerima Tugas dari Perintah Pemilik | "Ya disesuaikan, Hanya menerima dari delegator" | disepakati |
| S3-R6-Q3 | Lima tugas inti Triage ditambahkan ke draf 01 | "Iya setuju dengan rekomendasimu" | disepakati |
| S3-R6-Q4 | Triage kumpulkan bukti; 09 QA mereproduksi | "Ya, saya percaya usulmu dibanding draft yang aku kirim karena belum sempurna. Tolong sempurnakan" | disepakati |
| S3-R6-Q5 | Kolom prioritas P0–P3 di `tugas`; P0/P1 di depan antrean + Laporan | "Ya" | disepakati |
| S3-R6-Q6 | Titik persetujuan ke-6: keputusan arsitektur besar (tiga contoh) | "(a), khusus untuk tiga contoh di atas. Kejadiannya jarang, tapi dampaknya besar." | disepakati |
| S3-R6-Q7 | 03: rencana teknis per Tugas; dokumen menyeluruh hanya saat perubahan besar | "Ya" | disepakati |
| S3-R6-Q8 | 04/06: hanya branch Tugas, tanpa kredensial production | "Ya" | disepakati |
| S3-R6-Q9 | 09: BLOCKED → Pertanyaan Klarifikasi; boleh menulis test | "Ya" | disepakati |
| S3-R6-Q10 | Code review v1 = QA + Security + persetujuan merge Pemilik | "Ya (a) untuk v1" | disepakati |
| S3-R6-Q11 | Konfigurasi server production = deploy production; backup sebelum migrasi | "Ya" | disepakati |
| S3-R6-Q12 | Istilah di kartu: the Company, Task, Pemilik, lewat Orchestrator | "Ya" | disepakati |
| S3-R7-Q1 | Staging lokal tanpa kredensial production pihak ketiga, Odoo hanya dibaca, DB test | "Ya" | disepakati |
| S3-R7-Q2 | P0 menjeda Tugas berjalan; P1 antrean pertama | "B" | disepakati |
| R5-Q3 | Sesi dilanjutkan 5 round lagi (round 6–10) | "Lanjut dong mau 10 round juga boleh" | disepakati |

## Alternatif yang ditolak

- R2-Q1: pola generik tanpa divisi (b) dan semua lapisan plus 21 agen sekaligus (c). Pemilik tidak memberi alasan.
- R2-Q5: n8n (b), LangGraph/CrewAI (c), menunda (d). Pemilik tidak memberi alasan. Hosting di Replit juga ditolak, tanpa alasan.
- R3-Q1: usulan agen agar pemecahan agen menjadi titik persetujuan ke-5. Alasan Pemilik: "saya selalu setuju ke orchestrator".
- R5-Q1: Pemilik bicara langsung ke Orchestrator Divisi. Pemilik meralat jawaban R4-Q5: "saya salah jawaban di sebelumnya".
- S2-R2-Q1: hosting full Vercel dengan Vercel Workflows (c) dan VPS penuh sejak awal (a). Pemilik memilih hybrid, lalu menggantinya dengan lokal di S2-R4-Q2.
- R4-Q5: usulan agen agar Perintah di luar divisi ditolak dengan Laporan. Pemilik memilih menyimpannya sebagai Tugas `diterima`; tidak ada alasan tambahan.

## Catatan inferensi

- Pindah dari Mac (Apple Silicon/ARM) ke VPS (umumnya x86) kemungkinan lancar karena image Docker umumnya multi-arsitektur, tetapi perlu dicek saat pindah (inferensi agen; `belum terverifikasi`).

- Lingkup R1-Q2 (siap implementasi) kemungkinan butuh lebih dari 5 round (inferensi agen).
- R3-Q4: pilihan agen adalah PostgreSQL — sesuai stack Pemilik dan aman untuk penulisan bersamaan oleh beberapa agen (inferensi agen). Pemilik menyebut pilihan ini bisa ditanyakan ke Bos Arga; belum ditinjau.
- R4-Q1: pilihan agen adalah pengecualian merge otomatis untuk perubahan yang hanya menyentuh `agents/` (inferensi agen).
