# Functions Reference

Reference 141 fungsi JavaScript di `ToeflSim.html` dikelompokkan per modul. Format: `functionName(params)` — deskripsi singkat.

## Bootstrap & Source Loader (8)

| Function | Deskripsi |
|---|---|
| `DOMContentLoaded` handler | Show init screen, fetch Bank Soal list, apply editor access |
| `loadDefaultData()` | Clone & load `defaultData` constant ke testData |
| `handleInitFileLoad(event)` | Handle file input upload, parse JSON, init app |
| `initAppWithData(rawData)` | Normalize + set testData, build shuffled map, show start screen |
| `normalizeData(rawData)` | Apply defaults, auto-migrate legacy structure, validate |
| `enterEditorEmpty()` | Load empty test template, switch ke editor tab |
| `fetchBankSoalList()` | Fetch GitHub `rezatbahtera/toeflsim` contents, render bank list |
| `refreshBankList()` | Force re-fetch bank list (bypass cache) |

## Source Tab Switching (3)

| Function | Deskripsi |
|---|---|
| `switchSourceTab(tab)` | Switch antara tab Bank/URL/Folder/Upload/Default di init screen |
| `fetchJsonFromUrl()` | Fetch JSON dari URL eksternal yang user paste |
| `fetchFolderJsons()` | Fetch listing folder (GitHub API atau Apache HTML), render |

## Directory Parser (1)

| Function | Deskripsi |
|---|---|
| `parseDirectoryListing(text, baseUrl)` | Parse Apache/nginx directory listing HTML → array files |
| `renderFolderList(files)` | Render hasil folder list dengan click handler |
| `renderBankList(files)` | Render bank soal list dengan metadata (title, section count) |

## Start & Test Setup (3)

| Function | Deskripsi |
|---|---|
| `setupStartScreen()` | Build start screen UI: title, section summary, prefill name |
| `startTest()` | Capture nama + mode, init test state, panggil `loadTarget(0,0,0)` |
| `loadTarget(sIdx, pIdx, qIdx)` | Core navigation: set current indices, trigger direction kalau perlu, render UI |

## View Switch & Navigation (5)

| Function | Deskripsi |
|---|---|
| `switchTab(tab)` | Switch antara `'test'` dan `'editor'` view |
| `showInitScreen()` | Tampilkan halaman pilih sumber data, hide test view |
| `confirmGoHome()` | Konfirmasi keluar test, force-stop semua, kembali ke init |
| `openMobileSidebar()` | Slide sidebar drawer dari kiri (mobile) |
| `closeMobileSidebar()` | Close sidebar drawer + overlay |

## Sidebar (1)

| Function | Deskripsi |
|---|---|
| `renderSidebar()` | Build sidebar navigation: section/part/group buttons dengan highlight active |

## Direction Screens (5)

| Function | Deskripsi |
|---|---|
| `showSectionDirectionScreen(sIdx)` | Tampilkan overlay direction Section (z-30), play audio kalau ada |
| `dismissSectionDirectionScreen(keepAudio)` | Stop direction audio, chain ke Part direction kalau ada |
| `showDirectionScreen(sIdx, pIdx)` | Tampilkan overlay direction Part (z-20), play audio |
| `dismissDirectionScreen(keepAudio)` | Stop direction audio, resume timers, play deferred question audio |
| `isAnyDirectionScreenActive()` | Boolean cek apakah salah satu direction screen sedang tampil |

## Question UI (8)

| Function | Deskripsi |
|---|---|
| `loadQuestionUI()` | Render passage + audio card + options untuk current question |
| `selectOption(optIdx)` | Handle radio click, save jawaban, re-render kalau review mode |
| `checkAnswer(isAutoShow)` | Reveal correct/wrong styling + reasons untuk semua options |
| `togglePracticeCheck()` | Toggle Cek Pembahasan (reveal/hide) di practice mode |
| `hidePracticeCheck()` | Reset visuals: clear correct/wrong styling, hide reasons |
| `renderPassageHtml(text)` | Render passage dengan paragraph wrapping + line gutter |
| `injectVisualLineNumbers(rootEl)` | Compute line numbers via Range API, inject markers |
| `renderInlineMarkers(text)` | Convert `~[kata]~(A)` ke `<u>kata</u>` |

## Audio Playback (6)

| Function | Deskripsi |
|---|---|
| `playQuestionAudio(qData, qKey)` | Build & start audio sequence (group → question audio) |
| `playAudioSequence(urls, callback)` | Chain audio elements via 'ended' event |
| `playDeferredAudioIfAny()` | Execute audio yang ditunda saat direction screen tampil |
| `stopAudioCompletely()` | Stop all audio (question + section + part direction) |
| `stopDirectionAudiosOnly()` | Stop only direction audios, keep question audio |
| `startGracePeriod(seconds)` | Countdown setelah audio selesai sebelum auto-advance |

## Practice Mode Controls (4)

| Function | Deskripsi |
|---|---|
| `practiceReplayAudio()` | Reset audio currentTime=0 + replay (guarded saat direction aktif) |
| `practiceToggleAudio()` | Pause/play audio question (guarded) |
| `practiceToggleTimer()` | Pause/resume semua 3 timer (guarded) |
| `updatePracticeControlsVisibility()` | Show/hide + disable/enable buttons berdasarkan mode + direction state |

## Timer (8)

| Function | Deskripsi |
|---|---|
| `startSectionTimer(sIdx)` | setInterval untuk Section timer, decrement, update display |
| `startPartTimer(sIdx, pIdx)` | setInterval untuk Part timer (kalau durationSeconds > 0) |
| `startQuestionTimer(seconds)` | setInterval untuk per-question timer |
| `handleSectionTimeout(sIdx)` | Lock section, auto-advance ke next, atau finish |
| `handlePartTimeout()` | Auto-advance ke Part berikutnya |
| `updateSectionTimerDisplay(sIdx)` | Format MM:SS, update DOM, warning color saat <2 menit |
| `updatePartTimerDisplay()` | Update Part timer display |
| `updateQuestionTimerDisplay()` | Update question timer display |

## Navigation Logic (6)

| Function | Deskripsi |
|---|---|
| `navigateQuestion(step, forcedByTimer)` | Move Prev/Next; dismiss direction kalau aktif; cross Part/Section |
| `updateNavigationState()` | Enable/disable Prev/Next buttons berdasarkan posisi & mode |
| `confirmFinishTest(forcedByTimer)` | Show finish confirm modal (kalau bukan forced) atau langsung finish |
| `proceedFinishTest()` | Dismiss modal, panggil `finishTest()` |
| `finishTest()` | Compute score, set review mode, show score modal |
| `retakeTest()` | Reset state, kembali ke start screen |
| `resetTest()` | Clear answers, timers, masuk start screen baru |

## Scoring (4)

| Function | Deskripsi |
|---|---|
| `calculateITPTotal(sectionScores)` | Average scaled scores → total ITP (310-677) |
| `rawToScaled(correct, total, type)` | Power curve raw count → scaled (31-68 or 31-67) |
| `computeResultsBreakdown()` | Build full breakdown object untuk export & display |
| `showResultsReview()` | Modal review per-section dengan collapsible details |

## Editor UI (6)

| Function | Deskripsi |
|---|---|
| `renderEditorUI()` | Build full editor: sections cards dengan group-aware pagination |
| `toggleEditorCollapse(key)` | Collapse/expand Section atau Part card di editor |
| `toggleQuestionExpand(sIdx, pIdx, qIdx)` | Expand 1 question, auto-collapse semua yang lain di Part sama |
| `setEditorQPage(sIdx, pIdx, page)` | Set current pagination page untuk Part |
| `renderPaginationControls(sIdx, pIdx, curPage, totalPages, totalQs)` | Build prev/next/numbered pagination buttons |
| `scrollToEditorEl(elId)` | Smooth scroll ke element + flash highlight |

## Editor Instruction Widget (2)

| Function | Deskripsi |
|---|---|
| `setInstructionCollapsed(key, collapsed)` | Set instruction widget collapsed state |
| `toggleInstructionCollapse(key)` | Toggle instruction widget |

## Editor Mutations (16)

| Function | Deskripsi |
|---|---|
| `addSection()` | Append new Section ke testData |
| `addPart(sIdx)` | Append new Part ke section |
| `addQuestion(sIdx, pIdx)` | Append new Question, auto-expand it |
| `addQuestionToGroup(sIdx, pIdx, groupId)` | Append Question yang sudah ber-groupId |
| `deleteSection(e, sIdx)` | Hapus Section dengan konfirmasi |
| `deletePart(e, sIdx, pIdx)` | Hapus Part dengan konfirmasi |
| `deleteQuestion(sIdx, pIdx, qIdx)` | Hapus Question dengan konfirmasi |
| `moveSection(sIdx, dir)` | Swap Section dengan atas/bawah |
| `movePart(sIdx, pIdx, dir)` | Swap Part dengan atas/bawah dalam Section |
| `movePartToSection(fromSIdx, pIdx, toSIdx)` | Pindah Part ke Section lain |
| `moveQuestion(sIdx, pIdx, qIdx, dir)` | Swap Question dengan atas/bawah dalam Part |
| `updateDataState(event, sIdx, pIdx, qIdx, field)` | Generic field updater (oninput handler) |
| `updateGroupField(groupId, sIdx, pIdx, field, value)` | Update group field by groupId |
| `promptCreateGroup(sIdx, pIdx)` | Prompt user pilih soal-soal yang dijadikan 1 group |
| `ungroupQuestions(sIdx, pIdx, groupId)` | Remove groupId dari semua question, hapus group |
| `collapseAllExceptSection(sIdx)` / `collapseAllExceptPart(partKey)` | Bulk collapse helper |

## Bulk Dump Parser (11)

| Function | Deskripsi |
|---|---|
| `openBulkDumpModal()` | Show modal dengan dropdown format + textarea |
| `openBulkDumpModalFor(sIdx, pIdx)` | Open modal pre-targeted untuk Part tertentu |
| `closeBulkDumpModal()` | Hide modal |
| `processBulkDump()` | Parse text + insert ke testData berdasarkan target |
| `previewBulkDump()` | Preview hasil parsing tanpa insert |
| `updateBulkExample()` | Update example placeholder berdasarkan format yang dipilih |
| `updateBulkStats(text)` | Hitung & tampilkan jumlah soal terdeteksi |
| `parseAuto(text)` | Auto-detect format berdasarkan pattern |
| `parseListening(text)` | Parse Part A short conversation style |
| `parseListeningLong(text)` | Parse Part B/C dengan "Questions X through Y" header |
| `parseStructure(text)` | Parse sentence completion dengan blank `_____` |
| `parseWrittenExpression(text)` | Parse underlined error format (new + legacy) |
| `parseReading(text)` | Parse passage + multiple questions |

## Audio Bank Picker (15)

| Function | Deskripsi |
|---|---|
| `configAudioBank()` | Prompt URL base catalog, simpan di localStorage |
| `openAudioPicker(inputId, sIdx, pIdx, qIdx, field)` | Show modal targeted ke input field |
| `openAudioPickerForGroup(inputId, groupId, sIdx, pIdx)` | Variant untuk group audio field |
| `closeAudioPicker()` | Hide modal |
| `audioPickerLoadCatalog()` | Fetch listing dari URL (GCS JSON, HTML, atau manifest) |
| `renderAudioPickerList(folders, files, baseUrl)` | Render list dengan filter applied |
| `renderAudioPickerBreadcrumb()` | Build breadcrumb nav untuk folder hierarchy |
| `audioPickerSelect(url)` | Fill input field dengan URL terpilih, fire input event |
| `audioPickerApplyFilter()` | Re-render list dengan filter text (real-time) |
| `audioPickerClearFilter()` | Clear filter input, re-render full list |
| `parseAudioCatalogHtml(html)` | Parse Apache/nginx HTML listing → folders + files |
| `parseGcsXmlListing(xml, currentPath)` | Parse GCS XML response (placeholder, lebih sering pakai JSON) |
| `isGcsBucketUrl(url)` | Detect URL pattern `storage.googleapis.com/BUCKET/...` |
| `toGcsListUrl(url)` | Convert public URL → GCS JSON API URL with `delimiter=/` |

## Export (7)

| Function | Deskripsi |
|---|---|
| `downloadResults()` | Export JSON breakdown (raw data) |
| `downloadResultsCsv()` | Export CSV dengan UTF-8 BOM untuk Excel |
| `downloadResultsHtml()` | Export self-contained HTML report |
| `buildHtmlReport(breakdown)` | Compose HTML string dengan inline CSS + print stylesheet |
| `toggleDownloadMenu(e)` | Show/hide dropdown menu dengan click-outside handler |
| `closeDownloadMenu()` | Hide dropdown menu |
| `exportDataToJson()` | Export testData ke JSON (editor → backup) |

## Github Save (2)

| Function | Deskripsi |
|---|---|
| `handleGithubSave()` | Open modal save ke GitHub repo via PUT API |
| `closeGithubModal()` | Hide modal |

## Access Control (2)

| Function | Deskripsi |
|---|---|
| `isAuthorizedEditor()` | Check user identity = "cakresa" (case-insensitive, field or storage) |
| `applyEditorAccessControl()` | Show/hide editor button berdasarkan source + identity |

## Helpers & Utilities (15)

| Function | Deskripsi |
|---|---|
| `escapeText(str)` | HTML escape untuk innerHTML insertion |
| `escapeAttr(str)` | Attribute escape (untuk value=, title=, dll) |
| `formatTime(seconds)` | Convert detik ke "MM:SS" |
| `shuffleArray(array)` | Fisher-Yates shuffle in-place |
| `initShuffledMap()` | Build per-question shuffled options indices |
| `newGroupId()` | Generate unique group id `g_{rnd}{ts}` |
| `getGroupForQuestion(part, q)` | Lookup Group object dari q.groupId |
| `getQuestionsInGroup(part, groupId)` | Array indices of questions with this groupId |
| `getGlobalQNumber(sIdx, pIdx, qIdx)` | Compute soal nomor global dalam Section (continues across Parts) |
| `getSectionQCount(section)` | Sum total questions di Section |
| `groupConsecutive(arr)` | Group consecutive indices ke ranges (untuk pagination) |
| `showNotification(message, type)` | Toast notification (success/error/info), auto-fade |
| `autoGrowTextarea(textarea)` | Resize textarea height to fit content |
| `initAutoGrow(rootEl)` | Apply auto-grow ke semua textarea di root |

## Score Modal Actions (1)

| Function | Deskripsi |
|---|---|
| `closeFinishConfirm()` / `closeResultsReview()` | Close modals |

---

## Function Call Graph (key paths)

### Test entry flow

```
loadDefaultData / fetchBankSoalList / fetchJsonFromUrl
    └→ initAppWithData(data)
        └→ normalizeData(data)
        └→ initShuffledMap()
        └→ setupStartScreen()
            └→ (user clicks Mulai)
                └→ startTest()
                    └→ applyEditorAccessControl()
                    └→ updatePracticeControlsVisibility()
                    └→ loadTarget(0, 0, 0)
                        ├→ showSectionDirectionScreen() [if needed]
                        ├→ showDirectionScreen() [if needed]
                        ├→ startSectionTimer()
                        ├→ startPartTimer() [if part.durationSeconds > 0]
                        └→ loadQuestionUI()
                            ├→ renderPassageHtml()
                            ├→ injectVisualLineNumbers()
                            ├→ playQuestionAudio() OR deferredAudioFn = ...
                            └→ updateNavigationState()
```

### Navigation flow

```
User clicks Next (or auto-advance)
    └→ navigateQuestion(1, forcedByTimer)
        ├→ [if direction active] → dismissDirectionScreen/SectionDirectionScreen
        ├→ [if last question of last section] → confirmFinishTest()
        │   └→ proceedFinishTest()
        │       └→ finishTest()
        │           ├→ stopAudioCompletely()
        │           ├→ computeResultsBreakdown()
        │           └→ show score modal
        └→ [else] → loadTarget(newS, newP, newQ)
```

### Export flow

```
User clicks Download → toggleDownloadMenu()
    ├→ HTML: downloadResultsHtml() → buildHtmlReport(breakdown) → Blob → download
    ├→ CSV: downloadResultsCsv() → build rows → CSV string → UTF-8 BOM Blob → download
    └→ JSON: downloadResults() → JSON.stringify(breakdown) → Blob → download
```

### Editor mutation flow

```
User clicks "Tambah Soal"
    └→ addQuestion(sIdx, pIdx)
        ├→ push new question ke testData
        ├→ editorQPage[partKey] = lastPage
        ├→ editorQExpanded[newKey] = true (auto-expand)
        ├→ collapse others
        ├→ renderEditorUI()
        └→ scrollToEditorEl(`q-card-...`)
```

### Audio Bank Picker flow

```
User clicks "📁 Pilih" → openAudioPicker(inputId, sIdx, pIdx, qIdx, field)
    └→ audioPickerLoadCatalog()
        ├→ [GCS bucket] → toGcsListUrl(url) → fetch JSON API
        ├→ [HTML] → fetch + parseAudioCatalogHtml()
        └→ [JSON manifest] → fetch + JSON.parse()
            └→ renderAudioPickerList(folders, files)
                ├→ folder click → audioPickerCurrentPath += folder/ → audioPickerLoadCatalog()
                └→ file click → audioPickerSelect(url)
                    └→ input.value = url → input event → updateDataState()
```
