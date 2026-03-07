# 汉字 Flashcards

A simple offline-capable flashcard app for practising the most common Chinese characters. No account, no installation — just open the page and go.

Live: https://m1ke.github.io/hanzi

## How it works

Enter any three letters as a **seed** (e.g. `ABC`, `sun`, `cat`). The same seed always produces the same set of 20 characters, so you can share a seed with someone and study the same batch.

### Screens

| Screen | Description |
|--------|-------------|
| **Home** | Enter a seed, view previous sessions. Click a past session row to reload its seed. |
| **Preview** | See all 20 characters for a seed before you start, displayed as a grey grid. |
| **Card** | One character at a time. Type the pinyin and press Enter or Submit. Press Esc or Skip to move on without answering. |
| **Result** | Shows whether you were correct (green) or wrong/skipped (red), plus the correct pinyin. Press Enter or Next to continue. |
| **Summary** | Your score out of 20, a colour-coded grid of all 20 cards, and options to try again or start a new session. |

### Practice mode

Tick **Practice mode** on the home screen before starting. When enabled, any wrong or skipped answer shows an extra input on the result screen — you must type the correct pinyin before the Next button unlocks. Typing the answer in this way does **not** affect your score; it is purely for reinforcement.

Keyboard flow in practice mode:
1. Wrong/skip → result screen, practice input is focused automatically
2. Type the correct pinyin and press Enter (or click Check)
3. Input turns green and Next is focused — press Enter to continue

### Pinyin input

- Type bare pinyin — **no tone marks required** (`zhong` not `zhōng`)
- The displayed answer always shows the correct toned pinyin
- For characters with ü (e.g. 绿 lǜ), you can type either `lv` or `lu`
- Matching is case-insensitive

### Sessions

Each completed session is saved locally (localStorage). Up to 50 sessions are stored. Your history appears on the home screen; scores are colour-coded green (≥80%), yellow (≥50%), or red (<50%).

## Running locally

No build step required. Serve the directory with any static file server:

```sh
python3 -m http.server 8765
# or
npx serve .
```

Then open http://localhost:8765.

Opening `index.html` directly via `file://` also works in most browsers.
