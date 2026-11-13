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
