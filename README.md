# desouza-flashcards

A single-file flashcard study app with spaced repetition, built for CSV-based decks. No install, no server, no dependencies beyond Google Fonts — just open the HTML file.

**Live**: https://m4zzi.github.io/desouza-flashcards/

---

## Features

- **Study mode** — SM-2 spaced repetition. Rate each card (Again / Hard / Good / Easy) and cards resurface based on how well you know them.
- **Teaching mode** — Browse all cards with answers visible. Good for a first pass through new material.
- **Category filtering** — Cards are grouped by category. Select which categories to include before starting a session.
- **Progress tracking** — Per-category performance bars showing Again / Hard / Good / Easy breakdown.
- **Default decks** — The CSVs listed in `DEFAULT_DECKS` load automatically, so a first visit opens ready to study. A corrected CSV, with its `version` counted up, reaches browsers that already hold the deck on their next visit with a connection. Ships with *Foundations 2022* and *Tintinalli*.
- **Persistent** — Decks and progress are saved in this browser's `localStorage`, so returning to the page in the same browser picks up where you left off.
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
- Tab-delimited files can be uploaded too; a default deck must be comma-separated (below)
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
  { file: 'Tintinalli_EM_Flashcards.csv', name: 'Tintinalli',       version: 3 }
];
```

- `file` is the CSV in the repo root and the deck's identity. Each deck records the CSV it was built from, and card ids, and so saved progress, are built from the filename. That is how the app recognizes the deck under any name: a default is not added over a copy the user uploaded by hand, and uploading the file again updates every deck built from it instead of adding another. **Never rename a file once it has shipped.**
- A default CSV is checked before the app replaces anything with it. A file that fails is skipped, and browsers keep the copy they hold:
  1. its header row is exactly `question,answer,category`;
  2. every line has three comma-separated fields, with its quotes paired;
  3. every card has an answer;
  4. it has at least as many rows as the highest-numbered card in a copy the browser got from a shipped version, since rows are never deleted. A copy uploaded by hand sets no minimum, since it may hold rows of its own.

  These checks stop an error or login page, a truncated file and broken quoting from replacing the deck. They cannot tell whether a file that passes all four is complete; for example, a last answer cut short still passes. Check the file before deploying.
- `name` is what the app displays.
- `version` names the edition of the CSV. Start a new deck at `1`, and count up every time its CSV changes (next section).
- Each browser receives each default once. A deck added to the list later reaches returning users on their next visit. A default the user removes stays removed, unless they remove every deck, which brings all the defaults back.

Set to `[]` to stop loading defaults. Decks a browser has already stored stay.

Multiple decks can be loaded manually via the upload zone and studied together or separately.

### Correcting a deck that has shipped

Every browser keeps its own copy of each deck it has loaded. To get a fix into those copies:

1. **Edit the CSV without moving any card.** A card's id is its row number, and saved progress is keyed by id. Change a card by editing its row in place, and add new cards at the end. Never insert, delete or reorder rows, the header included, and keep each card on one line: the app does not read line breaks inside a field.
   - To retire a card, delete its question and leave the rest of the row. The app skips a row without a question, and the rows below keep their numbers. Do not blank the whole line: blank lines are dropped before rows are counted.
   - A CSV with every card retired empties the deck in browsers that hold it, and is not added to browsers that do not. To stop shipping a deck to new browsers, remove it from `DEFAULT_DECKS`; browsers that already hold it keep their copy, with no further updates.
2. **Count up the deck's `version`** in `DEFAULT_DECKS`, in the same commit as the CSV.
3. **Deploy.**

On its next visit with a connection, a browser that holds the deck at any other version downloads the CSV and replaces the deck's cards in place. So does one that holds it at an unknown version: a copy loaded before versions existed, or one uploaded by hand. The deck keeps the name the user gave it, its place in the list and its selection. Every unchanged card keeps its progress. A studied card whose question or answer changed starts over with a fresh schedule and is due at once, so the correction gets studied instead of waiting out an interval earned on the old text.

How that holds: saved progress is stamped with a 53-bit fingerprint of the question and answer it was earned on, and it counts only while the card reads the same. So a changed card starts over however its change arrived: an update, a hand upload, a second open tab, or a save that failed partway. If rows do shift, a shifted card whose text differs from the card that held its row starts over instead of inheriting that card's progress. Two rows with identical question and answer can trade places unnoticed.

Progress the app cannot place counts as fresh too: the card starts over rather than keep a schedule that may have been earned on other text. That covers three cases:
- progress saved before fingerprints existed for a card the browser no longer held when it first ran this version;
- progress two copies of a deck shared where their texts differ;
- anything written by a tab still running an older version of the app.

An update that lands while a session is open updates the session too: its cards show their new text and a retired card drops out. In study mode, a card on screen that changed is shown again from its question, and only a session on screen takes ratings, so a rating always goes to text the user has just read. Uploading a deck's CSV again by hand restarts changed cards in the same way, in every deck built from that CSV. Another open tab takes up the change and does the same.

Limits:

- A CSV changed without a new `version` reaches only browsers that load the deck for the first time, or that hold a copy of unknown version (one uploaded by hand).
- An update arrives only on a visit with a connection. The page itself may be cached for up to an hour (Firebase Hosting's default), so a browser that opened the app just before the deploy can take that long to see it.
- An update never brings back a deck the user removed.
- A CSV uploaded by hand under a default's filename is that default, so its next update replaces it with the shipped CSV.
- Everything is stored in this browser's `localStorage`, which has no transactions:
  - Progress saved in one open tab is merged into the others card by card, but two tabs saving in the same instant can lose one rating.
  - A tab that saves its decks from an out-of-date copy can undo another tab's change to a custom deck. A default deck is protected: a tab never writes an older edition of a default back over the current one.
  - If the browser cannot store a removal, the removed default can come back.
- Older versions of the app could leave two copies of a deck built from one CSV under the same card ids, so the copies shared progress. Each visit, a later copy whose text differs from the first at any shared card gets ids of its own. Copies whose text agrees keep sharing, which is harmless.

---

## Tech

Single HTML file — no build step, no framework, no external dependencies beyond Google Fonts. Decks and progress live in `localStorage`; selections, mode and the open session last only while the page is open.
