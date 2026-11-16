# Orchestrator

Konteks sistem agen AI yang menerima perintah Pemilik, membaginya ke lapisan di bawahnya, dan mengoordinasi pengerjaannya untuk satu Perusahaan.

## Bahasa

### Pihak

**Pemilik**:
Orang yang memberi perintah kepada sistem dan memegang hak persetujuan atas tindakan berisiko.
_Avoid_: Me, User, Owner

**Perusahaan**:
Satu entitas bisnis milik Pemilik yang dilayani oleh tepat satu Orchestrator Utama.
_Avoid_: Bisnis, Company, Tenant

### Lapisan agen

**Delegator**:
Agen juru bicara Pemilik; satu-satunya titik kontak Pemilik dengan sistem.
_Avoid_: Asisten, Bot, Agen utama

**Orchestrator Utama**:
Agen koordinasi tingkat Perusahaan yang membagi pekerjaan ke Orchestrator Divisi.
_Avoid_: Orchestrator (tanpa kualifikasi), Orchestrator Maxipro

**Orchestrator Divisi**:
Agen koordinasi untuk satu bidang kerja di dalam Perusahaan, yang membagi pekerjaan ke Agen Spesialis.
_Avoid_: Sub-orchestrator, Orchestrator Agent, Divisi (tanpa kualifikasi)

**Agen Spesialis**:
Agen yang mengerjakan satu jenis pekerjaan di bawah satu Orchestrator Divisi.
_Avoid_: Worker, Sub-agent, Persona

**Definisi Agen**:
Berkas yang memuat jobdesk dan instruksi satu agen.
_Avoid_: Prompt, Persona, Config agen

**Catatan Divisi**:
Berkas berisi pelajaran jangka panjang satu Orchestrator Divisi yang terus diperbarui antar-Tugas.
_Avoid_: Memory, Log, Jurnal

### Satuan kerja

**Maxione**:
Super Apps milik Perusahaan Maxipro: ekosistem berisi aplikasi utama internal Maxipro beserta aplikasi yang terhubung dengannya, seperti Maxichat; khusus untuk Maxipro.
_Avoid_: Maxipro Super Apps (di luar Maxipro), Super Apps (tanpa kualifikasi)

**Proyek**:
Satu aplikasi atau sistem yang menjadi sasaran kerja sebuah Orchestrator Divisi.
_Avoid_: Repo, App, Aplikasi (tanpa kualifikasi)

**Tugas**:
Satuan kerja yang dibentuk dari satu Perintah dan dilacak statusnya sampai tuntas.
_Avoid_: Tiket, Job, Task

**Sub-tugas**:
Bagian dari satu Tugas yang diserahkan ke satu Orchestrator Divisi atau satu Agen Spesialis.
_Avoid_: Subtask, Langkah

**Kriteria Penerimaan**:
Ukuran tertulis yang menentukan apakah hasil sebuah Sub-tugas sudah sesuai keinginan Pemilik.
_Avoid_: Acceptance criteria, DoD, Target

### Pesan antara Delegator dan Orchestrator Utama

**Perintah**:
Instruksi Pemilik yang diteruskan Delegator ke Orchestrator Utama.
_Avoid_: Request, Prompt, Order

**Laporan**:
Pesan dari Orchestrator Utama ke Delegator berisi progres atau hasil sebuah Tugas, atau kabar perubahan susunan agen.
_Avoid_: Notifikasi, Update

**Pertanyaan Klarifikasi**:
Pesan dari Orchestrator Utama ke Delegator yang meminta jawaban bebas dari Pemilik agar sebuah Tugas bisa dilanjutkan.
_Avoid_: Konfirmasi, Pertanyaan (tanpa kualifikasi)

**Permintaan Persetujuan**:
Pesan dari Orchestrator Utama ke Delegator yang meminta Pemilik memilih setuju atau tolak atas satu tindakan berisiko.
_Avoid_: Konfirmasi, Approval
