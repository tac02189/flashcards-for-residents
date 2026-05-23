# desouza-flashcards

A single-file flashcard study app with spaced repetition, built for CSV-based decks. No install, no server, no dependencies — just open the HTML file.

**Live**: https://m4zzi.github.io/desouza-flashcards/

---

## Features

- **Study mode** — SM-2 spaced repetition. Rate each card (Again / Hard / Good / Easy) and cards resurface based on how well you know them.
- **Teaching mode** — Browse all cards with answers visible. Good for a first pass through new material.
- **Category filtering** — Cards are grouped by category. Select which categories to include before starting a session.
- **Progress tracking** — Per-category performance bars showing Again / Hard / Good / Easy breakdown.
- **Auto-load** — If a CSV is placed in the same folder as the HTML, it loads automatically on first visit.
- **Persistent** — Progress is saved to `localStorage`. Returning to the page picks up where you left off.
- **Dark mode** — Follows system preference.

---

## CSV Format

```
question, answer, category
What is the powerhouse of the cell?, Mitochondria, Biology
What year did WWII end?, 1945, History
```

- Column order: `question`, `answer`, `category`
- `category` is optional — cards without one are grouped under *Uncategorized*
- Header row is auto-detected and skipped
- Tab-delimited files are also supported
- Quoted fields with commas inside work fine: `"Hard, harder, hardest", The answer, Category`

---

## Usage

### Hosted (recommended)

1. Fork this repo
2. Drop your CSV into the repo root
3. Update `DEFAULT_CSV` at the top of `flashcards.html` to match the filename
4. Enable GitHub Pages → Settings → Pages → Deploy from branch: `main` / root
5. Share the Pages URL

### Local

Open `flashcards.html` directly in a browser. Auto-load won't work over `file://` (browser security restriction), but you can drag and drop any CSV onto the upload zone.

---

## Adding or Swapping Decks

To change the default deck, edit the top of `flashcards.html`:

```js
const DEFAULT_CSV = 'your-deck.csv';
```

Set to `''` to disable auto-load and always show the upload zone instead.

Multiple decks can be loaded manually via the upload zone and studied together or separately.

---

## Tech

Single HTML file — no build step, no framework, no external dependencies beyond Google Fonts. All state lives in `localStorage`.
