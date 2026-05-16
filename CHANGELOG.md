# Changelog

Riwayat perubahan dan iterasi pengembangan `ToeflSim.html`.

## v1.0 — Baseline

Initial version dengan fitur dasar:

- Schema Section → Part → Question (no groups yet)
- Linear navigation
- Section timer + question timer
- TOEFL ITP scoring formula
- JSON load/save
- Simple test mode (no strict/practice differentiation)

## v1.1 — Groups & Bulk Dump

- **Schema diperluas**: Group level antara Part dan Question (untuk Part B/C listening, Reading sets)
- **Auto-migration**: legacy `part.passage`/`part.audioUrl` → group implicit saat load
- **Bulk Dump Parser**: 6 parsers (Listening short/long, Structure, Written Expression, Reading, Auto)
- **Direction detection** di parser: `Part A Directions:`, `Section X Directions:` dll
- **Underline marker** `~[kata]~(A)` untuk Written Expression

## v1.2 — Test Modes & UI Polish

- **Strict mode (Real Test)**: section dikunci setelah lewat, sidebar nav restricted, audio 1x
- **Practice mode (Latihan)**: free nav, Cek Pembahasan toggle, audio bisa replay
- **Sticky section header** di editor untuk navigasi panjang
- **Sticky part header** (kemudian DIHAPUS karena bug blank space)
- **Mobile layout**: sidebar drawer dengan hamburger button, bottom nav compact
- **Mobile viewport fix**: `100dvh` + `env(safe-area-inset-bottom)`
- **Score modal**: 4 tombol grid (Pembahasan, Download, Ringkasan, Ulangi)

## v1.3 — Direction Screen System

- **Section direction screen** (z-30, gradient dark slate-blue) — overlay full-screen saat first entry ke Section
- **Part direction screen** (z-20, gradient blue-indigo) — overlay saat first entry ke Part
- **Chained transition**: Section direction → Part direction (kalau keduanya ada)
- **Audio direction**: bisa play narrator audio bareng dengan text instruction
- **Tracking state**: `sectionDirectionShown`, `directionShown`, `sectionEntered`, `partEntered`
- **Deferred question audio**: audio soal ditunda kalau direction screen aktif, play setelah dismiss
- **State reset** di retakeTest, setupStartScreen

## v1.4 — Audio Bank Picker

- **Modal "Pilih dari Bank Audio"** untuk browse file mp3 dari catalog
- **Support 3 source format**:
  - GCS bucket (`storage.googleapis.com/BUCKET/...`) → auto-convert ke JSON list API
  - Apache/nginx HTML directory listing → parse `<a>` tags
  - JSON manifest `{folders: [], files: []}`
- **Breadcrumb navigation** untuk folder hierarchy (e.g. `soal/section/part/file.mp3`)
- **Inline audio preview** di tiap file
- **Tombol 📁 Pilih** di sebelah tiap input audio URL (question, group, part, section direction, part direction)
- **localStorage `audioBankUrl`** untuk persist konfigurasi
- **Tombol toolbar "🎵 Bank Audio"** di editor untuk config base URL

## v1.5 — Editor UX Improvements

- **Auto-collapse soal**: clicking 1 soal expand it + auto-collapse semua yang lain di Part sama
- **Header soal**: preview pertanyaan + correct letter badge
- **Pagination group-aware**: group tidak split ke 2 halaman (greedy-fill algorithm)
- **Move Part antar Section**: dropdown "↔ Pindah Section..." di Part header
- **Visual line numbering** di passage (Range API + getClientRects)
- **Underline marker shortcut Ctrl+\`** (kemudian DIHAPUS per user request)

## v1.6 — Navigation Polish

- **Sidebar render order fix**: `renderSidebar()` dipanggil **setelah** `cQuestionIdx = qIdx`, jadi group highlight match dengan soal aktif
- **Home button** di top bar: konfirmasi dialog mid-test, force-stop semua, return ke init screen
- **Tombol Lanjut dismiss direction**: saat direction screen aktif dan user pencet Next button, intercept dan dismiss overlay (sama dengan tombol Mulai biru di tengah)

## v1.7 — Practice Controls

- **Practice quick-controls** di bottom nav kiri:
  - 🔄 Replay audio
  - ⏸ Pause/Play audio
  - ⏱ Pause/Resume timer
- **Guard runtime**: tombol di-disabled visual + reject runtime kalau direction screen aktif (prevent accidental trigger audio soal saat instruction audio play)
- **Pause timer**: pause semua 3 timer (section + part + question), resume restart dari nilai sisa

## v1.8 — Filter Audio Picker

- **Filter keyword** di Audio Bank picker dengan real-time search (case-insensitive)
- **Counter** "N / M items" untuk total filtered vs all
- **Empty state** dengan tombol "Hapus filter" kalau no match
- **Filter auto-reset** saat navigate ke folder lain
- **Cache `audioPickerLastList`** untuk re-filter tanpa re-fetch

## v1.9 — Direction Audio-Only Support

- **Bug fix**: direction screen tidak muncul kalau hanya audio terisi (tanpa text instruction). Sebelumnya: condition cek `instruction.trim() !== ''`
- **Fix**: condition diperluas ke `instruction || directionAudioUrl`
- **Placeholder text** saat audio-only: "(Direction audio diputar — silakan dengarkan sebelum mulai.)"
- **Chained transition** Section→Part juga update untuk audio-only

## v1.10 — Audio Stop Bug Fix

- **Bug**: `stopAudioCompletely()` hanya stop `#question-audio`, tidak menyentuh direction audios. Saat user pencet Home/Lanjut/Next saat direction audio playing → audio direction lanjut play di background
- **Fix**: `stopAudioCompletely()` sekarang stop SEMUA 3 audio elements (question + section direction + part direction). Pakai `pause() + currentTime=0 + src=''`
- **Helper baru**: `stopDirectionAudiosOnly()` untuk selective stop
- **Removed**: tombol Lanjutkan kuning di direction screen (user tidak butuh, dual-button confusing)
- **dismissDirectionScreen always stops audio**: parameter `keepAudio` retained untuk backward-compat tapi diabaikan

## v1.11 — Download Formats

Sebelumnya: hanya JSON murni (UX kurang user-friendly).

Sekarang: dropdown 3 pilihan:

1. **📄 Laporan HTML (Rekomendasi default)** — self-contained, inline CSS, print/Save PDF native via `window.print()`. Beautiful gradient header, hero score cards, per-section table, detail soal dengan row colored
2. **📊 CSV Spreadsheet** — UTF-8 BOM untuk Excel, header info peserta + per-question rows + summary footer
3. **🔧 JSON Raw Data** — format lama untuk developer/integrasi

- `computeResultsBreakdown` tambah `scaledScore` per section
- `buildHtmlReport(breakdown)` compose HTML string
- Toggle dropdown dengan click-outside handler

## v1.12 — Participant Name

- **Field "Nama Peserta"** di start screen sebelum mode selector
- **Auto-focus** saat halaman start screen terbuka
- **Pre-fill** dari `localStorage.participantName`
- **Validasi**: kalau kosong → confirm "Lanjutkan sebagai 'Peserta Anonim'?"
- **Display** di:
  - Score modal: `👤 Nama Peserta` badge
  - HTML report header
  - CSV info rows
  - JSON breakdown field `participantName`
  - Filename: `{nama}_{judul}_{date}.{ext}`
- **localStorage persist** untuk sesi berikutnya

## v1.13 — Score Fix & Editor Access Control

### Bug fix

- **Score 0 di export**: `computeResultsBreakdown` baca `#score-total-scaled` tapi ID actual `#score-itp-total` → semua format dapat score 0. Fix: ganti ID reference.

### Editor access control

- **Global flag `loadedFromBankSoal`**: true hanya saat data dari Bank Soal Github
- **`isAuthorizedEditor()`**: cek nama peserta = "cakresa" (case-insensitive)
- **`applyEditorAccessControl()`**: hide tab Editor kalau (loadedFromBankSoal && !authorized)
- **Hooks**: di DOMContentLoaded, tiap loader, startTest, input event di nama field (live update)
- **Behavior**: file lokal/URL/default tetap bebas akses Editor. Hanya data master Bank Soal yang restricted.

## v1.14 — Bug Fix: Practice Controls & Direction

- **Tombol Lanjut sekarang dismiss direction screen**: saat user di layar instruction pencet Next button bottom nav, intercept dan dismiss (sama dengan tombol Mulai biru). Mengurangi dual-button confusion.
- **btn-check (Cek Pembahasan) ikut di-disable**: saat direction screen aktif, btn-check juga di-disable bareng practice controls. `togglePracticeCheck()` juga guarded.
- **Re-enable otomatis** saat direction dismissed via tombol manapun (Mulai atau Lanjut).

---

## Architectural Decisions

### Why single HTML file?

- Zero deploy complexity (upload ke static hosting apa saja)
- No build step, no node_modules, no version mismatch
- Easy backup & share (drag ke email, WA, drive)
- Offline-capable setelah load awal
- Easy debugging (View Source langsung)

Trade-off: file size ~365 KB (acceptable, ~5x size halaman web normal). Kalau perlu split nanti, modular `<script type="module">` bisa diapply.

### Why no React/Vue?

Vanilla JS sufficient untuk scope ini. Tidak butuh complex state management. DOM manipulation langsung lebih predictable. Bundle size tetap kecil.

### Why Tailwind via CDN?

- No build step (compatible dengan single-file approach)
- Familiar utility-first
- Trade-off: file pengguna butuh internet pertama kali load (cached after)
- Inline custom CSS untuk hal yang tidak dicover Tailwind (z-index layers, mobile drawer, etc)

### Why no server?

- Tidak ada user accounts (cukup nama peserta untuk identifikasi laporan)
- Tidak ada cross-user data sharing (data tes shared via Github repo public)
- Tidak ada real-time features
- Tidak ada secrets server-side

Editor saves via Github PUT API (user supply own PAT). Audio dari public GCS bucket. Test data dari public Github contents API. Semua serverless dari sudut pandang aplikasi.

### Why GCS untuk audio?

- Hosting murah untuk file binary besar
- Public access mudah dikonfigurasi
- Hierarchical folder structure cocok dengan model `soal/section/part/file.mp3`
- XML/JSON listing API standardized

### Why Github repo untuk Bank Soal?

- Free hosting untuk JSON (max 100 MB per file)
- Versioning built-in
- Public access via raw URL atau contents API
- Editor user "cakresa" punya akses write via PAT
- User publik baca-only

## Statistics

- **Total fungsi**: 141
- **Lines of JS**: ~5000
- **Lines of HTML**: ~900
- **Lines of CSS** (inline): ~100
- **Total file size**: ~365 KB
- **Number of HTML elements at runtime**: ~50 (DOM-static), unlimited (per question render)
- **Browser support**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Mobile support**: iOS 14+, Android Chrome
