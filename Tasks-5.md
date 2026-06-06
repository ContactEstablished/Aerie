# Task 5 — Reading List widget

Depends on **Task 1** (stores) and the `newsSave()` writer from **Task 4**. Can be built in
parallel with Task 4.

---

## Goal

Render the saved articles (`myhome.news.saved`) to the right of the News Feed. Each entry:
click to read in a new tab, or **mark as read** to remove it from the list.

## Files

- `home.html` only.

## Implementation steps

### 1. `loadReadingList(w, body)` / `renderReadingList(body)`

- Read `readSaved()` (newest first — `newsSave` unshifts).
- Per entry, render a compact row:
  ```
  [thumb]  Headline (links to article, new tab)
           Source · saved 2h ago        [✓ Read]
  ```
  - The headline/thumbnail link opens `it.link` in a new tab.
  - A **`✓ Read`** button (`data-act="read" data-key=…`) removes the entry.
- **Empty state:** "Saved articles show up here. Hit 💾 on a story in your feed." (mirrors the
  feed's empty-state styling).
- Use delegated `click` handling on the list container, like Task 4.

### 2. Mark-as-read

```js
function readingMarkRead(key){
  writeSaved(readSaved().filter(s=>s.key!==key));   // "disappears from the list"
  refreshReadingList();
}
```

> Decision: marking read **removes** the entry (the user said "it just disappears"). We do
> **not** keep a read archive — preference learning lives in the votes store, not here. (If we
> later want "recently read," revisit; out of scope now.)

### 3. `refreshReadingList()` helper (shared with Task 4)

```js
function refreshReadingList(){
  const rl = state.widgets.find(w=>w.type==="readinglist");
  const el = rl && document.querySelector(`.w[data-id="${rl.id}"] .w-body`);
  if(el) loadWidget(rl, el);
}
```

(Define this once; Task 4's `newsSave` calls it. If both tasks are in flight, whoever lands
first defines it.)

### 4. Header buttons

In the card-header builder (~home.html:1056), make `readinglist`:

- **not removable** (singleton) — minimize only;
- no ⟳ needed (it's local state; it re-renders on save/read). A ⟳ that just re-renders is
  harmless if you prefer consistency.

### 5. CSS

Add `.rl-row` / `.rl-read` styles near `.news-card`. Compact, single-line-ish rows so the
narrow column (3 grid cols) stays readable.

## Acceptance criteria

- Saving from the feed (Task 4) makes an entry appear here immediately (newest on top).
- Clicking an entry opens the article in a new tab.
- **✓ Read** removes the entry; it persists removed across reloads (`myhome.news.saved`
  updated).
- Empty state renders when nothing is saved.

## Manual test

1. Save 2–3 articles from the feed → confirm they list here, newest first, with thumbnails.
2. Click one → opens in a new tab.
3. Mark one read → it disappears → reload → still gone.
4. Clear all → empty state shows.
