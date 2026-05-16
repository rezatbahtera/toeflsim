# Flow & Logic

Dokumen ini menjelaskan alur user, state transitions, dan logic kompleks (direction screen, audio, scoring).

## 1. Application Flow

```
┌─────────────────────────────┐
│  index.html dibuka          │
│  DOMContentLoaded fires     │
└───────────┬─────────────────┘
            ▼
┌─────────────────────────────┐
│  Init Screen                │
│  - Tab: Bank Soal (default) │
│  - Tab: URL                 │
│  - Tab: Folder URL          │
│  - Tab: Upload File         │
│  - Tab: Default             │
└───────────┬─────────────────┘
            │ initAppWithData(data)
            ▼
┌─────────────────────────────┐
│  Start Screen               │
│  - Title + section summary  │
│  - Nama Peserta (input)     │
│  - Mode: Real Test/Latihan  │
│  [Mulai Ujian Sekarang]     │
└───────────┬─────────────────┘
            │ startTest()
            ▼
┌─────────────────────────────┐
│  loadTarget(0, 0, 0)        │
│  ├─ Check Section direction │
│  ├─ Check Part direction    │
│  └─ Load question UI        │
└───────────┬─────────────────┘
            │
            ▼
┌─────────────────────────────┐
│  Test View                  │
│  - Sidebar nav              │
│  - Section/Part/Q timer     │
│  - Passage + audio          │
│  - Options (radio)          │
│  - Bottom nav: Prev/Next    │
│  - Practice controls (opt)  │
└───────────┬─────────────────┘
            │ navigateQuestion(1) ke akhir
            │ confirmFinishTest()
            ▼
┌─────────────────────────────┐
│  Finish Confirm Modal       │
│  "Yakin Selesai?"           │
└───────────┬─────────────────┘
            │ proceedFinishTest()
            ▼
┌─────────────────────────────┐
│  Score Modal                │
│  - Score 310-677            │
│  - Per-section breakdown    │
│  - [Pembahasan]             │
│  - [Download ▾]             │
│  - [Ringkasan]              │
│  - [Ulangi]                 │
└─────────────────────────────┘
```

## 2. Source Loading Flow

### Bank Soal (default, GitHub repo `rezatbahtera/toeflsim`)

1. `DOMContentLoaded` → `fetchBankSoalList()`
2. Fetch `https://api.github.com/repos/rezatbahtera/toeflsim/contents/banksoal`
3. Parse JSON contents array → daftar file
4. Untuk ≤12 file: pre-fetch tiap file untuk dapatkan `title` dan `sectionCount` metadata
5. `renderBankList(files)` tampilkan list dengan icon + title
6. User klik file → fetch JSON → `initAppWithData()` → `loadedFromBankSoal = true` → `applyEditorAccessControl()`

### URL eksternal

1. User paste URL di tab "URL"
2. `fetchJsonFromUrl()` → fetch + parse JSON
3. `initAppWithData()` → `loadedFromBankSoal = false`

### Folder URL

1. User paste URL folder (Github API atau folder Apache/nginx)
2. `fetchFolderJsons()` parse listing pakai `parseDirectoryListing()`
3. Tampilkan list file → user pilih → load

### Upload File

1. User select file `.json` via `<input type="file">`
2. `handleInitFileLoad(event)` → FileReader → JSON.parse → `initAppWithData()`

### Default

1. `loadDefaultData()` → clone `defaultData` constant → `initAppWithData()`

## 3. Direction Screen Logic

Direction screen muncul saat user **pertama kali** masuk Section atau Part. Dikontrol oleh tracking state.

### State variables

```js
sectionDirectionShown = { [sIdx]: bool }   // Section direction sudah ditampilkan
directionShown        = { [partKey]: bool } // Part direction sudah ditampilkan
sectionEntered        = { [sIdx]: bool }   // Section sudah dimasuki
partEntered           = { [partKey]: bool } // Part sudah dimasuki
```

`partKey` = `"${sIdx}_${pIdx}"`.

### Trigger logic di `loadTarget(sIdx, pIdx, qIdx)`

```js
const wasDiffSec = (cSectionIdx !== sIdx) || !sectionEntered[sIdx];
const wasDiffPart = (cSectionIdx !== sIdx) || (cPartIdx !== pIdx) || !partEntered[partKey];

if (isTestRunning && !isReviewMode && qIdx === 0) {
    const secHasDir = section.instruction || section.directionAudioUrl;
    if (wasDiffSec && secHasDir && !sectionDirectionShown[sIdx]) {
        showSectionDirectionScreen(sIdx);  // z-30, gradient slate-blue
    }
    else {
        const partHasDir = part.instruction || part.directionAudioUrl;
        if (wasDiffPart && partHasDir && !directionShown[partKey]) {
            showDirectionScreen(sIdx, pIdx);  // z-20, gradient blue-indigo
        }
    }
}
```

Kunci: trigger HANYA saat `qIdx === 0` (soal pertama dari Part). Naviasi back-and-forth dalam Part tidak re-trigger.

### Chained transition Section → Part

Saat user dismiss Section direction (`dismissSectionDirectionScreen()`):

1. Set `sectionDirectionShown[sIdx] = true`
2. Cek apakah Part juga punya direction
3. Kalau ada: `showDirectionScreen(sIdx, pIdx)` (Part direction muncul langsung)
4. Kalau tidak ada: start timers + play deferred audio

### Audio direction & question audio deferral

Saat direction screen tampil dengan audio, **audio question DIBLOK** sampai direction dismissed:

```js
// di loadQuestionUI:
if (!isReviewMode) {
    const audioFn = () => playQuestionAudio(q, qKey);
    if (isAnyDirectionScreenActive()) {
        deferredAudioFn = audioFn;  // tunda
    } else {
        setTimeout(audioFn, 250);   // play normal
    }
}

// di dismissDirectionScreen:
playDeferredAudioIfAny();  // execute deferred audio sekarang
```

`isAnyDirectionScreenActive()` cek apakah salah satu overlay tidak `hidden`.

### Dismissal: Mulai Soal button vs Next button

User punya 2 cara dismiss direction screen:

1. **Tombol "Mulai Section / Mulai Soal"** biru di tengah overlay
2. **Tombol "Lanjut"** biru di pojok kanan bawah bottom nav

Keduanya panggil flow yang sama (`dismissDirectionScreen(false)` atau `dismissSectionDirectionScreen(false)`). Implementasi di `navigateQuestion`:

```js
function navigateQuestion(step, forcedByTimer = false) {
    if (!forcedByTimer && step === 1 && isAnyDirectionScreenActive()) {
        // Intercept: Next = Dismiss direction
        if (sectionDirActive) dismissSectionDirectionScreen(false);
        else if (partDirActive) dismissDirectionScreen(false);
        return;  // BUKAN navigate ke Q berikutnya
    }
    // ... normal navigation ...
}
```

## 4. Audio Sequence Logic

### Single question audio

`playQuestionAudio(qData, qKey)`:

1. Cek `audioHasBeenPlayed[qKey]` — kalau true, skip (audio 1x play di mode ujian)
2. Cek apakah ada group audio shared (`group.audioUrl`)
3. Build sequence:
   - Kalau ada group audio dan belum played → play group audio dulu
   - Lalu play question audio
4. `playAudioSequence(urls, callback)` chain audio elements via `ended` event
5. Setelah audio terakhir selesai → `startGracePeriod(seconds)` countdown sebelum auto-advance

### Grace period

Setelah audio selesai, user diberi waktu untuk pilih jawaban:

- Default: 10 detik
- Override: `question.timeLimitSeconds` (kalau > 0)
- Counter visible di status card

Setelah grace period habis → `navigateQuestion(1, true)` auto-advance.

### Stop audio scenarios

`stopAudioCompletely()` dipanggil saat:

- User navigate ke soal lain (Prev/Next)
- User klik Home button
- Test berakhir / finish
- Retake test

Stop SEMUA audio element:

```js
['question-audio', 'section-direction-audio', 'part-direction-audio'].forEach(id => {
    const el = document.getElementById(id);
    if (el) { el.pause(); el.currentTime = 0; el.src = ''; }
});
```

`src = ''` penting agar browser release media buffer (fix bug audio ghost lanjut play).

## 5. Timer Logic

### Section Timer

- Started saat user dismiss direction screen pertama kali masuk Section
- `currentSectionTimerInterval = setInterval(updateSectionTimerDisplay, 1000)`
- Tampil di top header: format `MM:SS`
- Saat habis → `handleSectionTimeout(sIdx)`:
  1. Lock section: `lockedSections.add(sIdx)`
  2. Force navigate ke Section berikutnya (atau finish test kalau last)
  3. Disable sidebar nav ke section ini

### Part Timer

- Started kalau `part.durationSeconds > 0`
- Independent dari Section timer
- Saat habis → `handlePartTimeout()`:
  1. Force navigate ke Part berikutnya
  2. Tidak lock section

### Question Timer

- Started saat `playAudioSequence` selesai dan question punya `timeLimitSeconds > 0`
- Atau grace period (default 10s) kalau timeLimitSeconds == 0
- Saat habis → auto-advance ke soal berikutnya

### Practice mode pause

`practiceToggleTimer()` pause/resume semua 3 timer:

```js
// Pause:
clearInterval(currentSectionTimerInterval);
clearInterval(currentPartTimerInterval);
clearInterval(currentQuestionTimerInterval);
practiceTimerPaused = true;

// Resume:
startSectionTimer(cSectionIdx);
if (part.durationSeconds > 0) startPartTimer(cSectionIdx, cPartIdx);
practiceTimerPaused = false;
```

## 6. Scoring Logic

### Raw count

Untuk tiap section, hitung:

- `correct`: soal dengan `selectedAnswers[qKey] === q.correctIndex`
- `wrong`: soal dengan answer tapi salah
- `empty`: soal tanpa jawaban
- `total`: total soal di section

### Raw to Scaled conversion

TOEFL ITP scaled score per section: 31-68 (Listening, Structure) atau 31-67 (Reading).

```js
function rawToScaled(correct, total, type) {
    if (total === 0) return 31;
    const pct = correct / total;
    const curve = Math.pow(pct, 0.85);  // power curve, sedikit forgiving
    const max = (type === 'reading') ? 67 : 68;
    const scaled = Math.round(31 + curve * (max - 31));
    return Math.max(31, Math.min(max, scaled));
}
```

### Total ITP score

```js
function calculateITPTotal(sectionScores) {
    // Standard formula: (L + S + R) / 3 * 10
    const sum = sectionScores.reduce((a, s) => a + s.scaled, 0);
    const total = Math.round((sum / sectionScores.length) * 10);
    return Math.max(310, Math.min(677, total));
}
```

### Score breakdown

`computeResultsBreakdown()` build object lengkap untuk export:

```js
{
    title: "Test Title",
    participantName: "User Name",
    generatedAt: "2026-05-16T...",
    totalScaled: 540,
    sections: [
        {
            title: "Section 1 Listening",
            type: "listening",
            correct: 25, wrong: 5, empty: 0, total: 30,
            scaledScore: 56,
            parts: [
                {
                    title: "Part A",
                    questions: [
                        {
                            number: 1,
                            question: "What does the woman mean?",
                            userAnswerLetter: "A",
                            correctAnswerLetter: "B",
                            status: "wrong"  // 'correct' | 'wrong' | 'empty'
                        }
                    ]
                }
            ]
        }
    ]
}
```

## 7. Mode Differences

### Strict (Real Test)

- Section dikunci setelah dilewati (no Prev across sections)
- Sidebar nav: hanya bisa klik soal di Section/Part aktif
- Cek Pembahasan: hidden
- Practice controls: hidden
- Audio: 1x play per soal (tracking `audioHasBeenPlayed`)
- Saat Section timer habis: auto-lock + auto-advance

### Practice (Latihan)

- Free navigation antar Section/Part
- Sidebar nav full (klik manapun)
- Cek Pembahasan: visible, toggle
- Practice controls: visible (replay, pause audio, pause timer)
- Audio: bisa replay manual
- Timer bisa di-pause

## 8. Pagination Group-Aware

Editor menampilkan max 10 soal per halaman per Part. Tapi **group tidak boleh split** ke 2 halaman.

Algoritma `greedy-fill` di `renderEditorUI`:

```js
let cur = [];
let i = 0;
while (i < allQs.length) {
    const q = allQs[i];
    let unitEnd = i + 1;
    if (q.groupId) {
        while (unitEnd < allQs.length && allQs[unitEnd].groupId === q.groupId) unitEnd++;
    }
    const unitSize = unitEnd - i;
    // Kalau current page sudah ada isinya DAN tambah unit ini akan exceed:
    // tutup current page, mulai page baru
    if (cur.length > 0 && cur.length + unitSize > 10) {
        pageBoundaries.push([cur[0], cur[cur.length - 1] + 1]);
        cur = [];
    }
    // Tambah unit ke current page
    for (let j = i; j < unitEnd; j++) cur.push(j);
    i = unitEnd;
}
```

**Contoh:** 3 group × 4 soal = 12 soal, max 10/page

- **Page 1:** g1 (Q1-4) + g2 (Q5-8) = 8 soal (g3 tidak fit, start page baru)
- **Page 2:** g3 (Q9-12) = 4 soal

Group banner muncul di occurrence pertama group di page tersebut.

## 9. Access Control Logic

`applyEditorAccessControl()` dipanggil di:

- `DOMContentLoaded`
- Tiap loader yang set `loadedFromBankSoal` flag
- `startTest()` setelah nama di-lock
- `input` event di field `#participant-name` (live update)

Logic:

```js
function isAuthorizedEditor() {
    const fromField = (input.value || '').trim().toLowerCase();
    const fromStorage = (localStorage.getItem('participantName') || '').trim().toLowerCase();
    const id = fromField || fromStorage;
    return id === 'cakresa';
}

function applyEditorAccessControl() {
    const restricted = loadedFromBankSoal && !isAuthorizedEditor();
    if (restricted) {
        editorBtn.classList.add('hidden');
        if (currentlyInEditor) switchTab('test');
    } else {
        editorBtn.classList.remove('hidden');
    }
}
```

### Skenario

| Loader | User Identity | Editor visible? |
|---|---|---|
| Bank Soal | (kosong) | ✗ Hidden |
| Bank Soal | "Budi" | ✗ Hidden |
| Bank Soal | "cakresa" | ✓ Visible |
| Bank Soal | "CAKRESA" | ✓ Visible (case-insensitive) |
| File lokal | (apapun) | ✓ Visible |
| URL eksternal | (apapun) | ✓ Visible |
| Default data | (apapun) | ✓ Visible |
| Editor kosong | (apapun) | ✓ Visible |

User dengan akses lokal punya copy data sendiri, bebas edit. Editor hanya restricted untuk data master dari Bank Soal yang shared via Github repo.
