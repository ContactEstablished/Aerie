# Task 1 — Foundation: data model, migration, storage

Part of the **News Feed overhaul**. This task lays the data foundation; nothing visual
changes for the user yet, but the app must still load and render without errors after it.

---

## Design summary (applies to all News tasks)

Agreed direction (decisions locked with the user):

- **Replace**, don't coexist: the per-source feed widgets (CNN/BBC/Verge, `type:"feed"`)
  are removed and replaced by **one `newsfeed` widget** (left) + **one `readinglist`
  widget** (right of it).
- **Topics, not feeds:** the user maintains a list of topic strings (pills/chits). Each
  topic is searched on Google News; we take the **top 5 articles per topic**.
- **Google News search = RSS search endpoint** (no HTML scraping):
  `https://news.google.com/rss/search?q=<topic>&hl=en-US&gl=US&ceid=US:en`.
  This flows through the existing rss2json → CORS-proxy pipeline already used by feeds.
- **AI** produces the one-line summary and helps rank (reusing the existing summarize
  pipeline). **Degrade gracefully** when no AI key is set: titles + thumbnails + working
  thumbs/save still render; summaries + ranking appear once a key exists.
- **Votes** (👍/👎) are remembered and used to **AI re-rank** future pulls (Task 6).
- **Save** copies an article to the **Reading List**; thumbs-down removes it from the feed
  and suppresses it permanently; both leave the article in the feed otherwise.
- **Dedupe:** an article already shown is not re-fetched for **48h**.
- **Seed topics** on first run / after migration: `Technology`, `World news`, `Business`.

Task order: **1 → 2 → 3 → 4 → 5 → 6**. Tasks 4 and 5 can be built in parallel once 3 lands.

---

## Goal

Introduce the new state shape, the `__v: 16` migration, and the localStorage stores that
the rest of the feature reads/writes.

## Files

- `home.html` only.

## Implementation steps

### 1. Update `defaultState()` (~home.html:840)

Remove the three `type:"feed"` widgets (`cnn`, `bbc`, `verge`). Add two new widgets in
their place (keep `weather`/`clock`/`art` in the right column at `c:8`):

```js
{id:"newsfeed", type:"newsfeed", title:"News Feed",
  pos:{c:0,r:0,w:5,h:9}, z:1, collapsed:false,
  news:{ topics:["Technology","World news","Business"], itemsPerTopic:5, refreshMins:30,
         ai:{provider:"deepseek", model:"deepseek-chat"} }},
{id:"readinglist", type:"readinglist", title:"Reading List",
  pos:{c:5,r:0,w:3,h:9}, z:2, collapsed:false},
```

(Re-number the `z` of weather/clock/art to 3/4/5 so they stay above.)

### 2. Add migration `__v: 16` (append inside `migrate()`, before the closing brace ~home.html:961)

```js
if(s.__v < 16){
  // News overhaul: drop per-source feed widgets; add one News Feed + one Reading List.
  const feeds  = (s.widgets||[]).filter(w=>w.type==="feed");
  const seedAi = (feeds.find(f=>f.ai && f.ai.provider && f.ai.provider!=="off")||{}).ai
              || {provider:"deepseek", model:"deepseek-chat"};
  let z = s.zCounter || 10;
  s.widgets = (s.widgets||[]).filter(w=>w.type!=="feed");
  s.widgets.unshift(
    {id:"newsfeed", type:"newsfeed", title:"News Feed",
      pos:{c:0,r:0,w:5,h:9}, z:++z, collapsed:false,
      news:{ topics:["Technology","World news","Business"], itemsPerTopic:5,
             refreshMins:(s.ai && s.ai.cacheMinutes) || 30, ai:seedAi }},
    {id:"readinglist", type:"readinglist", title:"Reading List",
      pos:{c:5,r:0,w:3,h:9}, z:++z, collapsed:false}
  );
  s.zCounter = z;
  s.__v = 16;
}
```

> Note: positions are best-effort. If the migrated layout overlaps existing
> weather/clock/art cards, that's acceptable — the user can drag to taste, and
> **Settings → Advanced → Reset layout** restores a clean arrangement.

### 3. Add the localStorage store helpers (near the existing `FEED_KEY` block ~home.html:1523)

Four new keys, each with read/write helpers mirroring `readFeeds()/writeFeedCache()`:

| Key | Shape | Purpose |
| --- | --- | --- |
| `myhome.news.cache` | `{ ts, sig, items:[…] }` | Built feed cache (one object; single feed). `sig` = signature of topics+provider+model+count so changing any rebuilds. |
| `myhome.news.votes` | `{ [key]: { vote:1\|-1, ts, title, source, topic } }` | Thumbs up/down memory. Keeps title/source/topic so re-ranking has signal after the article ages out. |
| `myhome.news.saved` | `[ { key,title,summary,image,link,source,topic,savedAt } ]` | The Reading List. |
| `myhome.news.seen`  | `{ [key]: ts }` | Dedupe ledger; entries older than 48h are pruned each build. |

```js
const NEWS_CACHE_KEY="myhome.news.cache", NEWS_VOTES_KEY="myhome.news.votes",
      NEWS_SAVED_KEY="myhome.news.saved", NEWS_SEEN_KEY="myhome.news.seen";
function readJSON(k, fallback){ try{ return JSON.parse(localStorage.getItem(k)) ?? fallback; }catch(e){ return fallback; } }
function writeJSON(k, v){ try{ localStorage.setItem(k, JSON.stringify(v)); }catch(e){} }
function readVotes(){ return readJSON(NEWS_VOTES_KEY, {}); }
function readSaved(){ return readJSON(NEWS_SAVED_KEY, []); }
function readSeen(){ return readJSON(NEWS_SEEN_KEY, {}); }
// writeVotes/writeSaved/writeSeen/readNewsCache/writeNewsCache similarly.
```

### 4. Canonical article key (used everywhere for identity/dedupe)

Google News links are unstable redirect URLs, so identity is derived from **title + source
domain**, not the URL:

```js
// "Headline - The Verge" -> "headline"; strips trailing " - Source", lowercases, alnum-only.
function newsKey(it){
  const t = String(it.title||"").replace(/\s+-\s+[^-]+$/,"").toLowerCase().replace(/[^a-z0-9]+/g," ").trim();
  const src = String(it.source||"").toLowerCase().replace(/[^a-z0-9]+/g,"");
  return t + "|" + src;
}
```

### 5. Dispatch + ticker stubs (so the app loads cleanly)

- In `loadWidget()` (~home.html:1236) add temporary stubs so the new widget types don't
  fall through to nothing:
  ```js
  if(w.type==="newsfeed")    return loadNewsFeed(w, body);     // implemented in Task 3/4
  if(w.type==="readinglist") return loadReadingList(w, body);  // implemented in Task 5
  ```
  For this task, define `loadNewsFeed`/`loadReadingList` as one-line placeholders that render
  `loadingHTML` (real bodies arrive in later tasks). This keeps the page functional.
- In `startFeedTicker()` (~home.html:1256) change the type guard from `w.type!=="feed"` to
  also auto-refresh `newsfeed` (e.g. `if(!w || w.hidden || (w.type!=="feed" && w.type!=="newsfeed")) return;`).
  `feedRefreshMins(w)` already falls back correctly; for `newsfeed`, read `w.news.refreshMins`.

## Data shapes (rendered article — used by Tasks 3/4/5)

```js
{ key, title, summary, image, link, source, topic, when }
```

## Acceptance criteria

- Opening `home.html` with **existing saved state** (old CNN/BBC/Verge layout) migrates to
  `__v:16`: feed cards gone, a `newsfeed` + `readinglist` widget present, no console errors.
- Opening with **cleared storage** yields the same two widgets from `defaultState()`.
- `localStorage["myhome.v1"].__v === 16`; the four `myhome.news.*` keys initialize lazily
  (empty `{}`/`[]`) without throwing.
- `newsKey({title:"Big News - The Verge", source:"The Verge"})` →
  `newsKey({title:"Big News", source:"theverge"})` produce the **same** key.

## Manual test

1. Back up current `localStorage` (DevTools → Application), then load `home.html`.
2. Confirm migration ran (`__v:16`) and the two new widgets render placeholders.
3. Reset everything and reload; confirm `defaultState()` path also gives two widgets.
