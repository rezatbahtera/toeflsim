# Architecture

Dokumen ini menjelaskan struktur aplikasi, hierarki data, dan state management.

## 1. File Structure (1 file, 3 layer)

```
ToeflSim.html
├── <head>          : meta, title, Tailwind CDN, inline custom CSS (~100 baris)
├── <body>          : HTML markup untuk semua view + modal
│   ├── #init-load-screen      (halaman pilih sumber data, default tab Bank Soal)
│   ├── #start-screen          (input nama peserta + mode selector)
│   ├── #view-test             (testing view: sidebar + content + bottom nav)
│   │   ├── #section-direction-screen   (z-30 overlay, gradient slate-blue)
│   │   ├── #direction-screen           (z-20 overlay, gradient blue-indigo, Part level)
│   │   ├── #score-modal                (hasil ujian dengan 4 tombol aksi)
│   │   ├── #finish-confirm-modal       (konfirmasi sebelum selesai)
│   │   └── #results-review-modal       (review per-section)
│   ├── #view-editor           (visual editor)
│   ├── #audio-picker-modal    (browse audio bank, filter)
│   ├── #bulk-dump-modal       (paste text dump parser)
│   └── #github-save-modal     (save ke Github API)
└── <script>        : ~5000 baris JavaScript vanilla (141 fungsi)
```

## 2. Schema Hierarki

```
testData
├── title: string
└── sections: [Section]
    ├── title: string                  ("Section 1 Listening Comprehension")
    ├── type: 'listening'|'structure'|'reading'
    ├── durationMinutes: number        (untuk Section timer)
    ├── instruction: string            (teks direction Section)
    ├── directionAudioUrl: string      (audio direction Section, opsional)
    └── parts: [Part]
        ├── title: string              ("Part A", "Part B")
        ├── instruction: string        (teks direction Part)
        ├── directionAudioUrl: string  (audio direction Part)
        ├── passage: string            (legacy, untuk Part tanpa groups)
        ├── audioUrl: string           (legacy, untuk Part shared audio)
        ├── durationSeconds: number    (Part timer, 0 = no timer)
        ├── groups: [Group]            (opsional)
        │   ├── id: string             (e.g. "g_abc123")
        │   ├── passage: string        (shared passage untuk multiple Q)
        │   └── audioUrl: string       (shared audio untuk multiple Q)
        └── questions: [Question]
            ├── groupId: string|null   (FK ke Group.id, kalau ada)
            ├── audioUrl: string       (audio per-soal, opsional)
            ├── passage: string        (passage per-soal, kalau tidak pakai group)
            ├── question: string       (teks pertanyaan, support marker ~[kata]~(A))
            ├── correctIndex: 0|1|2|3
            ├── timeLimitSeconds: number  (per-question timer, 0 = no limit)
            └── options: [Option × 4]
                ├── text: string
                └── reason: { en: string, id: string }
```

### Auto-migration

Saat `normalizeData()` dipanggil, struktur lama tanpa groups otomatis dimigrasi:

- Kalau Part punya `passage`/`audioUrl` tapi `groups` kosong dan ada `questions` → buat 1 group implicit yang dishare semua soal, kosongkan `part.passage`/`part.audioUrl`.
- Field undefined di-set ke default (string kosong, array kosong, sesuai konteks).
- Field `reason` di-normalize: kalau string → `{en, id: ''}`, kalau object missing field → fill kosong.

### Underline Marker

Untuk soal Written Expression (struktur dengan bagian bergaris bawah), gunakan marker inline:

```
She have ~[gone]~(A) to the ~[market]~(B) for ~[buying]~(C) groceries ~[yesterday]~(D).
```

- `~[kata]~(X)` → render sebagai `<u>kata</u>` di tampilan, dan tag huruf jawaban X.
- Marker dipreservasi di JSON, di-strip di preview/export.

## 3. State Management

Semua state global di top of `<script>` block (lihat `FUNCTIONS.md` untuk reference).

### Core test state

| Variable | Tipe | Deskripsi |
|---|---|---|
| `testData` | Object | Data tes lengkap (lihat schema di atas) |
| `cSectionIdx`, `cPartIdx`, `cQuestionIdx` | number | Pointer current question |
| `selectedAnswers` | `{[qKey]: number}` | qKey = `${s}_${p}_${q}` → optionIdx |
| `isTestRunning` | boolean | True saat user sedang ujian (bukan review) |
| `isReviewMode` | boolean | True setelah test selesai, masuk review |
| `testMode` | 'strict'\|'practice' | Mode dipilih di start screen |
| `participantName` | string | Nama peserta, dipakai di hasil & export |
| `lockedSections` | Set | Section yang sudah dikunci (strict mode) |
| `shuffledMap` | Array | Mapping shuffle pilihan ganda |

### Direction screen state

| Variable | Tipe | Deskripsi |
|---|---|---|
| `sectionDirectionShown` | `{[sIdx]: bool}` | Track Section direction sudah ditampilkan |
| `directionShown` | `{[partKey]: bool}` | Track Part direction sudah ditampilkan |
| `sectionEntered` | `{[sIdx]: bool}` | Track Section pertama kali dimasuki |
| `partEntered` | `{[partKey]: bool}` | Track Part pertama kali dimasuki |
| `deferredAudioFn` | function\|null | Audio question yang ditunda saat direction tampil |

### Timer state

| Variable | Tipe | Deskripsi |
|---|---|---|
| `sectionTimeRemaining` | Map | sIdx → detik tersisa |
| `partTimeRemaining` | Map | partKey → detik tersisa |
| `currentSectionTimerInterval` | number\|null | setInterval handle |
| `currentPartTimerInterval` | number\|null | setInterval handle |
| `currentQuestionTimerInterval` | number\|null | setInterval handle |
| `practiceTimerPaused` | boolean | True saat user pause timer di practice mode |

### Audio state

| Variable | Tipe | Deskripsi |
|---|---|---|
| `audioHasBeenPlayed` | `{[qKey]: bool}` | Track audio soal sudah diputar |
| `partAudioHasBeenPlayed` | `{[groupKey]: bool}` | Track group/part audio shared |
| `audioPlaybackInProgress` | boolean | Lock untuk prevent double-play |
| `audioGracePeriodTimeout` | number\|null | setTimeout handle untuk grace period |
| `audioProgressInterval` | number\|null | setInterval handle untuk progress display |

### Editor state

| Variable | Tipe | Deskripsi |
|---|---|---|
| `editorCollapsed` | `{[key]: bool}` | Section/Part collapse state |
| `editorQPage` | `{[partKey]: number}` | Current pagination page per Part |
| `editorQExpanded` | `{[qKey]: bool}` | Question card body expanded (auto-collapse others) |
| `currentActiveFilename` | string | Filename current data, untuk save target |

### Source & access

| Variable | Tipe | Deskripsi |
|---|---|---|
| `loadedFromBankSoal` | boolean | True kalau data dari Github `rezatbahtera/toeflsim` |
| `bankListCache` | Array\|null | Cache hasil scan Bank Soal list |
| `audioPickerTarget` | Object | Current target field untuk audio picker |
| `audioPickerCurrentPath` | string | Current path di audio bank navigation |
| `audioPickerLastList` | Object | Cache list untuk re-filter tanpa re-fetch |

## 4. Module Organization

JavaScript dipisah secara logical (bukan fisik) ke modul-modul:

| Modul | Fungsi-fungsi utama |
|---|---|
| **Bootstrap** | `DOMContentLoaded`, `loadDefaultData`, `initAppWithData`, `normalizeData` |
| **Source Loader** | `fetchBankSoalList`, `renderBankList`, `fetchJsonFromUrl`, `fetchFolderJsons`, `parseDirectoryListing`, `handleInitFileLoad` |
| **Start & Test Setup** | `setupStartScreen`, `startTest`, `loadTarget` |
| **View Switch** | `switchTab`, `switchSourceTab`, `confirmGoHome`, `showInitScreen` |
| **Sidebar Navigation** | `renderSidebar`, `openMobileSidebar`, `closeMobileSidebar` |
| **Direction Screens** | `showSectionDirectionScreen`, `dismissSectionDirectionScreen`, `showDirectionScreen`, `dismissDirectionScreen`, `isAnyDirectionScreenActive` |
| **Question UI** | `loadQuestionUI`, `selectOption`, `checkAnswer`, `togglePracticeCheck`, `hidePracticeCheck`, `renderPassageHtml`, `injectVisualLineNumbers` |
| **Audio Playback** | `playQuestionAudio`, `playAudioSequence`, `playDeferredAudioIfAny`, `stopAudioCompletely`, `stopDirectionAudiosOnly`, `startGracePeriod` |
| **Practice Controls** | `practiceReplayAudio`, `practiceToggleAudio`, `practiceToggleTimer`, `updatePracticeControlsVisibility` |
| **Timer** | `startSectionTimer`, `startPartTimer`, `startQuestionTimer`, `handleSectionTimeout`, `handlePartTimeout`, `updateSectionTimerDisplay`, `updatePartTimerDisplay`, `updateQuestionTimerDisplay` |
| **Navigation** | `navigateQuestion`, `updateNavigationState`, `confirmFinishTest`, `proceedFinishTest`, `finishTest`, `retakeTest`, `resetTest` |
| **Scoring** | `calculateITPTotal`, `rawToScaled`, `computeResultsBreakdown`, `showResultsReview` |
| **Editor UI** | `renderEditorUI`, `toggleEditorCollapse`, `toggleQuestionExpand`, `setEditorQPage`, `renderPaginationControls`, `scrollToEditorEl` |
| **Editor Mutations** | `addSection`, `addPart`, `addQuestion`, `addQuestionToGroup`, `deleteSection`, `deletePart`, `deleteQuestion`, `moveSection`, `movePart`, `movePartToSection`, `moveQuestion`, `updateDataState`, `updateGroupField`, `promptCreateGroup`, `ungroupQuestions` |
| **Bulk Dump Parser** | `openBulkDumpModal`, `processBulkDump`, `previewBulkDump`, `updateBulkExample`, `parseAuto`, `parseListening`, `parseListeningLong`, `parseStructure`, `parseWrittenExpression`, `parseReading` |
| **Audio Bank Picker** | `openAudioPicker`, `openAudioPickerForGroup`, `closeAudioPicker`, `audioPickerLoadCatalog`, `renderAudioPickerList`, `renderAudioPickerBreadcrumb`, `audioPickerSelect`, `audioPickerApplyFilter`, `audioPickerClearFilter`, `configAudioBank`, `parseAudioCatalogHtml`, `parseGcsXmlListing`, `toGcsListUrl`, `isGcsBucketUrl` |
| **Export** | `downloadResults`, `downloadResultsCsv`, `downloadResultsHtml`, `buildHtmlReport`, `toggleDownloadMenu`, `closeDownloadMenu`, `exportDataToJson` |
| **Access Control** | `isAuthorizedEditor`, `applyEditorAccessControl` |
| **Utilities** | `escapeText`, `escapeAttr`, `formatTime`, `shuffleArray`, `newGroupId`, `getGroupForQuestion`, `getQuestionsInGroup`, `getGlobalQNumber`, `getSectionQCount`, `showNotification`, `autoGrowTextarea`, `initAutoGrow`, `groupConsecutive` |

Total: **141 fungsi**.

## 5. Z-Index Stack

Saat overlay muncul, urutan stacking dari belakang ke depan:

| Layer | Element | z-index |
|---|---|---|
| 0 | Sidebar drawer overlay (mobile) | 19 |
| 1 | Part direction screen | 20 |
| 2 | Section direction screen | 30 |
| 3 | Score modal | 50 |
| 4 | Bulk dump modal | 55 |
| 5 | Audio picker modal | 60 |
| 6 | Github save modal | 65 |
| 7 | Notifications (top-right toast) | 100 |

Section direction selalu di atas Part direction karena Section dimuat lebih awal di flow (chained: Section → Part → soal).

## 6. Responsive Breakpoints

- **`< md` (768px)**: mobile layout — sidebar jadi drawer (hamburger button), bottom nav single row, score modal 95vw
- **`md+`**: desktop — sidebar persistent left, content full width

CSS custom:

```css
body { height: 100vh; height: 100dvh; padding-bottom: env(safe-area-inset-bottom); }
```

`100dvh` (dynamic viewport height) memperbaiki bug mobile dimana `100vh` overflow karena address bar.

## 7. Persistence (localStorage)

| Key | Value | Set saat |
|---|---|---|
| `participantName` | string (max 80 char) | `startTest()` saat user submit |
| `audioBankUrl` | string URL | `configAudioBank()` atau `audioPickerLoadCatalog()` |

Tidak ada session state lain di localStorage — semua state dalam memory dan reset saat reload.
