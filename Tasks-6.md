# Task 6 — Vote-based AI re-ranking + polish + docs

Depends on **Tasks 3–5**. Turns the stored votes into smarter pulls, then tidies up.

---

## Goal

Use the user's 👍/👎 history to **rank, order, and lightly filter** newly fetched articles so
the feed surfaces what they're likely to want. Falls back cleanly when no AI key is set.

## Files

- `home.html` (logic), `README.md` + `ROADMAP.md` (docs).

## Implementation steps

### 1. Build a compact taste profile from votes

```js
function tasteProfile(){
  const votes = readVotes();
  const liked = [], disliked = [];
  Object.values(votes).forEach(v=>{
    const tag = [v.title, v.topic && "("+v.topic+")", v.source].filter(Boolean).join(" ");
    (v.vote===1 ? liked : disliked).push({tag, ts:v.ts});
  });
  // Most-recent first; cap to keep the prompt small + cheap.
  const recent = a => a.sort((x,y)=>y.ts-x.ts).slice(0,20).map(o=>o.tag);
  return { liked:recent(liked), disliked:recent(disliked) };
}
```

(`vote===-1` items are already removed from candidates by Task 3's suppression, so `disliked`
here is mostly *signal* for the model — "this kind of thing, less of it" — rather than a hard
filter.)

### 2. Re-rank step inside `buildNewsFeed` (after summaries, before caching)

When `newsAIConfig(w)` returns a key **and** the profile has any signal:

- Send one AI call: the taste profile + the candidate list (index, title, topic, source,
  summary) → ask for a JSON array of candidate indices ordered best-first, optionally with a
  0–1 score; instruct it to **demote** items resembling dislikes and **promote** items
  resembling likes, **without inventing or dropping** items (we still show everything, just
  reordered — keep it predictable).
- Reorder `items` by the returned order; ignore/append any indices the model omits so nothing
  silently vanishes. On any parse/HTTP error, **keep the original order** (never fail the feed
  over ranking).
- Reuse `aiComplete(ai, RANK_SYS, payload, maxTok)`; add a `RANK_SYS` constant near
  `SUMMARY_SYS` (~home.html:1432). Keep `max_tokens` small (indices only).

```js
const RANK_SYS = "You are a personalization engine. Given a user's liked and disliked article "
 + "signals and a numbered list of candidate articles, return ONLY JSON shaped exactly as "
 + '{"order":[<indices best-first>]} reordering ALL candidates so ones resembling likes come '
 + "first and ones resembling dislikes come last. Include every index exactly once. No prose.";
```

### 3. Cost control

- Ranking is **one** extra call per rebuild, gated by the same refresh window as the summary
  call — no per-scroll calls.
- Skip ranking entirely when there are 0 votes (nothing to personalize) or no AI key.
- Optional: fold ranking into the *same* call as summarization if you want to save a request
  (one prompt that returns summaries + order). Acceptable, but separate calls are simpler to
  reason about — only merge if cost matters.

### 4. Polish pass

- **Dedupe edge cases:** confirm the seen-store prune (48h) runs every build and doesn't grow
  unbounded; confirm a story under two topics shows once.
- **Vote ↔ rank coherence:** thumbs-down both removes the card now *and* feeds future ranking;
  thumbs-up keeps the card and boosts similar future stories.
- **Reset:** extend **Settings → Advanced → Reset everything** to also clear `myhome.news.*`
  keys. Consider a smaller "Clear news votes / Reset Reading List" affordance in the News tab.
- **Empty/degraded paths** all render sensibly (no topics, no key, all-suppressed).

### 5. Docs

- **README.md:** rewrite the **News Feeds** + **AI Summaries** sections to describe topics +
  Google News search + thumbs/save + Reading List + 48h dedupe + vote-based ranking. Update
  the **Caching & Local Storage** table with the four `myhome.news.*` keys. Update the
  **Settings Panel** table (News tab) and **Data Sources** (add Google News RSS search).
- **ROADMAP.md:** move the old feed bullets to Shipped as appropriate and add a "News Feed
  overhaul" Shipped entry; bump the schema note to `__v: 16`.

## Acceptance criteria

- With several 👍/👎 votes and an AI key, a rebuild visibly reorders the feed toward liked
  topics/sources; with ranking disabled/no key/no votes, order is the natural merge order.
- A ranking failure (kill the network mid-call) leaves the feed in original order, not broken.
- **Reset everything** wipes votes, saved, seen, and cache.
- README + ROADMAP reflect the new feature accurately.

## Manual test

1. Thumb up a few items in topic A and thumb down items in topic B; ⟳; confirm A-like stories
   rise and B-like stories sink.
2. Reset news votes; ⟳; confirm neutral order returns.
3. Reset everything; confirm all `myhome.news.*` keys are gone and the feed reseeds.
