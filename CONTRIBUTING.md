# Contributing a Kannada Learning Game

Thank you for contributing! All games are **single-file, self-contained `.html` files** placed inside the `games/` directory.

## Rules
1. Place your file in `games/your-game-name.html` (use lowercase letters and hyphens).
2. Keep all CSS and JavaScript inside the single `.html` file so there are no broken links.
3. Include the required `<meta name="game:*">` tags in your `<head>` so the automated index lists your game properly.

## Required Metadata Template

Copy and paste this into your `<head>`:

```html
<title>Your Game Title (ಕನ್ನಡ ಶೀರ್ಷಿಕೆ)</title>
<meta name="game:category" content="Aksharamale (Alphabet & Script)">
<meta name="game:level" content="Beginner">
<meta name="game:age" content="4-7, 8-12">
<meta name="game:tags" content="Swaragalu, Puzzle, Audio">
<meta name="game:description" content="One-sentence summary of what learners practice.">
<meta name="game:author" content="Your Name or GitHub Handle">
