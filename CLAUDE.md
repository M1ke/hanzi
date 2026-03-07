# CLAUDE.md — AI Assistant Guide for Hanzi Flashcards

## Project Overview

**Hanzi Flashcards** is a minimal, offline-capable single-page web application for practising the most common Chinese characters using spaced repetition flashcards. Users enter a 3-letter seed to deterministically select 20 characters from the Jun Da frequency corpus.

- **Live site**: https://m1ke.github.io/hanzi
- **License**: GNU GPL v3
- **Stack**: Pure HTML/CSS/JS — no build step, no backend, no package manager

---

## Repository Structure

```
hanzi/
├── index.html        # Entire application: markup + Alpine.js logic
├── data.js           # 819 Chinese characters with pinyin ({c, p} format)
├── README.md         # User-facing documentation
├── CONTRIBUTING.md   # Developer/architecture documentation
├── CLAUDE.md         # This file
├── LICENSE           # GNU GPL v3
├── .nojekyll         # Prevents GitHub Pages from running Jekyll
└── .gitignore        # Ignores .idea/ and screenshots/
```

---

## Running Locally

There is no build step. Serve the static files directly:

```sh
python3 -m http.server 8765
# then open http://localhost:8765
```

Or use any static file server (`npx serve .`, VS Code Live Server, etc.).

The app also works opened directly as a `file://` URL in a browser.

---

## No Build, No Tests, No Dependencies

- **No package.json** — no npm/yarn/pnpm
- **No bundler** — no webpack, vite, rollup, esbuild
- **No TypeScript** — plain JavaScript inside `index.html`
- **No test suite** — testing is manual via the browser
- **No CI/CD pipeline** — GitHub Pages deploys automatically when `main` is pushed
- **No environment variables** — fully self-contained

CDN dependencies loaded in `index.html`:
- **Alpine.js v3** (reactivity/state) — `unpkg.com/alpinejs@3`
- **Tailwind CSS** (styling) — `cdn.tailwindcss.com`

---

## Architecture

### All Logic Lives in `index.html`

The application is split into two logical sections inside `index.html`:

1. **HTML (lines ~1–222)**: Semantic markup with Alpine.js directives (`x-data`, `x-show`, `x-model`, `@click`, etc.)
2. **JavaScript (lines ~223–339)**: The `app()` function and utility functions, defined in a `<script>` tag

### State Machine (screens)

Navigation is driven by a `screen` property in Alpine.js state:

```
'home' → (optional) 'preview' → 'card' → 'result' → 'card' → ... → 'summary' → 'home'
```

| Screen | Purpose |
|--------|---------|
| `'home'` | Seed input, session history |
| `'preview'` | Grid of 20 characters before starting |
| `'card'` | One character shown; user types pinyin |
| `'result'` | Shows correct/incorrect feedback |
| `'summary'` | Final score (X/20) with per-card breakdown |

### Key State Properties

Defined inside `app()` in `index.html`:

| Property | Type | Purpose |
|----------|------|---------|
| `screen` | string | Current UI screen |
| `seedInput` | string | Bound to the seed text input (max 3 chars) |
| `currentSeed` | string | Active seed used for the session |
| `cards` | array | The 20 selected `{c, p}` objects |
| `cardIndex` | number | Which card is currently shown (0–19) |
| `userInput` | string | User's current pinyin attempt |
| `results` | array | Per-card `{char, correct, expected, got}` |
| `history` | array | Loaded from `localStorage`; up to 50 sessions |

### Key Functions

| Function | Location | Purpose |
|----------|----------|---------|
| `seedFromWord(word)` | index.html `<script>` | Hashes a 3-char string to a uint32 |
| `mulberry32(seed)` | index.html `<script>` | Mulberry32 PRNG; returns a seeded random function |
| `pickCards(word)` | index.html `<script>` | Partial Fisher-Yates shuffle; returns 20 characters |
| `stripTones(s)` | index.html `<script>` | Strips Unicode combining diacritics from pinyin |

---

## Data Format (`data.js`)

```js
const HANZI = [
  {c: "的", p: "de"},
  {c: "一", p: "yī"},
  // ... 819 entries
];
```

- `c` — single Chinese character
- `p` — toned pinyin using Unicode combining diacritics (e.g., `"zhōng"`)
- Ordered by Jun Da frequency (most common first)
- **Do not reorder** — the seeding algorithm relies on stable array indices

---

## Pinyin Matching Rules

Matching is lenient to allow easy typing without a toned input method:

1. Both sides lowercased
2. Both sides trimmed of whitespace
3. `v` in user input is converted to `u` (e.g., `lv` matches `lǜ`)
4. Tone marks stripped from the stored pinyin before comparison

**Example**: Character `中` stores `"zhōng"` → stripped to `"zhong"` → matches user input `"zhong"` or `"ZHONG"`.

---

## Session History

Stored in `localStorage` under key `'hanzi_history'`:

```js
[
  { seed: "ABC", score: 18, total: 20, date: "3/7/2026" },
  // ...
]
```

- Max 50 entries; oldest are discarded when limit is reached
- Score colour coding: green ≥80%, yellow ≥50%, red <50%

---

## Keyboard Shortcuts

| Key | Screen | Action |
|-----|--------|--------|
| Enter | `card` | Submit pinyin answer |
| Enter | `result` | Advance to next card (when Next button is focused) |
| Escape | `card` | Skip the current card without answering |

---

## Deployment

GitHub Pages serves the `main` branch directly. To deploy:

```sh
git push origin main
```

The `.nojekyll` file ensures GitHub Pages serves raw static files without running Jekyll. No build artefacts need to be generated.

---

## Conventions to Follow

### When Editing `index.html`

- Keep all JavaScript inside the single `<script>` block near the bottom of the file
- Use Alpine.js directives for reactivity — do not manipulate the DOM directly with `document.querySelector` etc.
- All CSS is Tailwind utility classes — do not add a `<style>` block or external CSS unless absolutely necessary
- Preserve the `x-data="app()"` binding on the root container

### When Editing `data.js`

- Maintain the `{c, p}` object shape exactly
- Pinyin values must use Unicode combining diacritics for tone marks (not numbered tones like `zhong1`)
- Do not reorder entries — frequency order matters for the seeding algorithm
- The `HANZI` array must remain a `const` declared at the top level (no wrapping function or module)

### General

- Keep the project dependency-free (no npm packages, no bundler)
- Do not introduce a build step — all files must be servable as-is
- Do not add a backend — this is intentionally a client-side-only app
- Avoid adding new files unless strictly necessary; prefer editing existing ones
- Match the existing code style: 2-space indentation, no semicolons in Alpine template expressions

---

## Common Tasks

### Add a New Character

Append an entry to the `HANZI` array in `data.js`:

```js
{c: "新", p: "xīn"},
```

Ensure the pinyin uses proper Unicode tone marks.

### Change the Number of Cards per Session

The number `20` is referenced in `pickCards()` (returns first N elements after partial shuffle) and in the summary screen template. Update both locations.

### Add a New Screen

1. Add a new `x-show="screen === 'newscreen'"` section in the HTML
2. Add navigation logic in `app()` (set `this.screen = 'newscreen'`)
3. Follow the same `transition` and layout patterns as existing screens

### Modify Pinyin Matching

The `stripTones()` function and the `v`→`u` substitution are both in the `checkAnswer()` method inside `app()`. Adjust there.
