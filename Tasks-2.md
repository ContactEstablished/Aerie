# Task 2 — Topics UI (pill/chit input) + Settings rewrite

Depends on **Task 1**. Replaces the "News Feeds" settings tab with a topic editor and
removes the add-a-feed concept from the UI.

---

## Goal

Let the user manage their topic list as pills/chits (type a word/phrase → Enter adds a pill;
click ✕ on a pill to remove it), plus set the news feed's refresh interval, items-per-topic,
and AI provider/model. Wire it all to `state.widgets[newsfeed].news`.

## Files

- `home.html` only.

## Implementation steps

### 1. Build a reusable pill-input component

A small helper that renders into a container and manages an array of strings:

- **Markup:** a flex-wrap row of `.pill` chips (each `text` + a `.pill-x` ✕ button) followed
  by a borderless text `<input class="pill-input">` placeholder `Add a topic…`.
- **Behavior:**
  - `Enter` (or comma) commits the trimmed input value as a new pill if non-empty and not a
    case-insensitive duplicate; clears the input.
  - `Backspace` on an empty input removes the last pill (nice-to-have).
  - Clicking a pill's ✕ removes that pill.
  - Every change calls a `onChange(topics)` callback.
- Reuse existing pill styling if present (`--accent-soft` is the pill fill token per
  `applyTheme()`); otherwise add a `.pill`/`.pill-x`/`.pill-input` CSS block near the other
  component styles.

Expose as e.g. `mountPillInput(containerEl, topics, onChange)`.

### 2. Rewrite the "News Feeds" settings tab → "News"

- Rename the tab button at **home.html:638** from `News Feeds` to `News` (keep
  `data-tab="feeds"` to avoid churn, or rename to `data-tab="news"` and update `selectTab`
  references — your call; keep it consistent).
- Replace the tab body (the `addFeedBtn2` / feed-list block around **home.html:676**) with:
  - **Topics** — the pill input from step 1, bound to `news.topics`.
  - **Articles per topic** — number input (1–10, default 5) → `news.itemsPerTopic`.
  - **Refresh every** — select (`Off`, `15`, `30`, `60` min) → `news.refreshMins`
    (`Off` = a large sentinel so the ticker never fires).
  - **AI provider + model** — reuse the existing per-feed provider/model picker logic from
    `renderFeedEditor()` (~home.html:2821), but bound to `news.ai` instead of `w.ai`.
    Keep the `✎ Custom…` model option and the `AI_PROVIDERS`/`MODEL_OPTIONS`/`DEFAULT_MODELS`
    constants.
- On change, mutate the `newsfeed` widget's `news` object, `save()`, and re-render the feed
  body if open (call `loadWidget(newsfeedWidget, body)` or just invalidate its cache).

### 3. Remove the old feed-management UI

- Delete the **`＋ Add feed`** button (`addFeedBtn2`, ~home.html:676) and its handler.
- Delete the top-bar **`＋ Feed`** button if present, and any `addFeed()` callers.
- `addFeed()` (~home.html:2710) and `renderFeedEditor()` (~home.html:2821) can be removed
  **or** left dormant — but nothing should reference them in the UI. Prefer removal to keep
  the file lean; verify no other call sites first (search for `addFeed(`, `renderFeedEditor`).
- Keep `feedAIConfig`, `parseFeedItems`, `buildAIFeed`, `renderForYou`, the `FEED_KEY` cache,
  and `aiComplete` — Task 3 reuses them.

### 4. (Optional) inline topic editing on the card

A small ✚/edit affordance in the News Feed widget header that opens the same pill editor in a
popover (mirroring the Todo widget's ✨ popover, `toggleTodoAI` ~home.html:2113). Not required
for this task; can be deferred to polish.

## Acceptance criteria

- The Settings **News** tab shows the topic pills, articles-per-topic, refresh, and AI
  provider/model — all reading/writing `state.widgets[newsfeed].news` and persisting across
  reload.
- Typing a topic + Enter adds a pill; ✕ removes it; duplicates (case-insensitive) are ignored.
- No `＋ Add feed` / `＋ Feed` UI remains; no console errors from removed handlers.
- Changing provider/model updates `news.ai` and (after save) triggers a rebuild on next load.

## Manual test

1. Open Settings → News. Add/remove a few topic pills; reload; confirm they persist.
2. Switch provider and model; confirm `news.ai` updates in `localStorage`.
3. Confirm the old add-feed controls are gone and the page has no errors.
