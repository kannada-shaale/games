# AGENTS.md — AI Coding Assistant Guide for Kannada Learning Games

This repository powers a static GitHub Pages hub (`index.html`) of community-contributed, single-file HTML games for learning Kannada. Automated CI parses `<meta name="game:*">` tags from `games/*.html` to build `games-index.json`.

---

## 1. Repository Architecture & File Layout

```text
kannada-games/
├── index.html                        # Static hub page (fetches games-index.json, renders categories & filters)
├── games-index.json                  # Auto-generated manifest (DO NOT edit manually)
├── AGENTS.md                         # AI assistant development guidelines
├── CONTRIBUTING.md                   # Contributor guide & HTML metadata template
├── audio/                            # Pre-generated neural Kannada MP3s (kn-IN-SapnaNeural)
│   ├── amma.mp3
│   ├── praise_shahabbas.mp3
│   ├── try_again.mp3
│   └── ...
├── scripts/
│   └── generate_audio.py             # edge-tts script to batch-generate audio/*.mp3 clips
├── games/
│   ├── swara-fill.html               # Standalone single-file HTML games
│   ├── akshara-practice.html
│   ├── gunitakshara-blender.html
│   ├── action-simon-says.html
│   └── animal-safari.html
└── .github/
    └── workflows/
        └── build-index.yaml          # Validates metadata & rebuilds games-index.json on push/PR

```
Core Architectural Rules
Single-File Self-Contained Games: Every game in games/<slug>.html MUST contain all its HTML, CSS (<style>), and JavaScript (<script>) inline in one file. Do not create separate .css or .js bundles per game.

Permitted External Assets:

Google Fonts (Noto Sans Kannada and Plus Jakarta Sans) via https://fonts.googleapis.com.

Local pre-generated audio clips from ../audio/<key>.mp3 (with a Web Speech API fallback).

Never Edit games-index.json Manually: build-index.yaml overwrites games-index.json on every push to main touching games/*.html. Always update <meta> tags inside the HTML file instead.

Clean HTML Boundaries: Ensure every games/*.html file starts on line 1 with <!DOCTYPE html> and ends with </html> (no stray Markdown code fences or prose outside HTML tags), and keep all <meta>, <link>, and <style> tags properly inside <head>.

2. Required HTML <meta> Schema (Enforced by CI)
Every file in games/*.html is parsed by Python's built-in html.parser in .github/workflows/build-index.yaml. If game:category is missing or invalid, CI exits with status 1 and blocks the build.

Include this exact block inside <head>:

HTML
<!DOCTYPE html>
<html lang="kn">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ಕನ್ನಡ ಶೀರ್ಷಿಕೆ · English Game Title</title>

  <!-- Directory Metadata -->
  <meta name="game:category" content="Aksharamale (Alphabet & Script)">
  <meta name="game:level" content="Beginner">
  <meta name="game:age" content="4-7, 8-12">
  <meta name="game:tags" content="Vocabulary, Audio, Quiz">
  <meta name="game:description" content="Concise one-sentence summary of what learners practice.">
  <meta name="game:author" content="Author or GitHub Handle">
</head>
Allowed game:category Values (Exact String Match Required)
Aksharamale (Alphabet & Script)

Vocabulary & Themes

Reading & Word Building

Listening & Speaking

Grammar & Sentences

Culture, Rhymes & Stories

Standard game:level Values
Beginner (default if omitted; ideal for ages 4–7 / oral & picture-first games)

Early Reader

Intermediate

Fluent

3. Kannada Audio Workflow (audio/ + Web Speech Fallback)
Browser Web Speech API (speechSynthesis) often sounds robotic or shaky on devices lacking a local kn-IN voice, or when utter.rate is slowed below 0.90. All new games with spoken Kannada prompts MUST use the MP3-first + Neural Voice Fallback pattern.

A. Reusable Clips Already in audio/
Before generating new audio files, check if your game can reuse existing clips in audio/:

Praise / Feedback: praise_shahabbas.mp3 (ಶಹಬ್ಬಾಸ್! ಸರಿ ಉತ್ತರ!), try_again.mp3 (ಮತ್ತೊಮ್ಮೆ ಪ್ರಯತ್ನಿಸಿ)

Swara Starter Words: amma.mp3, aane.mp3, ili.mp3, eerulli.mp3, ungura.mp3, oota.mp3, rushi.mp3, ele.mp3, eni.mp3, aidu.mp3, onte.mp3, odu.mp3, aushadha.mp3, angadi.mp3, aha.mp3

Actions: act_baa.mp3, act_kulithuko.mp3, act_nillu.mp3, act_odu.mp3, act_noodu.mp3, act_chappale.mp3, act_kai_etthu.mp3, act_nagu.mp3

Animals: ani_naayi.mp3, ani_bekku.mp3, ani_hasu.mp3, ani_aane.mp3, ani_kothi.mp3, ani_simha.mp3, ani_kappe.mp3, ani_gili.mp3

B. Generating New MP3 Clips (edge-tts)
When creating a new game with new Kannada words or sentences:

Add the new <audio_key>: "<Kannada text>" entries to scripts/generate_audio.py (using lowercase snake_case keys, e.g., col_kempu.mp3, num_ondu.mp3).

Generate the MP3 files using Microsoft's neural Kannada voice (kn-IN-SapnaNeural, rate="-5%"):

Bash
pip install edge-tts
python3 scripts/generate_audio.py
Commit the new audio/*.mp3 files alongside games/<slug>.html (or remind the user to run scripts/generate_audio.py / the GitHub Action workflow before pushing).

C. Required JavaScript Audio Helper
Use this pattern in games/*.html so games play ../audio/<key>.mp3 when available and gracefully fall back to the best available kn-IN browser voice:

JavaScript
let currentAudio = null;
let bestKannadaVoice = null;

function loadBestVoice() {
  if (!('speechSynthesis' in window)) return;
  const voices = window.speechSynthesis.getVoices();
  const knVoices = voices.filter(v => v.lang.toLowerCase().includes('kn'));
  bestKannadaVoice =
    knVoices.find(v => /natural|online|google/i.test(v.name)) ||
    knVoices[0] ||
    null;
}

if ('speechSynthesis' in window) {
  loadBestVoice();
  window.speechSynthesis.onvoiceschanged = loadBestVoice;
}

function speakFallback(text) {
  if (!('speechSynthesis' in window)) return;
  window.speechSynthesis.cancel();
  const utter = new SpeechSynthesisUtterance(text);
  utter.lang = 'kn-IN';
  if (bestKannadaVoice) utter.voice = bestKannadaVoice;
  utter.rate = 0.95;
  utter.pitch = 1.05;
  window.speechSynthesis.speak(utter);
}

function playKannada(audioKey, fallbackText) {
  if (currentAudio) {
    currentAudio.pause();
    currentAudio.currentTime = 0;
  }
  const audio = new Audio(`../audio/${audioKey}.mp3`);
  currentAudio = audio;
  audio.play().catch(() => speakFallback(fallbackText));
}
4. Kannada Domain & Pedagogical Guidelines
Preserve Akshara (Syllable) Boundaries:

Never split Kannada strings with .split('') when creating letter tiles or blanks—this breaks Unicode consonant + vowel sign (gunitakshara) and conjunct (ottakshara) clusters (e.g., ಕಿ, ಮ್ಮ, ರ್ಯ).

Always define words explicitly as arrays of syllabic units (e.g., ["ಕ", "ಮ", "ಲ"], ["ಆ", "ನೆ"], ["ಹೂ", "ವು"]) or Intl.Segmenter('kn', { granularity: 'grapheme' }).

Typography & Readability:

Load Noto Sans Kannada (600, 700, 800) and render interactive Kannada glyphs at >= 1.8rem with generous line-height (>= 1.3) so bottom vowel signs (ು, ೂ, ೃ) and subscripts (ottaksharas) never clip.

Age-Appropriate UX (Ages 4–7 vs. 8+):

For Beginner / 4-7 games, pair Kannada script with emojis/illustrations, English/phonetic cues, and spoken audio so pre-readers can play independently.

Include a <a href="../index.html">← All Games</a> navigation link in the top bar of every game.

If using SpeechRecognition (kn-IN) for speaking games, always include a manual/parent-check button (✅ Child said it!) for browsers that do not support Kannada speech recognition.

5. Pre-Commit Checklist for New Games
When generating or reviewing a new game in games/<slug>.html, always verify and remind the user:

[ ] File is saved in games/<lowercase-hyphenated-name>.html with no stray Markdown fences.

[ ] <meta name="game:category"> matches one of the 6 exact allowed categories.

[ ] All <meta>, <link>, and <style> tags are inside <head>.

[ ] Audio uses playKannada(audioKey, fallbackText) with rate = 0.95 fallback.

[ ] Audio Reminder: New word/phrase MP3 keys are added to scripts/generate_audio.py and generated into audio/*.mp3 before or immediately after pushing.
