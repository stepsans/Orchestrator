# 0001 — Orchestrator hanya mengoordinasi, tidak mengerjakan tugas teknis

Status: accepted

Orchestrator Utama dan Orchestrator Divisi hanya memecah pekerjaan, memilih lapisan di bawahnya, melacak status, dan menggabungkan hasil; pekerjaan teknis selalu dikerjakan Agen Spesialis. Pemilik memutuskan ini agar setiap lapisan punya satu peran yang jelas. Alternatif yang ditolak — orchestrator ikut mengerjakan tugas kecil sendiri — lebih cepat untuk pekerjaan sepele, tetapi membuat batas tanggung jawab antara orchestrator dan Agen Spesialis kabur dan membuat hasil sulit dilacak ke agen yang mengerjakannya (alasan diusulkan agen dan disetujui Pemilik; `belum terverifikasi` dalam praktik).
