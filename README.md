# desouza-flashcards

A single-file flashcard study app with spaced repetition, built for CSV-based decks. No install, no server, no dependencies — just open the HTML file.

**Live**: https://m4zzi.github.io/desouza-flashcards/

---

## Features

- **Study mode** — SM-2 spaced repetition. Rate each card (Again / Hard / Good / Easy) and cards resurface based on how well you know them.
- **Teaching mode** — Browse all cards with answers visible. Good for a first pass through new material.
- **Category filtering** — Cards are grouped by category. Select which categories to include before starting a session.
- **Progress tracking** — Per-category performance bars showing Again / Hard / Good / Easy breakdown.
- **Default decks** — The CSVs listed in `DEFAULT_DECKS` load automatically, so a first visit opens ready to study, and a corrected CSV reaches browsers that already hold the deck. Ships with *Foundations 2022* and *Tintinalli*.
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
  { file: 'Foundations 2022.csv',         name: 'Foundations 2022', version: 1 },
  { file: 'Tintinalli_EM_Flashcards.csv', name: 'Tintinalli',       version: 1 }
];
```

- `file` is the CSV in the repo root and the deck's identity. Card ids, and so saved progress, are built from it, and it is how the app recognizes the deck under any name: a default is not added over a copy the user uploaded by hand, and uploading the file again updates the existing deck instead of adding a second one. **Never rename a file once it has shipped.** A default CSV must open with its header row (`question,answer,category`): the app checks for it, so a login or error page served in the file's place is never loaded as the deck.
- `name` is what the app displays.
- `version` names the edition of the CSV. Start a new deck at `1`, and count up every time its CSV changes (next section).
- Each browser receives each default once. A deck added to the list later reaches returning users on their next visit; a default the user removes stays removed, unless they remove every deck, which brings all the defaults back.

Set to `[]` to disable auto-load and always show the upload zone instead.

Multiple decks can be loaded manually via the upload zone and studied together or separately.

### Correcting a deck that has shipped

Every browser keeps its own copy of each deck it has loaded. To get a fix into those copies:

1. **Edit the CSV without moving any card.** A card's id is its row number, and saved progress is keyed by id. Change a card by editing its row in place, and add new cards at the end. Never insert, delete or reorder rows, the header included, and keep each card on one line: the app does not read line breaks inside a field.
   - To retire a card, delete its question and leave the rest of the row. The app skips a row without a question, and the rows below keep their numbers. Do not blank the whole line: blank lines, and in a tab-separated file lines of nothing but tabs, are dropped before rows are counted.
   - Keep at least one card in the deck. A CSV with no cards left reads as a failed download, so browsers keep the copy they hold and download the file again on every visit. To withdraw a whole deck, remove it from `DEFAULT_DECKS`: a browser that does not hold it yet never receives it, but one that already holds it keeps that copy, which gets no further updates.
2. **Count up the deck's `version`** in `DEFAULT_DECKS`, in the same commit as the CSV.
3. **Deploy.**

On its next visit, a browser that holds the deck at any other version downloads the CSV and replaces the deck's cards in place. So does one that holds it at an unknown version: a copy loaded before versions existed, or one uploaded by hand. The deck keeps the name the user gave it, its place in the list and its selection. Every unchanged card keeps its progress. A studied card whose question or answer changed starts over with a fresh schedule and is due at once, so the correction gets studied instead of waiting out an interval earned on the old text. If rows do shift, each shifted card no longer matches its old text, so it starts over instead of inheriting another card's progress.

An update that lands while a session is open updates the session too: its cards show their new text and a retired card drops out. In study mode, a card on screen that changed is shown again from its question, and only a session on screen takes ratings, so a rating always goes to text the user has just read. Uploading a deck's CSV again by hand restarts changed cards in the same way, in every deck built from that CSV.

Limits:

- A CSV changed without a new `version` reaches only browsers that load the deck for the first time.
- An update arrives only on a visit with a connection. The page itself may be cached for up to an hour (Firebase Hosting's default), so a browser that opened the app just before the deploy can take that long to see it.
- An update never brings back a deck the user removed.
- A CSV uploaded by hand under a default's filename is that default, so its next update replaces it with the shipped CSV.
- Saved progress is kept per card id, so two decks built from one CSV share it. Only older versions of the app could create such a pair. Updating a default, or uploading that CSV again, gives both decks the same text, but a pair of custom decks left with different text keeps sharing progress until then: rating a card in one reschedules the same card in the other.

---

## Tech

Single HTML file — no build step, no framework, no external dependencies beyond Google Fonts. All state lives in `localStorage`.
