# 0002 — v1 berjalan dalam satu proses dengan subagent

Status: accepted

Pada v1, Orchestrator Utama memanggil Orchestrator Divisi dan Agen Spesialis sebagai subagent Claude Agent SDK di dalam satu proses Node.js, bukan sebagai layanan terpisah yang berkomunikasi lewat antrean pesan. Pemilik memilih ini untuk v1. Alternatif yang ditolak — tiap lapisan sebagai layanan terpisah dengan queue — memungkinkan agen berjalan lama dan diskalakan sendiri-sendiri, tetapi menambah infrastruktur dan menyulitkan debugging di tahap awal (alasan diusulkan agen dan disetujui Pemilik; `belum terverifikasi`). Konsekuensi: pindah ke model antrean nanti berarti merombak cara antar-lapis berkomunikasi (`perkiraan`).
