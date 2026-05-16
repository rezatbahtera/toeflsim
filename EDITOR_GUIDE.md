# Editor Guide

Panduan menggunakan visual editor di `ToeflSim.html` untuk membuat & edit data tes.

## Akses Editor

Klik tab **Editor** di top bar (sebelah tab Ujian).

**Catatan:** Kalau data di-load dari Bank Soal Github (`rezatbahtera/toeflsim`), tab Editor hanya muncul untuk user dengan nama "cakresa" di field Nama Peserta. Untuk source lain (lokal/URL/default), tab Editor selalu visible.

## Layout Editor

```
┌──────────────────────────────────────────────┐
│  Toolbar atas:                               │
│  [Tambah Section] [Export JSON] [Save GH]    │
│  [Dump Longman] [Bank Audio] [Reset]         │
├──────────────────────────────────────────────┤
│  Title input: testData.title                 │
├──────────────────────────────────────────────┤
│  ┌─ Section Card (sticky header) ─────────┐  │
│  │ [Title input] [Type] [Duration] [...]  │  │
│  │ ┌─ 📢 Direction Section (collapsible) ┐│  │
│  │ │ - Instruction text                  ││  │
│  │ │ - Direction audio URL + 📁 Pilih   ││  │
│  │ └─────────────────────────────────────┘│  │
│  │ ┌─ Part Card ───────────────────────┐ │  │
│  │ │ [Title] [Instruction] [...]       │ │  │
│  │ │ [📢 Direction Part audio]         │ │  │
│  │ │ [Audio Part legacy (kalau no grp)]│ │  │
│  │ │ Pagination: 1 2 3 4 ...           │ │  │
│  │ │ ┌─ Group Card (purple gradient) ┐ │ │  │
│  │ │ │ [Group passage] [Group audio] │ │ │  │
│  │ │ │ ┌─ Question N (collapsed) ─┐  │ │ │  │
│  │ │ │ │ ▶ Q5 (P-3) Preview... A  │  │ │ │  │
│  │ │ │ ├─ Question N+1 (expanded)─┤  │ │ │  │
│  │ │ │ │ ▼ Q6 (P-4) Preview... B  │  │ │ │  │
│  │ │ │ │ Audio URL + Pilih        │  │ │ │  │
│  │ │ │ │ Passage textarea          │  │ │ │  │
│  │ │ │ │ Question textarea         │  │ │ │  │
│  │ │ │ │ Options (4 radio)         │  │ │ │  │
│  │ │ │ └──────────────────────────┘  │ │ │  │
│  │ │ └──────────────────────────────────┘ │ │  │
│  │ └────────────────────────────────────┘ │  │
│  └────────────────────────────────────────┘  │
│  [+ Tambah Section]                          │
└──────────────────────────────────────────────┘
```

## Section Card

### Field Section

- **Judul Section**: e.g. "Section 1 Listening Comprehension"
- **Tipe**: dropdown `listening` / `structure` / `reading` — menentukan formula scoring
- **Durasi (menit)**: total Section timer (0 = no timer)

### Tombol Section

- **↑ ↓** — Pindah urutan Section
- **🗑 Hapus** — Konfirmasi dialog, lalu hapus

### 📢 Direction Section (collapsible)

Klik untuk expand:

- **Teks Direction** — textarea, tampil di Section direction screen
- **URL Audio Direction** — opsional, audio narrator yang play saat direction screen muncul
- **📁 Pilih** — buka Audio Bank picker untuk browse file

Direction screen muncul **hanya saat user pertama kali masuk Section**. Bisa dipicu juga kalau hanya audio (tanpa text) terisi.

## Part Card

Beberapa Part bisa berada dalam satu Section. Pagination editor 10 soal per halaman.

### Field Part

- **Judul Part**: e.g. "Part A", "Reading Set 1"
- **Instruction Part**: textarea untuk direction Part-level
- **Durasi (detik)**: Part timer opsional

### Tombol Part

- **↔ Pindah Section...** — dropdown pilih Section target, konfirmasi → pindah Part
- **↑ ↓** — Pindah urutan Part dalam Section
- **🗑 Hapus** — Konfirmasi

### URL Audio

- **URL Audio Direction Part** + 📁 Pilih
- **URL Audio Part (legacy)** — hanya muncul kalau Part tidak punya groups. Saat user buat group, field ini di-migrate

## Group Card

Group adalah cara share passage + audio antar multiple questions. Cocok untuk:

- **Listening Part B/C**: 1 conversation → 3-4 pertanyaan
- **Reading**: 1 passage → 8-12 pertanyaan

### Field Group

- **Passage Group**: teks atau transkrip shared
- **URL Audio Group**: file audio shared (play 1x sebelum questions)
- **📁 Pilih** untuk audio picker

### Tombol Group

- **+ Tambah Soal** — Append question dengan groupId ini
- **Bubarkan Group** — Konfirmasi, lalu set groupId=null di semua questions
- **🗑 Hapus Group** — Hapus group + semua questions di dalamnya

### Buat Group dari soal yang sudah ada

Di Part header: tombol **+ Group Baru**:

1. Prompt: "Masukkan nomor soal (e.g. 1-3 atau 1,2,3)"
2. Parser ekstract range/list
3. Buat group baru dengan ID unik
4. Set `groupId` di semua question yang dipilih
5. **Penting**: question yang dipilih harus *consecutive* (berurutan)

## Question Card

### Header (clickable)

Format: `▶/▼ Q5 (P-3) [preview pertanyaan] [A]`

- **▶/▼** chevron menunjukkan expanded/collapsed
- **Q5** = nomor global dalam Section (continuous across Parts)
- **(P-3)** = posisi dalam Part (1-indexed)
- **Preview** = 80 char pertama dari pertanyaan
- **[A]** = letter jawaban benar (badge hijau)

Klik header → expand body, **otomatis collapse semua soal lain di Part yang sama**.

### Header buttons (event.stopPropagation)

- **Timer X s** — input nilai timeLimitSeconds per soal
- **↑ ↓** — Pindah urutan dalam Part
- **🗑 Hapus** — konfirmasi

### Body (saat expanded)

- **URL Audio (opsional)** + 📁 Pilih
- **Passage / Transkrip** — kalau soal stand-alone tanpa group. Support marker `~[kata]~`
- **Pertanyaan** — teks pertanyaan, support marker
- **Pilihan Ganda** — 4 options:
  - Text input untuk teks pilihan
  - Checkbox "✓ Benar" untuk set `correctIndex`
  - Reason EN + ID textareas (collapsible)

## Pagination

Saat Part punya >10 soal, pagination muncul:

```
[<< Awal] [< Prev] [1] [2] [3] [4] [Next >] [Akhir >>]
```

Algoritma group-aware: **group tidak split ke 2 halaman**. Contoh 3 group × 4 soal:

- Page 1: g1 + g2 (8 soal)
- Page 2: g3 (4 soal)

State `editorQPage[partKey]` track current page per Part.

## Toolbar Atas

### + Tambah Section

Append new Section di akhir.

### 💾 Export JSON

Download `testData` sebagai file JSON (untuk backup atau share). Filename: `{title}_{date}.json`.

### 📤 Save Github

Klik untuk modal save ke Github repo via PUT API. User harus punya personal access token (PAT) dengan scope `repo`. Token di-prompt + disimpan di localStorage.

### + Dump Longman

Open Bulk Dump modal — paste raw text dari buku Longman atau format TOEFL standard, parser akan extract structure.

Format yang didukung: Listening short, Listening long (Part B/C), Structure, Written Expression, Reading. Auto-detect via dropdown.

Lihat `JSON_TEMPLATE.md` section "Bulk Dump Parser" untuk format details.

### 🎵 Bank Audio

Konfigurasi URL base catalog audio. Default: `https://storage.googleapis.com/toeflsim-audio/`. URL disimpan di localStorage `audioBankUrl`.

Setelah set, semua tombol **📁 Pilih** di field audio URL akan buka picker yang browse dari catalog ini.

## Audio Bank Picker

Modal yang membuka folder browser untuk audio.

### Header

- **URL base** — bisa edit langsung, klik Muat untuk reload
- **Filter** — search box real-time, filter folder/file by name (case-insensitive)
- **Count** — "X / Y items" menunjukkan filtered vs total
- **Breadcrumb** — navigation path: `/ (root) > section-1 > part-a > exercise-01.mp3`

### Content

- **Folder** (icon kuning) — klik untuk navigate ke dalam
- **File** (icon biru) — punya inline audio preview + tombol **Pakai**

Format file yang ditampilkan: `.mp3`, `.m4a`, `.wav`, `.ogg`, `.aac`, `.flac`.

### Source format support

- **GCS bucket** (`storage.googleapis.com/BUCKET/...`) — auto-convert ke JSON list API dengan `?delimiter=/`
- **Apache/nginx HTML directory listing** — parse `<a href="...">` tags
- **JSON manifest** — `{"folders": ["a/", "b/"], "files": ["f.mp3"]}` atau array of strings

## Live Preview

Setiap perubahan di editor langsung tersimpan di `testData` (in-memory). Untuk test perubahan:

1. Switch ke tab **Ujian**
2. Klik **Mulai Ujian Sekarang**
3. Test soal yang baru diedit

Tidak ada explicit save — data persist hanya selama tab tidak di-reload. Export JSON untuk backup persistent.

## Shortcut Keys

(Tidak ada keyboard shortcut khusus. Editor full mouse-based.)

## Common Workflows

### Workflow 1: Buat tes dari scratch

1. Klik **+ Tambah Section** (atau pakai data default)
2. Set Section title, type, duration
3. Klik **+ Tambah Part** di dalam Section
4. Set Part title + instruction
5. Klik **+ Tambah Soal** di Part
6. Isi pertanyaan, 4 options, set correct answer
7. (Opsional) Set passage atau audio URL
8. Repeat untuk soal lainnya
9. **Export JSON** untuk backup

### Workflow 2: Import dari buku Longman

1. Copy text mentah dari PDF/buku Longman
2. Klik **Dump Longman** di toolbar editor
3. Pilih format dropdown (Auto / Listening / Structure / Reading / WE)
4. Paste text di textarea
5. Klik **Preview** untuk cek hasil parsing
6. Pilih target: Section/Part baru atau append ke existing
7. Klik **Import**

### Workflow 3: Buat group untuk Listening Part B

1. Tambah Part baru "Part B"
2. Klik **+ Tambah Soal** 4 kali (Q1-Q4)
3. Isi 4 soal yang berhubungan dengan 1 conversation
4. Klik **+ Group Baru** di header Part
5. Input "1-4" → semua 4 soal di-group
6. Edit Group passage: paste transkrip conversation
7. Edit Group audio URL: pakai 📁 Pilih dari Audio Bank

### Workflow 4: Move Part antar Section

1. Klik dropdown **↔ Pindah Section...** di header Part
2. Pilih target Section
3. Konfirmasi dialog
4. Part dipindah ke akhir Section target (dengan semua questions, groups, audio)

### Workflow 5: Edit cepat audio URL pakai picker

1. Klik tombol **📁 Pilih** di sebelah input audio URL
2. Modal terbuka, navigate folder bila perlu
3. Pakai filter untuk cari file cepat
4. Klik **Pakai** di file yang diinginkan
5. URL otomatis masuk ke input, tersimpan di testData

## Tips

- **Auto-collapse** soal hemat tempat. Klik header soal untuk expand satu saja.
- **Section sticky header** memudahkan navigasi panjang. Header tetap pinned saat scroll.
- **Underline marker** `~[kata]~(X)` untuk soal Written Expression — visual underline + tag answer letter.
- **Bulk dump** lebih cepat dari input manual untuk soal banyak. Pakai format yang sesuai.
- **Group berguna** untuk Listening Part B/C dan Reading. Jangan pakai group untuk Part A (short conversation).
- **Test sebelum publish**: switch ke Ujian tab, jalankan 1-2 soal untuk verifikasi audio, options, scoring.
