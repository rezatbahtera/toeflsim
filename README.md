# TOEFL ITP Simulator Pro

Aplikasi single-file HTML untuk simulasi ujian TOEFL ITP dengan editor data, scoring, dan ekspor laporan.

**File deliverable:** `ToeflSim.html` (~365 KB, satu file mandiri tanpa dependency eksternal kecuali Tailwind CDN).

## Quick Start

1. Buka `ToeflSim.html` di browser modern (Chrome, Firefox, Safari, Edge).
2. Pilih sumber data di halaman init:
   - **Bank Soal** (default) — auto-load dari Github `rezatbahtera/toeflsim`
   - **URL** — paste URL JSON eksternal
   - **Folder URL** — load multi-file dari folder (Github API atau index.json)
   - **Upload File** — upload `.json` lokal
   - **Default** — pakai data sample bawaan
3. Isi **Nama Peserta** di start screen (tersimpan untuk sesi berikutnya).
4. Pilih mode: **Real Test** (strict, urut, locked) atau **Latihan** (free nav, cek pembahasan).
5. Klik **Mulai Ujian Sekarang**.

## Daftar Dokumen

| File | Isi |
|---|---|
| `README.md` | Quick start + overview (file ini) |
| `ARCHITECTURE.md` | Struktur aplikasi, state management, hierarki section/part/group/question |
| `JSON_TEMPLATE.md` | Format JSON untuk data tes, contoh lengkap, parser bulk dump |
| `FLOW_LOGIC.md` | Alur user flow, direction screen, timer, audio, scoring |
| `FUNCTIONS.md` | Reference 141 fungsi JavaScript dikelompokkan per modul |
| `EDITOR_GUIDE.md` | Panduan pakai editor visual (sections, parts, groups, questions, audio picker) |
| `EXPORT_FORMATS.md` | Spek 3 format download: HTML report, CSV, JSON |
| `CHANGELOG.md` | Riwayat perubahan dan iterasi pengembangan |

## Fitur Utama

- **Schema fleksibel**: Section → Part → Group (opsional, untuk Part B/C listening) → Question
- **Direction screen**: full-screen overlay untuk instruksi Section dan Part dengan audio direction
- **Audio handling**: auto-play 1x, group-shared audio, transcript reveal saat review
- **Timer**: Section timer (durasi total), Part timer (opsional), Question timer (per-soal)
- **Visual editor**: drag-free struktural editor dengan pagination group-aware, bulk dump importer dari clipboard
- **Audio Bank picker**: browse file mp3 dari Google Cloud Storage / Apache directory listing dengan filter
- **Bulk dump parser**: parse text mentah Longman/standard TOEFL ITP ke struktur question
- **Scoring TOEFL ITP**: rumus konversi raw ke scaled score (range 310-677)
- **Practice mode**: replay audio, pause audio/timer, cek pembahasan toggle
- **Export 3 format**: laporan HTML self-contained (rekomendasi, bisa print/PDF), CSV, JSON raw
- **Access control**: Editor hanya akses untuk user `cakresa` saat data dari Bank Soal repo

## Browser Support

- Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- Mobile: iOS Safari, Chrome Android (responsive layout dengan sidebar drawer)
- Tidak mendukung IE11 atau browser legacy

## Deploy

File `.html` self-contained — host di static hosting apa saja (Github Pages, Vercel, Netlify, Nginx, Apache). Tidak perlu backend, Node.js, atau build step.

**Production deployment:** https://toeflsim.cakresa.id/

## License

Internal proyek. Hubungi pemilik repo `rezatbahtera/toeflsim` untuk lisensi penggunaan.
