# Task 3 — Google News fetch + aggregated feed builder

Depends on **Tasks 1–2**. This is the data engine: turn the topic list into a deduped,
summarized, cached list of article cards. (Rendering is Task 4; vote-based ranking is Task 6.)

---

## Goal

Implement `buildNewsFeed(w)` → returns the rendered article array
`[{key,title,summary,image,link,source,topic,when}]`, plus `loadNewsFeed(w, body)` which
handles cache, refresh window, loading/error states, and calls the renderer (Task 4).

## Files

- `home.html` only.

## Implementation steps

### 1. Topic → Google News RSS search URL

```js
function googleNewsUrl(topic){
  return "https://news.google.com/rss/search?q=" + encodeURIComponent(topic) +
         "&hl=en-US&gl=US&ceid=US:en";
}
```

Reuse `parseFeedItems(url, limit)` (~home.html:1300) to fetch + parse — it already handles
rss2json + the CORS-proxy/XML fallback and image extraction. Pass `limit = news.itemsPerTopic`.

> **Source name:** Google News titles look like `"Headline - Source"` and the RSS carries a
> `<source>` element. In `parseFeedItems`'s XML path, capture `it.querySelector("source")`
> text into a `source` field; for rss2json results, derive source from the trailing
> `" - Source"` in the title (and strip it from the displayed title). Add `source` to the
> object returned by `parseFeedItems` (additive — existing feed callers ignore it).

### 2. Aggregate across topics

```js
async function buildNewsFeed(w){
  const cfg = w.news, perTopic = cfg.itemsPerTopic || 5;
  // Fetch all topics concurrently (bounded), tag each item with its topic.
  const perTopicResults = await mapLimitCollect(cfg.topics, 4, async topic => {
    const items = await parseFeedItems(googleNewsUrl(topic), perTopic).catch(()=>[]);
    return items.map(it => ({ ...it, topic, source: it.source || sourceFromTitle(it.title), key: newsKey(it) }));
  });
  let items = perTopicResults.flat();
  // … dedupe (step 3) … image+text enrich (step 4) … summarize (step 5) … rank (Task 6) …
}
```

(`mapLimit` at ~home.html:1530 doesn't collect return values — either add a collecting
variant `mapLimitCollect` or push into an array inside the worker.)

### 3. Dedupe + suppression

In order:

1. **Within this pull:** collapse duplicate `key`s (same story surfaced under two topics) —
   keep the first; optionally record all topics it matched.
2. **Disliked:** drop any `key` with `votes[key].vote === -1` (permanent suppression).
3. **Already saved:** drop any `key` already in the Reading List (no point re-showing).
4. **Seen in last 48h:** drop any `key` in `myhome.news.seen` with `ts > Date.now() - 48*3600e3`.
   Prune older-than-48h entries from the seen store while you're there.

After the feed is built and rendered, **stamp** every surviving article's `key` into the
seen store with `Date.now()` so it won't be re-grabbed for 2 days.

> If suppression empties the feed (everything seen/disliked), still render whatever survives;
> if truly empty, show an empty state inviting the user to add topics or wait for refresh.

### 4. Enrich with image + text (best effort)

Reuse the per-article enrichment loop from `buildAIFeed()` (~home.html:1387): `mapLimit(items,
4, …)` → `fetchProxiedText(it.link)` → `pickOgImage(doc, it.link)` and `pickArticleText(doc)`.

> **Known limitation — Google News redirect links:** `it.link` is a
> `news.google.com/rss/articles/…` redirect, which often won't yield a clean `og:image` via
> the proxy. Fallback order for the image: rss2json/RSS thumbnail → resolved-article og:image
> (if reachable) → none (card renders text-only). Don't block the feed on image fetches;
> cap time and move on. AI cannot invent an image — we surface what's available.

### 5. AI summary (graceful degrade)

```js
const ai = newsAIConfig(w);   // {provider,model,key} or null if no key for chosen provider
if(ai){
  const summaries = await aiSummarize(ai, items.map(it=>({title:it.title,
      content:(it.text||stripHtml(it.desc)).replace(/\s+/g," ").trim().slice(0,1400)})));
  items.forEach((it,i)=> it.summary = (summaries[i]||stripHtml(it.desc).slice(0,150)||"").trim());
} else {
  items.forEach(it => it.summary = stripHtml(it.desc).slice(0,150).trim());  // no key → snippet
}
```

`newsAIConfig(w)` mirrors `feedAIConfig` (resolve `w.news.ai.provider/model` against
`state.ai.keys`; return `null` when AI disabled or the key is missing → degrade gracefully).

### 6. `loadNewsFeed(w, body)` — cache + refresh + dispatch

Model on `loadFeedAI()` (~home.html:1365):

- Compute `sig = JSON.stringify({topics:cfg.topics, perTopic, provider, model})`.
- Read `myhome.news.cache`; if `cache.sig===sig` and `Date.now()-cache.ts <
  feedRefreshMins(w)*60000` → render from cache (no network).
- Else show the spinner state, `buildNewsFeed(w)`, write cache `{ts,sig,items}`, stamp seen
  store, and call `renderNewsFeed(w, body, items, ts)` (Task 4).
- On error with a stale cache, render the stale cache rather than failing hard.
- Set `feedLoadedAt[w.id]=Date.now()` and `startFeedTicker()` (Task 1 made the ticker
  newsfeed-aware).
- The ⟳ refresh button (header `data-act="refresh"`) should clear `myhome.news.cache` before
  reloading — extend the refresh handler (~home.html:1099) to clear the news cache when
  `w.type==="newsfeed"`.

## Acceptance criteria

- With topics set and an AI key present, the feed builds: each topic contributes up to
  `itemsPerTopic` articles, merged into one list with summaries and (where available) images.
- Re-opening within the refresh window does **no** network calls (served from cache); after
  the window, it rebuilds. ⟳ forces a rebuild.
- An article shown today is **not** present in a fresh pull for 48h (seen store works).
- Disliked and already-saved articles never appear.
- With **no** AI key, the feed still builds with snippet summaries (no crash).

## Manual test

1. Set topics (e.g. `Formula 1`, `AI`) and an AI key; open the News Feed; confirm a merged,
   summarized list with sources/timestamps.
2. Reload within the window → instant (check Network panel shows no feed calls). Hit ⟳ →
   rebuilds.
3. Inspect `myhome.news.seen` / `myhome.news.cache` in DevTools after a build.
4. Clear the AI key; ⟳; confirm snippets render without errors.
