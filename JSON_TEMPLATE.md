# JSON Template & Format

Spek format JSON untuk data tes TOEFL ITP yang diterima `ToeflSim.html`.

## 1. Minimal Valid Template

```json
{
  "title": "Tes TOEFL ITP Praktik 1",
  "sections": [
    {
      "title": "Section 1 Listening Comprehension",
      "type": "listening",
      "durationMinutes": 35,
      "parts": [
        {
          "title": "Part A",
          "questions": [
            {
              "question": "What does the woman mean?",
              "correctIndex": 0,
              "options": [
                { "text": "She is going home.", "reason": { "en": "Correct.", "id": "Benar." } },
                { "text": "She lost her keys.", "reason": { "en": "Wrong.", "id": "Salah." } },
                { "text": "She is at work.", "reason": { "en": "Wrong.", "id": "Salah." } },
                { "text": "She forgot the time.", "reason": { "en": "Wrong.", "id": "Salah." } }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

**Field wajib di tiap level:**

- Root: `title`, `sections` (array, min 1)
- Section: `title`, `type`, `parts` (array, min 1). `durationMinutes` opsional default 0.
- Part: `title`, `questions` (array, min 1). `groups` opsional.
- Question: `question`, `correctIndex` (0-3), `options` (array 4 element)

Field lain optional — akan di-fill default oleh `normalizeData()`.

## 2. Template Lengkap (semua field)

```json
{
  "title": "TOEFL ITP Complete Test",
  "sections": [
    {
      "title": "Section 1 Listening Comprehension",
      "type": "listening",
      "durationMinutes": 35,
      "instruction": "Section direction text — explain what user will do in this section.",
      "directionAudioUrl": "https://storage.googleapis.com/toeflsim-audio/complete-test/section-1-dir.mp3",
      "parts": [
        {
          "title": "Part A",
          "instruction": "Part A direction — short conversations.",
          "directionAudioUrl": "https://storage.googleapis.com/toeflsim-audio/complete-test/part-a-dir.mp3",
          "durationSeconds": 600,
          "passage": "",
          "audioUrl": "",
          "groups": [],
          "questions": [
            {
              "groupId": null,
              "audioUrl": "https://storage.googleapis.com/toeflsim-audio/complete-test/parta/short-1.mp3",
              "passage": "[narrator] Woman: Are you going to the meeting? Man: I wish I could, but I have a doctor's appointment. [narrator] What does the man mean?",
              "question": "What does the man mean?",
              "correctIndex": 2,
              "timeLimitSeconds": 12,
              "options": [
                { "text": "He is going to the meeting.", "reason": { "en": "He explicitly said he can't.", "id": "Dia bilang tidak bisa." } },
                { "text": "He doesn't want to attend.", "reason": { "en": "He said 'I wish I could' — he wants to but cannot.", "id": "Dia ingin hadir tapi tidak bisa." } },
                { "text": "He has another commitment.", "reason": { "en": "Doctor's appointment is another commitment.", "id": "Janji dokter = komitmen lain." } },
                { "text": "He is sick.", "reason": { "en": "Doctor's appointment ≠ being sick.", "id": "Janji dokter belum tentu sakit." } }
              ]
            }
          ]
        },
        {
          "title": "Part B",
          "instruction": "Part B — longer conversations. You will hear a conversation followed by questions.",
          "directionAudioUrl": "",
          "durationSeconds": 0,
          "groups": [
            {
              "id": "g_abc123def",
              "passage": "[narrator] Listen to a conversation between two students about an upcoming exam.\nWoman: Have you started studying for the biology exam?\nMan: Not yet. I'm still finishing the chemistry paper.\nWoman: You should at least review the cell structure chapter.",
              "audioUrl": "https://storage.googleapis.com/toeflsim-audio/complete-test/partb/conv-1.mp3"
            }
          ],
          "questions": [
            {
              "groupId": "g_abc123def",
              "question": "What are the speakers discussing?",
              "correctIndex": 0,
              "options": [
                { "text": "Upcoming exam preparation", "reason": { "en": "Correct.", "id": "Benar." } },
                { "text": "A chemistry paper", "reason": { "en": "Only mentioned in passing.", "id": "Hanya disebut sekilas." } },
                { "text": "Library hours", "reason": { "en": "Not mentioned.", "id": "Tidak disebut." } },
                { "text": "A research project", "reason": { "en": "Not mentioned.", "id": "Tidak disebut." } }
              ]
            },
            {
              "groupId": "g_abc123def",
              "question": "What does the woman recommend?",
              "correctIndex": 2,
              "options": [
                { "text": "Skipping the exam", "reason": { "en": "Wrong.", "id": "Salah." } },
                { "text": "Finishing chemistry first", "reason": { "en": "Wrong.", "id": "Salah." } },
                { "text": "Reviewing the cell structure chapter", "reason": { "en": "Correct.", "id": "Benar." } },
                { "text": "Forming a study group", "reason": { "en": "Wrong.", "id": "Salah." } }
              ]
            }
          ]
        }
      ]
    },
    {
      "title": "Section 2 Structure and Written Expression",
      "type": "structure",
      "durationMinutes": 25,
      "instruction": "Section 2 direction.",
      "parts": [
        {
          "title": "Structure",
          "instruction": "Choose the answer that best completes the sentence.",
          "questions": [
            {
              "passage": "",
              "question": "Light _____ at a speed of about 300,000 kilometers per second.",
              "correctIndex": 1,
              "options": [
                { "text": "travels", "reason": { "en": "Wrong tense.", "id": "Tense salah." } },
                { "text": "travels", "reason": { "en": "Simple present for scientific fact.", "id": "Simple present untuk fakta ilmiah." } },
                { "text": "traveling", "reason": { "en": "Needs finite verb.", "id": "Butuh kata kerja finit." } },
                { "text": "has traveled", "reason": { "en": "Perfect not appropriate.", "id": "Perfect tidak tepat." } }
              ]
            }
          ]
        },
        {
          "title": "Written Expression",
          "instruction": "Identify the ONE underlined part that should be changed.",
          "questions": [
            {
              "question": "She ~[have]~(A) ~[gone]~(B) to the ~[market]~(C) ~[yesterday]~(D).",
              "correctIndex": 0,
              "options": [
                { "text": "A", "reason": { "en": "Should be 'has' or just 'went'.", "id": "Harusnya 'has' atau 'went'." } },
                { "text": "B", "reason": { "en": "Correct usage.", "id": "Benar." } },
                { "text": "C", "reason": { "en": "Correct usage.", "id": "Benar." } },
                { "text": "D", "reason": { "en": "Correct usage.", "id": "Benar." } }
              ]
            }
          ]
        }
      ]
    },
    {
      "title": "Section 3 Reading Comprehension",
      "type": "reading",
      "durationMinutes": 55,
      "instruction": "Section 3 direction.",
      "parts": [
        {
          "title": "Reading Set 1",
          "groups": [
            {
              "id": "g_read1",
              "passage": "Photosynthesis is the process by which green plants convert sunlight into chemical energy. The process occurs primarily in chloroplasts, which contain chlorophyll — the pigment that gives plants their green color.\n\nDuring photosynthesis, plants absorb carbon dioxide from the air and water from the soil. Using energy from sunlight, they transform these into glucose and oxygen."
            }
          ],
          "questions": [
            {
              "groupId": "g_read1",
              "question": "What is the main topic of the passage?",
              "correctIndex": 1,
              "options": [
                { "text": "Plant biology in general", "reason": { "en": "Too broad.", "id": "Terlalu umum." } },
                { "text": "The process of photosynthesis", "reason": { "en": "Correct.", "id": "Benar." } },
                { "text": "Chloroplasts", "reason": { "en": "Only mentioned briefly.", "id": "Hanya disebut sekilas." } },
                { "text": "The color of plants", "reason": { "en": "Side detail.", "id": "Detail samping." } }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## 3. Field Reference

### Root

| Field | Type | Wajib | Default | Notes |
|---|---|---|---|---|
| `title` | string | ✓ | "Tes Tanpa Judul" | Tampil di header, score modal, filename export |
| `sections` | array | ✓ | `[]` | Min 1 element |

### Section

| Field | Type | Wajib | Default | Notes |
|---|---|---|---|---|
| `title` | string | ✓ | "Section N" | Tampil di sidebar + score |
| `type` | string | ✓ | "listening" | `'listening'`/`'structure'`/`'reading'` — menentukan formula scoring |
| `durationMinutes` | number | ✗ | 0 | Section timer total (0 = unlimited) |
| `instruction` | string | ✗ | "" | Trigger Section direction screen kalau non-empty |
| `directionAudioUrl` | string | ✗ | "" | Audio direction Section (boleh ada tanpa text instruction) |
| `parts` | array | ✓ | `[]` | Min 1 element |

### Part

| Field | Type | Wajib | Default | Notes |
|---|---|---|---|---|
| `title` | string | ✓ | "Part N" | Tampil di sidebar |
| `instruction` | string | ✗ | "" | Trigger Part direction screen kalau non-empty |
| `directionAudioUrl` | string | ✗ | "" | Audio direction Part |
| `durationSeconds` | number | ✗ | 0 | Part timer (0 = no part timer) |
| `passage` | string | ✗ | "" | Legacy: passage shared kalau no groups |
| `audioUrl` | string | ✗ | "" | Legacy: audio shared kalau no groups (auto-migrated ke group implicit) |
| `groups` | array | ✗ | `[]` | Definisi shared passage/audio per group |
| `questions` | array | ✓ | `[]` | Min 1 element |

### Group

| Field | Type | Wajib | Default | Notes |
|---|---|---|---|---|
| `id` | string | ✓ | auto-generated `g_xxx` | FK target dari Question.groupId |
| `passage` | string | ✗ | "" | Shared text untuk multiple questions |
| `audioUrl` | string | ✗ | "" | Shared audio yang dimainkan 1x untuk semua Q di group |

### Question

| Field | Type | Wajib | Default | Notes |
|---|---|---|---|---|
| `groupId` | string\|null | ✗ | null | FK ke Group.id |
| `audioUrl` | string | ✗ | "" | Audio per-soal (selain group audio) |
| `passage` | string | ✗ | "" | Passage per-soal (kalau tidak pakai group) |
| `question` | string | ✓ | "" | Teks pertanyaan, support marker `~[kata]~(A)` |
| `correctIndex` | 0\|1\|2\|3 | ✓ | 0 | Index jawaban benar di array options |
| `timeLimitSeconds` | number | ✗ | 0 | Per-soal timer (0 = no limit, pakai section/part) |
| `options` | array(4) | ✓ | 4 element kosong | EXACTLY 4 options |

### Option

| Field | Type | Wajib | Default | Notes |
|---|---|---|---|---|
| `text` | string | ✓ | "" | Teks pilihan ganda |
| `reason` | object | ✗ | `{en:"",id:""}` | Penjelasan kenapa benar/salah |
| `reason.en` | string | ✗ | "" | English explanation |
| `reason.id` | string | ✗ | "" | Indonesian explanation |

## 4. Bulk Dump Parser

Selain JSON, editor menerima paste **text dump** dari soal mentah (mis. copy dari buku Longman). Parser otomatis ekstrak structure.

### Format yang didukung

#### Listening Short Conversation (Part A)

```
1. (woman) Are you going to the meeting?
   (man) I wish I could, but I have a doctor's appointment.
   (narrator) What does the man mean?
   (A) He is going to the meeting.
   (B) He doesn't want to attend.
   (C) He has another commitment.
   (D) He is sick.
   ANSWER: C
```

#### Listening Long Conversation (Part B/C with groups)

```
Questions 1 through 3 refer to the following conversation.
[narrator] Listen to a conversation between two students.
Woman: Have you started studying?
Man: Not yet. I'm finishing chemistry first.
Woman: Review cell structure at least.

1. What are they discussing?
   (A) Upcoming exam preparation
   (B) A chemistry paper
   ...
   ANSWER: A

2. What does the woman recommend?
   ...
```

Parser detect `Questions X through Y` header, group semua soal ke dalam 1 group dengan shared passage.

#### Structure (Sentence Completion)

```
1. Light _____ at a speed of about 300,000 kilometers per second.
   (A) travel
   (B) travels
   (C) traveling
   (D) has traveled
   ANSWER: B
```

Atau format dengan blank di depan + colon:

```
1. _____: the experiment proved the theory.
   (A) Although
   (B) Because of
   (C) Despite
   (D) Due to
   ANSWER: B
```

#### Written Expression (Underlined Errors)

Format **baru** dengan inline markers:

```
1. She ~[have]~(A) ~[gone]~(B) to the ~[market]~(C) ~[yesterday]~(D).
   ANSWER: A
```

Format **legacy** (auto-converted):

```
1. She (A)have gone to the (B)market yesterday for (C)buying (D)groceries.
   ANSWER: A
```

Parser convert legacy `(X)word` → `~[word]~(X)` saat import.

#### Reading

```
PASSAGE
Photosynthesis is the process by which green plants...

1. What is the main topic?
   (A) Plant biology
   (B) The process of photosynthesis
   ...
   ANSWER: B

2. According to the passage, where does photosynthesis occur?
   ...
```

`PASSAGE` header + multiple questions yang share passage.

### Direction Detection

Parser detect direction text yang diawali dengan:

- `Section X Directions:`
- `Section X Direction:`
- `Part X Directions:`
- `Part X Direction:`
- `Directions:` (di awal)

Text setelah marker di-assign ke `section.instruction` atau `part.instruction` sesuai konteks.

### Parser Functions

| Function | Trigger | Output |
|---|---|---|
| `parseAuto(text)` | Auto-detect berdasarkan pattern | Mix sections/parts |
| `parseListening(text)` | Part A style | Array questions tanpa group |
| `parseListeningLong(text)` | Part B/C style | Array groups + questions dengan groupId |
| `parseStructure(text)` | Structure (sentence completion) | Array questions |
| `parseWrittenExpression(text)` | Written Expression | Array questions dengan marker |
| `parseReading(text)` | Reading | Array groups + questions |

## 5. Validation

Saat `initAppWithData(rawData)` dipanggil:

1. Cek `rawData.sections` array dan non-empty
2. `normalizeData(rawData)` set semua default + auto-migrate legacy
3. Cek tiap question punya 4 options
4. `correctIndex` clamp ke 0-3
5. Generate `id` untuk group yang tidak punya
6. Build `shuffledMap` untuk mode ujian

Jika validasi gagal, alert error muncul dan data tidak di-load.
