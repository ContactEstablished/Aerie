# Task 4 — News Feed widget: cards + thumbs/save actions

Depends on **Task 3** (the builder). Can be built in parallel with **Task 5**.

---

## Goal

Render the aggregated article list as cards and wire the three per-article actions:
👍 thumbs up, 👎 thumbs down, 💾 save. Use the Todo widget (`renderTodoList` ~home.html:2127)
as the pattern for an interactive card with `data-act` buttons and event delegation.

## Files

- `home.html` only.

## Implementation steps

### 1. `renderNewsFeed(w, body, items, ts)`

Build on `renderForYou()` (~home.html:1495) but add per-card action buttons and a source/topic
line. Per card:

```
┌───────────────────────────────────────────────┐
│ [thumb]  Headline (links to article, new tab)  │
│          one-line AI summary (or snippet)      │
│          Source · topic · 3h ago               │
│          👍   👎   💾 Save                       │
└───────────────────────────────────────────────┘
```

- The headline/thumbnail link opens `it.link` in a new tab (`target="_blank" rel="noopener"`).
- Each action button carries `data-act` + the article `key`, e.g.
  `<button class="na-vote" data-act="up"   data-key="${esc(it.key)}">👍</button>`
  (`up` / `down` / `save`). Attach **one** delegated `click` listener on the list container
  (cleaner than per-button), `e.stopPropagation()` so clicks don't trigger the card link.
- Reflect current state on render: if `votes[key].vote===1` show 👍 active; if already in the
  Reading List, show the save button as "Saved ✓" (disabled).
- Header meta line: reuse the `renderForYou` "updated … · next refresh in Nm" treatment via
  `nextRefreshLabel(ts, w)`.

### 2. Action handlers

```js
function newsVote(w, key, dir){            // dir: 1 (up) or -1 (down)
  const votes = readVotes(), cur = votes[key]?.vote;
  const meta = findItem(w, key);           // pull title/source/topic from current feed items
  if(cur===dir){ delete votes[key]; }      // toggle off
  else { votes[key] = {vote:dir, ts:Date.now(), title:meta?.title, source:meta?.source, topic:meta?.topic}; }
  writeVotes(votes);
  if(dir===-1 && votes[key]?.vote===-1) removeItemFromFeedCache(w, key);  // 👎 removes from feed
  reloadNewsBody(w);                        // re-render from cache (no network)
}
function newsSave(w, key){
  const it = findItem(w, key); if(!it) return;
  const saved = readSaved();
  if(saved.some(s=>s.key===key)) return;    // already saved
  saved.unshift({ key:it.key, title:it.title, summary:it.summary, image:it.image,
                  link:it.link, source:it.source, topic:it.topic, savedAt:Date.now() });
  writeSaved(saved);
  reloadNewsBody(w);                         // mark this card "Saved ✓"
  refreshReadingList();                      // re-render the Reading List widget (Task 5)
}
```

Behavior rules (locked with the user):

- **👍 up:** records a like; the article **stays** in the feed (subtle highlight / active
  state). Toggling clears the vote.
- **👎 down:** records a dislike; the article is **removed from the feed immediately** and is
  permanently suppressed from future pulls (Task 3 dedupe reads `vote===-1`).
- **💾 save:** copies the article to the Reading List; it **stays** in the feed, the button
  flips to "Saved ✓".

`removeItemFromFeedCache(w, key)` mutates the cached `items` array (drop the key) and rewrites
`myhome.news.cache`, then re-renders from cache so 👎 feels instant without a network rebuild.

`refreshReadingList()` finds the `readinglist` widget's body in the DOM and calls
`loadWidget(rl, body)` (defined in Task 5).

### 3. States

- **Loading:** `loadingHTML` (spinner) — already shown by `loadNewsFeed`.
- **No topics:** empty state — "Add topics in ⚙ Settings → News to build your feed."
- **No AI key:** a one-line `keyhint` banner above the cards (per the graceful-degrade
  decision) — "Add an AI key in ⚙ Settings → AI to get summaries and smarter ranking." Cards
  still render with snippets + working actions.
- **Error (no cache):** `errorHTML(...)`.

### 4. Header buttons for the `newsfeed` widget

In the card-header builder (~home.html:1056–1069), ensure `newsfeed` gets:

- a **⟳ refresh** button (add `newsfeed` to the `refreshBtn` condition at ~home.html:1059);
- **not** removable (omit from the `removable` set at ~home.html:1056) — it's a singleton like
  Photo of the Day; only minimize. (Same for `readinglist` in Task 5.)
- (Optional) a ✚/✨ button opening the topic pill editor in a popover (see Task 2 step 4).

### 5. CSS

Add a `.news-card` / `.na-vote` / `.na-save` block near the `.fy-*` styles. Keep the dark,
glassy look; active vote = `--accent` tint; "Saved ✓" = muted/disabled. Thumbnails reuse the
`.fy-thumb img` treatment (`referrerpolicy="no-referrer"`, `loading="lazy"`, `onerror` removes
the thumb).

## Acceptance criteria

- Cards render headline + summary + source/topic/time + thumbnail (when available) + the three
  action buttons.
- 👍 toggles an active state and writes `myhome.news.votes`; 👎 removes the card immediately and
  it does not return on ⟳; 💾 adds to `myhome.news.saved`, flips to "Saved ✓", and the Reading
  List updates live.
- No-key mode shows the hint banner but fully functional cards.
- Clicking a headline opens the article in a new tab; action clicks do **not** also open it.

## Manual test

1. Build a feed (Task 3). Thumbs-up an item → reload page → still active.
2. Thumbs-down an item → it vanishes → ⟳ → it stays gone.
3. Save an item → it appears in the Reading List immediately and shows "Saved ✓" here.
4. Toggle the AI key off/on → hint banner appears/disappears; actions keep working.
