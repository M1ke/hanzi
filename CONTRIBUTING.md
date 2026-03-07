# Contributing

## Architecture

No build step, no bundler, no framework CLI. The entire app is four files:

| File | Role |
|------|------|
| `index.html` | All markup and application logic |
| `data.js` | Character dataset — a single `const HANZI` array |
| `.nojekyll` | Tells GitHub Pages not to run Jekyll |
| `README.md` | This repo's documentation |

**Alpine.js** (loaded from CDN) handles reactive state and DOM updates. **Tailwind CSS** (loaded from CDN) handles styling. Both are referenced via `<script>` tags in `index.html` — no local copies, no compilation.

## Application state

All state lives in the `app()` function returned to Alpine's `x-data`. The `screen` property drives which panel is visible:

```
'home' → 'preview' → 'home'
'home' → 'card' → 'result' → 'card' → ... → 'summary' → 'home'
```

Key state properties:

| Property | Purpose |
|----------|---------|
| `screen` | Which panel is shown |
| `seedInput` | Raw text from the home input |
| `currentSeed` | Uppercased seed in use for the current session |
| `cards` | Array of 20 `{c, p}` objects for this session |
| `cardIndex` | Index into `cards` for the current card |
| `results` | Array of `{card, correct}` built up during a session |
| `history` | Sessions loaded from / saved to localStorage |

## Seeded card selection

`pickCards(word)` uses a **mulberry32 PRNG** seeded from the three-letter word. It runs a partial Fisher-Yates shuffle over the full `HANZI` array and picks the first 20 results. The same seed always produces the same 20 characters in the same order.

```js
function seedFromWord(word) { ... }   // hashes 3 chars to uint32
function mulberry32(seed) { ... }      // returns a () => float [0,1) function
function pickCards(word) { ... }       // partial Fisher-Yates, returns 20 entries
```

## Pinyin matching

Characters in `data.js` store toned pinyin (e.g. `"zhōng"`). User input is matched after stripping tones:

```js
function stripTones(s) {
  return s.normalize('NFD').replace(/[\u0300-\u036f]/g, '');
}
// user input: toLowerCase().trim().replace(/v/g, 'u')
// card pinyin: stripTones(card.p)
```

The `v → u` substitution means typing `lv` matches `lǜ`, which is the standard Pinyin IME convention.

## Adding or editing characters

Edit `data.js`. Each entry is `{ c: "字", p: "toned-pinyin" }`:

- `c` — the Chinese character (single character only)
- `p` — pinyin **with** tone marks for display (e.g. `"zhōng"`, `"nǐ"`, `"lǜ"`)
- Tone marks use Unicode combining diacritics — standard UTF-8, paste directly
- No duplicate characters; each `c` value must be unique in the array

The dataset currently contains ~820 characters drawn from the Jun Da frequency corpus. To add more, append entries to the `HANZI` array. The order affects which characters get picked for lower-frequency seeds, but since selection is random within the array, ordering is not critical.

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| Enter | Submit answer (card screen) / advance (result screen, Next button focused) |
| Esc | Skip the current card |

## Deploying to GitHub Pages

Push to `main`. In repository Settings → Pages, set source to branch `main`, folder `/`. The `.nojekyll` file prevents GitHub's Jekyll processor from interfering with the static files.
