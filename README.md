# desouza-flashcards

A single-file flashcard study app with spaced repetition, built for CSV-based decks. No install, no server, no dependencies — just open the HTML file.

**Live**: https://m4zzi.github.io/desouza-flashcards/

---

## Features

- **Study mode** — SM-2 spaced repetition. Rate each card (Again / Hard / Good / Easy) and cards resurface based on how well you know them.
- **Teaching mode** — Browse all cards with answers visible. Good for a first pass through new material.
- **Category filtering** — Cards are grouped by category. Select which categories to include before starting a session.
- **Progress tracking** — Per-category performance bars showing Again / Hard / Good / Easy breakdown.
- **Default decks** — The CSVs listed in `DEFAULT_DECKS` load automatically, so a first visit opens ready to study. Ships with *Foundations 2022* and *Tintinalli*.
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
3. Add it to `DEFAULT_DECKS` at the top of `index.html`
4. Enable GitHub Pages → Settings → Pages → Deploy from branch: `main` / root
5. Share the Pages URL

### Local

Open `index.html` directly in a browser. Auto-load won't work over `file://` (browser security restriction), but you can drag and drop any CSV onto the upload zone.

---

## Adding or Swapping Decks

To change the default decks, edit the top of `index.html`:

```js
const DEFAULT_DECKS = [
  { file: 'Foundations 2022.csv',         name: 'Foundations 2022' },
  { file: 'Tintinalli_EM_Flashcards.csv', name: 'Tintinalli' }
];
```

- `file` is the CSV in the repo root and the deck's identity. Card ids, and so saved progress, are built from it, and it is how the app recognizes the deck under any name: a default is not added over a copy the user uploaded by hand, and uploading the file again updates the existing deck instead of adding a second one. **Never rename a file once it has shipped.**
- `name` is what the app displays.
- Each browser receives each default once. A deck added to the list later reaches returning users on their next visit; a default the user removes stays removed, unless they remove every deck, which brings all the defaults back.
- A deck already loaded in a browser is not refreshed when its CSV changes, so a fix to a shipped deck reaches only browsers that have not loaded it yet.

Set to `[]` to disable auto-load and always show the upload zone instead.

Multiple decks can be loaded manually via the upload zone and studied together or separately.

---

## Tech

Single HTML file — no build step, no framework, no external dependencies beyond Google Fonts. All state lives in `localStorage`.
