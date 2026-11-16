# Spesifikasi: Orchestrator

Status sesi: selesai (sesi 1: 2026-11-13; sesi 2: 2026-11-16). Nomor pertanyaan sesi 2 memakai awalan S2-. Sesi 3 (2026-11-16, awalan S3-) berjalan: semua 13 agen Software Factory masuk v1.

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
| `tugas` | id, perusahaan, divisi, proyek, perintah asli, Kriteria Penerimaan, status, waktu dibuat/diubah |
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
- Brief agen 02, 05, 07, 08, 10, 12, 13: disetujui Pemilik beserta draf Definisi Agen lengkap yang ia lampirkan (berbahasa Inggris); koreksi terhadap aturan yang sudah disepakati masih `terbuka` (S3-R3).

### Aturan tambahan sesi 3

- Sebelum bekerja di sebuah Proyek, setiap agen wajib membaca dan mematuhi `CLAUDE.md` dan `AGENTS.md` Proyek itu. Untuk urusan teknis kode, aturan Proyek menang; aturan persetujuan sistem ini tetap berlaku.
- `docker-compose.yml` menyertakan satu database test sekali-pakai yang dikosongkan setiap kali test dijalankan (untuk `*.dbtest.ts` dan uji migrasi).
- Rollback darurat: bila 12 Observability mendeteksi error parah setelah deploy, 11 Release/DevOps boleh langsung rollback ke versi sebelumnya tanpa menunggu persetujuan, lalu segera mengirim Laporan. Pengecualian dari titik persetujuan ke-1.
- Agen 08 hanya membaca dari Odoo production; semua penulisan ke Odoo lewat Permintaan Persetujuan (titik ke-3).

### Fakta Proyek Maxichat (dicek 2026-11-16 dari repo `stepsans/MaxiChat.App`, branch `main`)

- Monorepo pnpm. Bagian di `artifacts/`: `admin`, `api-server` (Express + drizzle-orm + PostgreSQL), `desktop`, `maxichat-video`, `mobile`, `mockup-sandbox`, `whatsapp-ai`.
- Mobile: Expo `~54.0.27`, React Native `0.81.5` (`artifacts/mobile/package.json`).
- Repo punya `AGENTS.md` dan `CLAUDE.md` sendiri; `AGENTS.md` merujuk pekerjaan berjalan integrasi Odoo di `docs/odoo-integration-brief.md`.
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
- Deploy ke staging boleh otomatis; deploy ke production butuh persetujuan.
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

### Persetujuan Pemilik

Tindakan berikut wajib menunggu Permintaan Persetujuan ke Pemilik lewat Delegator sebelum dijalankan:

1. Deploy ke production (Agen 11 Release/DevOps).
2. Perubahan skema database production (Agen 05 Database).
3. Menulis ke Odoo production (Agen 08 Odoo Integration).
4. Merge ke branch utama (kecuali perubahan yang hanya menyentuh `agents/`).
5. Melanjutkan Tugas yang melewati batas keras biaya.

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
| R8-Q4 | Branch per Tugas, PR, staging otomatis, production butuh persetujuan | "Ya" | disepakati |
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
| S3-R2-Q1 | Apakah Odoo staging ada | "Apa itu odoo staging jelaskan." | menunggu fakta |
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
