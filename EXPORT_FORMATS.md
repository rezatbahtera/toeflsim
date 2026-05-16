# Export Formats

3 format download yang tersedia setelah ujian selesai. Akses via tombol **Download ▾** di score modal.

## Format Comparison

| Aspek | 📄 HTML Report | 📊 CSV | 🔧 JSON |
|---|---|---|---|
| Visual menarik | ✓✓ | ✗ | ✗ |
| Buka di mana saja | Browser | Excel/Sheets | Text editor |
| Print/Save PDF | ✓ Native (Ctrl+P) | Via Excel | Tidak |
| Bisa di-share via WA/Email | ✓ | ✓ | Tidak ideal |
| Analisis data | Terbatas | ✓✓ | ✓✓ |
| Developer integration | Tidak | Parse manual | ✓✓ JSON.parse |
| File size | ~9-30 KB | ~5-15 KB | ~10-30 KB |
| Self-contained | ✓ (inline CSS) | ✓ | ✓ |

**Rekomendasi default**: HTML Report — terbaik untuk user akhir.

## 1. HTML Report

File self-contained dengan inline CSS, bisa dibuka di browser apa saja, mendukung print/save PDF native.

### Filename

```
{participantName}_{title}_{date}.html
```

Contoh: `Budi_Santoso_pretest-1_2026-05-16.html`

- `participantName`: sanitized (non-alphanumeric → `_`), truncated 30 char
- `title`: sanitized, truncated 40 char
- `date`: `YYYY-MM-DD` (ISO date portion)

### Struktur

```
┌────────────────────────────────────────┐
│  Header (gradient biru)                │
│  📊 Hasil Ujian TOEFL ITP              │
│  {test title}                          │
│  ──────────────────────────────────    │
│  👤 {nama peserta}     Dicetak: {tgl}  │
├────────────────────────────────────────┤
│  Tip kuning + tombol 🖨 Print/Save PDF │
├────────────────────────────────────────┤
│  Score Hero (5 stat cards)             │
│  ┌──────┐ ┌─────┐ ┌─────┐ ┌────┐ ┌──┐│
│  │ 540  │ │ 25  │ │  5  │ │ 0  │ │83%││
│  │ ITP  │ │Bnr  │ │Slh  │ │Ksn │ │Akur││
│  └──────┘ └─────┘ └─────┘ └────┘ └──┘│
├────────────────────────────────────────┤
│  📈 Ringkasan per Section               │
│  Section | Total | B | S | K | % | Sclg│
│  L      | 30    | 25| 5 | 0 |83%| 56  │
│  S      | 40    | 32| 8 | 0 |80%| 58  │
│  R      | 50    | 38|10 | 2 |76%| 55  │
├────────────────────────────────────────┤
│  📋 Detail Soal                         │
│  Section 1 Listening (25 dari 30)      │
│  No │ Status │ Jwb │ Bnr │ Pertanyaan │
│  1  │ ✓ Bnr  │ A   │ A   │ What does..│
│  2  │ ✗ Slh  │ B   │ C   │ What does..│
│  ... (row colored: hijau/merah/abu)    │
├────────────────────────────────────────┤
│  Footer (TOEFL ITP Simulator Pro)      │
└────────────────────────────────────────┘
```

### Print Stylesheet

`@media print`:

- `@page { margin: 18mm 12mm }` — print margin
- Hide tip + Print button (`.actions { display: none }`)
- Background colors preserved (`-webkit-print-color-adjust: exact`)
- Table page-break-inside: avoid (jangan split row)
- Headers (`h2, h3`) page-break-after: avoid

Saat user pencet **Ctrl+P** (atau klik tombol Print di laporan):

1. Browser print dialog terbuka
2. User pilih destination: **Save as PDF**
3. Hasil PDF berwarna lengkap dengan struktur tabel utuh

### Self-Containment

- Tidak ada external CSS/font/JS
- Tidak ada `<script>` (kecuali inline `onclick` di Print button → `window.print()`)
- Image-less (emoji unicode only)
- Bisa di-email, di-share via WA, dibuka offline

## 2. CSV Spreadsheet

File `.csv` dengan UTF-8 BOM agar Excel buka dengan encoding benar.

### Filename

```
{participantName}_{title}_{date}.csv
```

### Struktur

```csv
Nama Peserta,Budi Santoso
Tes,TOEFL ITP Praktik 1
Tanggal,16/05/2026 10.30.00

No,Section,Part,Status,Jawaban Anda,Jawaban Benar,Pertanyaan
1,Section 1 Listening,Part A,Benar,A,A,"What does the woman mean?"
2,Section 1 Listening,Part A,Salah,B,C,"What does the man imply?"
3,Section 1 Listening,Part A,Kosong,—,A,"How does the speaker..."
...

RINGKASAN
Total Score (ITP),540
Section 1 Listening,25/30,83%
Section 2 Structure,32/40,80%
Section 3 Reading,38/50,76%
```

### Encoding

UTF-8 dengan **BOM** (`\ufeff` di awal file) supaya Excel detect UTF-8 dan render karakter Indonesian/Unicode dengan benar.

### Escape Rules

- Field dengan `,` `"` atau `\n` → wrap dalam `"..."` dan double-quote internal `""`
- Lainnya → plain
- Pertanyaan truncated ke 250 char (untuk readability)

### Use Cases

- **Analisis di Excel/Sheets**: filter by status (Benar/Salah/Kosong), pivot table per Section, chart akurasi
- **Backup quick**: simple text yang bisa dibuka di text editor
- **Bulk grading**: kalau ada banyak hasil user, import semua CSV → consolidated workbook

## 3. JSON Raw Data

Format mentah untuk developer atau integrasi system.

### Filename

```
{participantName}_{title}_{date}.json
```

### Struktur

```json
{
  "title": "TOEFL ITP Praktik 1",
  "participantName": "Budi Santoso",
  "generatedAt": "2026-05-16T10:30:00.000Z",
  "totalScaled": 540,
  "sections": [
    {
      "title": "Section 1 Listening",
      "type": "listening",
      "correct": 25,
      "wrong": 5,
      "empty": 0,
      "total": 30,
      "scaledScore": 56,
      "parts": [
        {
          "title": "Part A",
          "questions": [
            {
              "number": 1,
              "question": "What does the woman mean?",
              "userAnswerLetter": "A",
              "correctAnswerLetter": "A",
              "status": "correct",
              "groupId": null
            },
            {
              "number": 2,
              "question": "What does the man imply?",
              "userAnswerLetter": "B",
              "correctAnswerLetter": "C",
              "status": "wrong",
              "groupId": null
            }
          ]
        }
      ]
    }
  ]
}
```

### Field Reference

#### Root

| Field | Type | Notes |
|---|---|---|
| `title` | string | Dari `testData.title` |
| `participantName` | string | Dari `participantName` global, fallback "Peserta Anonim" |
| `generatedAt` | string ISO | `new Date().toISOString()` |
| `totalScaled` | number | 310-677 |
| `sections` | array | Per Section breakdown |

#### Section

| Field | Type | Notes |
|---|---|---|
| `title` | string | |
| `type` | 'listening' \| 'structure' \| 'reading' | |
| `correct`, `wrong`, `empty`, `total` | number | Counts |
| `scaledScore` | number | 31-68 (L/S) atau 31-67 (R) |
| `parts` | array | Per Part breakdown |

#### Question

| Field | Type | Notes |
|---|---|---|
| `number` | number | Global Q number dalam Section |
| `question` | string | Teks pertanyaan, marker `~[]~` preserved |
| `userAnswerLetter` | 'A'\|'B'\|'C'\|'D'\|'—' | User pilih atau `—` kalau kosong |
| `correctAnswerLetter` | 'A'\|'B'\|'C'\|'D' | Jawaban benar |
| `status` | 'correct'\|'wrong'\|'empty' | |
| `groupId` | string\|null | Reference ke group kalau ada |

### Use Cases

- **Integrasi LMS**: parse di backend untuk record hasil ke database
- **Re-import**: rebuild test from JSON kalau perlu (lossless)
- **Programmatic analysis**: Python pandas, R, atau scripting tools
- **Backup developer**: complete dataset hasil ujian

## 4. Download Menu UX

Dropdown trigger di score modal:

```
┌─────────────────────────────────┐
│  Download ▾                     │
└─────────────────────────────────┘
        │ (click)
        ▼
┌───────────────────────────────────────┐
│ 📄 Laporan HTML  [Rekomendasi]        │
│    Bisa dibuka di browser, print, PDF │
├───────────────────────────────────────┤
│ 📊 CSV Spreadsheet                    │
│    Buka di Excel/Google Sheets        │
├───────────────────────────────────────┤
│ 🔧 JSON Raw Data                      │
│    Format data mentah untuk developer │
└───────────────────────────────────────┘
```

- Klik di luar menu → auto-close (event listener)
- Tap pilihan → trigger download + close menu
- Notifikasi sukses muncul setelah download
- Color-coded hover: HTML biru, CSV hijau, JSON abu

## 5. Cross-Format Consistency

Semua 3 format pakai output dari `computeResultsBreakdown()`:

- Score angka sama persis (totalScaled, scaledScore per section, correct count)
- Participant name sama
- Pertanyaan, jawaban user, jawaban benar konsisten
- Hanya struktur file yang beda (visual HTML vs flat CSV vs nested JSON)

## 6. Score Computation

Dari raw answers ke totalScaled:

```js
// Untuk tiap section
for (let q of section.questions) {
    const userAnswer = selectedAnswers[`${s}_${p}_${q}`];
    if (userAnswer === undefined) empty++;
    else if (userAnswer === q.correctIndex) correct++;
    else wrong++;
}

// Convert ke scaled
const pct = correct / total;
const curve = Math.pow(pct, 0.85);  // forgiving curve
const max = (type === 'reading') ? 67 : 68;
scaledScore = Math.round(31 + curve * (max - 31));

// Total ITP
totalScaled = Math.round((L_scaled + S_scaled + R_scaled) / 3 * 10);
totalScaled = Math.max(310, Math.min(677, totalScaled));  // clamp
```

Curve `pow(0.85)` membuat skor sedikit forgiving — 50% benar tidak langsung jadi 50% dari range, melainkan ~56% (lebih ke arah linear).
