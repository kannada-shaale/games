# AGENTS.md — Kannada Learning Games Hub

This file provides architectural context, conventions, and guardrails for AI coding agents working in this repository.

---

## 1. Project Overview
This repository hosts a static, community-driven **Kannada Learning Games Hub** deployed via **GitHub Pages**.
* **Zero-Build Static Architecture:** No bundler (Webpack/Vite), no Node.js runtime dependency for the frontend, and no backend server.
* **Single-File Games:** Every game lives as a standalone, self-contained `.html` file inside `games/`.
* **Automated Directory Indexing:** A GitHub Action (`.github/workflows/build-index.yml`) parses `<meta name="game:*">` tags from `games/*.html` using Python's built-in `html.parser` and generates `games-index.json` at the repository root.
* **Central Hub (`index.html`):** Fetches `games-index.json` client-side, groups games into 6 fixed learning categories, and provides search and filtering by skill level and target age.

---

## 2. Repository Structure
```text
├── index.html                        # Central game directory hub (fetches games-index.json)
├── games-index.json                  # Auto-generated manifest (DO NOT manually edit)
├── CONTRIBUTING.md                   # Contributor guidelines and HTML metadata template
├── AGENTS.md                         # AI assistant instructions and repo conventions
├── games/                            # All standalone HTML games live here
│   └── *.html                        # Single-file games (HTML + inline CSS + inline JS)
└── .github/
    └── workflows/
        └── build-index.yml           # Validates <meta> tags and rebuilds games-index.json
```

---

## 3. Core Architecture & Constraints

### A. Single-File Mandate for Games
When creating or modifying any game in `games/*.html`:
1. **Self-Contained:** All HTML, `<style>`, and `<script>` code **must** reside in a single `.html` file. Do **not** create separate `.css`, `.js`, or local image/audio asset files.
2. **External Dependencies:** Keep external calls minimal and reliable. Approved external resources:
   * Google Fonts (`Noto Sans Kannada` and `Plus Jakarta Sans`)
   * Browser-native APIs (Web Speech API `speechSynthesis` / `SpeechRecognition`, Web Audio API `AudioContext`)
   * Unicode emojis or inline SVGs for visuals (ensures zero broken image links).
3. **Navigation:** Include a visible link back to the main directory (`href="../index.html"`).
4. **File Naming:** Use lowercase kebab-case in `games/` (e.g., `games/swara-fill-blanks.html`, `games/color-match.html`).

### B. Required `<meta>` Schema (Strict Validation)
Every `.html` file in `games/` **must** include `<title>` and `<meta name="game:*">` tags inside `<head>`. The CI workflow (`build-index.yml`) will **fail** if `game:category` is missing or invalid.

```html
<!DOCTYPE html>
<html lang="kn">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ಕನ್ನಡ ಶೀರ್ಷಿಕೆ · English Game Title</title>

  <!-- Required Directory Metadata -->
  <meta name="game:category" content="Aksharamale (Alphabet & Script)">
  <meta name="game:level" content="Beginner">
  <meta name="game:age" content="4-7, 8-12">
  <meta name="game:tags" content="Swaragalu, Puzzle, Audio">
  <meta name="game:description" content="Concise one-sentence summary of what learners practice.">
  <meta name="game:author" content="Author Name">
</head>
```

#### Allowed `game:category` Values (Exact String Match Required)
1. `Aksharamale (Alphabet & Script)` — Vowels (*Swaragalu*), consonants (*Vyanjanagalu*), vowel signs (*Gunitakshara*), conjuncts (*Ottakshara*).
2. `Vocabulary & Themes` — Colors (*Bannagalu*), animals (*Pranigalu*), family (*Kutumba*), numbers, food, nature.
3. `Reading & Word Building` — Spelling puzzles, fill-in-the-blank words, syllable scrambles, early reading.
4. `Listening & Speaking` — Audio-first games for pre-readers, command/action games, pronunciation/microphone challenges.
5. `Grammar & Sentences` — Pronouns (*Idu*/*Adu*), directions (*Mele*/*Kelage*), tenses, verb conjugations, sentence order.
6. `Culture, Rhymes & Stories` — Folk tales (*Janapada*), proverbs (*Gadegalu*), riddles (*Ogatugalu*), children's songs (*Shishu Geethegalu*).

#### Allowed `game:level` Values
* `Beginner` (Oral/picture-first, no script reading required)
* `Early Reader` (Basic akshara recognition & simple 2–3 letter words)
* `Intermediate` (Gunitakshara, Ottakshara, simple sentences)
* `Fluent` (Complex grammar, stories, idioms)

#### Standard `game:age` Values (Comma-Separated)
* `4-7`, `8-12`, `13+`

---

## 4. Kannada Linguistic & UX Guidelines for Game Development
When generating educational Kannada content or game mechanics, follow these domain rules:

1. **Syllabic Segmentation (Akshara Integrity):**
   * Kannada is an abugida. Never split a Kannada string using naive `word.split('')` when creating letter tiles or blanks—doing so separates dependent vowel signs (*kagunita* / *matras* like `ಾ`, `ಿ`, `ೀ`, `ು`, `ೂ`, `ೆ`, `ೇ`, `ೈ`, `ೊ`, `ೋ`, `ೌ`, `ಂ`, `ಃ`) and virama/halant (`್`) from their base consonants.
   * Always define words as explicit arrays of complete akshara units (e.g., `["ಆ", "ನೆ"]`, `["ಈ", "ರು", "ಳ್ಳಿ"]`, `["ಸೂ", "ರ್ಯ"]`).
2. **Typography & Readability:**
   * Always include `Noto Sans Kannada` with generous font sizes (`>= 1.5rem` for interactive tiles) and comfortable line heights so conjuncts (*ottakshara*) and vowel modifiers are clearly legible for young learners.
3. **Audio & Speech Fallbacks:**
   * When using `window.speechSynthesis` with `utter.lang = 'kn-IN'`, always wrap calls safely in feature detection (`'speechSynthesis' in window`) because some browsers/OSes lack a built-in Kannada TTS voice.
   * When using `SpeechRecognition` / `webkitSpeechRecognition` (`lang = 'kn-IN'`), always provide a manual fallback button (e.g., *"I said it!"* or parent verification) so learners on unsupported browsers (such as Firefox or iOS Safari) are never soft-locked.
4. **Target Audience & Age Calibration:**
   * **Ages 4–7 (`Beginner`):** Rely on emojis/illustrations, audio prompts, large touch targets, and immediate positive reinforcement rather than English or Kannada text instructions alone.
   * Include bilingual labels (Kannada script + English meaning/transliteration, toggleable when appropriate) so diaspora learners and parents can follow along.

---

## 5. CI/CD & Indexing Rules
* **Do Not Edit `games-index.json` Manually:** It is automatically regenerated on pushes to `main` whenever files in `games/*.html` change.
* **Zero External Python Dependencies:** Keep `.github/workflows/build-index.yml` reliant only on Python's standard library (`html.parser`, `json`, `glob`, `os`, `sys`) so CI runs fast without `pip install` steps.
* **Local Verification:** Before finishing changes to any game or `index.html`, verify that:
  1. All `<meta name="game:*">` tags are present and `game:category` matches one of the 6 exact strings in Section 3B.
  2. No external relative assets (`.css`, `.js`, `.png`, `.mp3`) are referenced.
  3. All Kannada words and akshara arrays are orthographically accurate.
