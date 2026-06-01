# Aerie — Roadmap

Working list of what's done, what's next, and ideas on deck. Not a contract — priorities can move.

**Legend:** ✅ done · 🛠️ in progress · ⏭️ planned (next up) · 💡 idea / backlog
_Last updated: 2026-06-01_

---

## ✅ Shipped

- [x] Single-file dashboard (`home.html`, inline CSS/JS, zero dependencies)
- [x] Drag-to-move / resize-to-snap grid with auto-saved layout
- [x] Local weather (Open-Meteo) — IP / city / GPS, units toggle, last-good cache
- [x] News feeds via rss2json (+ CORS-proxy fallback), add/edit/remove any RSS/Atom
- [x] AI "For You" cards — article fetch → image + summary, 30-min per-feed cache
- [x] **Multi-provider AI** — DeepSeek, OpenAI, Anthropic, **provider + model per feed**
- [x] Photo of the Day — Fstoppers POTD / Art Institute / The Met, cached per day
- [x] Clock & optional Google search widgets
- [x] Theming — accent color **and** background base color pickers, themed scrollbars
- [x] Aerie branding — favicon, toolbar logo + wordmark, line-art watermark
- [x] Settings persistence + schema migrations (currently `__v: 6`)
- [x] **Tabbed Settings menu** — Weather & Clock · News Feeds · Photo of the Day · Colors · AI Settings · Advanced; widget show/hide toggles relocated into their tabs + collective "News" show/hide
- [x] **Clock upgrades** — per-card ⚙ config (analog/digital, show seconds, 24-hour, time zone + subtle tz label), multiple clocks each with its own zone, collective show/hide
- [x] README

---

## 🛠️ In Progress

- [ ] Final watermark artwork (being refined in Photoshop) — drop-in at `img/aerie-watermark.png`

---

## ⏭️ Planned (next up)

### Photo of the Day
- [ ] **Hide Photo of the Day** toggle (Settings)
- [ ] Header **shuffle button** — pick a random photo from the **last 14 days** of the chosen provider; a **reset** button returns to today's pick
  - _Keeps current behavior: first load of the day shows "today's" photo._

### Search
- [ ] Fit the search card cleanly in a **1-row-tall** card (no vertical scroll when shrunk)
- [ ] **Search-engine selector** — Google / Yahoo / Bing built-in + a custom search-URL field (Dogpile, etc.)

### Weather
- [ ] **Multiple weather cards** — several cities at once

### Feeds (carried over)
- [ ] **Better Reddit RSS image support** — Reddit `.rss` is Atom; the post image lives in the `<content>` HTML and/or `media:thumbnail`, which the current extractor misses. Fix is in the **image-extraction code** (`pickRSSImage` / rss2json mapping / `pickOgImage`), not the AI prompt (the AI only gets text and returns summaries — it never fetches images). Parse `<content>`/`media:thumbnail`, and for Reddit map `preview.redd.it` / `i.redd.it` links. Example feed: `https://www.reddit.com/r/coolgithubprojects.rss`
- [ ] Custom-model text entry in the per-feed AI selector (type any model name, not just the curated list)
- [ ] Thumbnails in the classic (no-key) feed list
- [ ] Optional rss2json API key field (raise the ~10-item cap per feed)
- [ ] "Feeds last updated" / next-refresh indicator

### Repo housekeeping
- [ ] Add hero screenshot to the README (`img/aerie-screenshot.png`)
- [ ] Add a `LICENSE` file (MIT or your choice) — README already references it

---

## 💡 Ideas / Backlog

- [ ] Import / export settings as a JSON file (backup + move between machines)
- [ ] Light-theme support with auto-contrast text (when a light background is chosen)
- [ ] Stocks / crypto ticker widget
- [ ] To-do / notes widget
- [ ] Calendar / agenda widget
- [ ] Keyboard shortcuts (open settings, refresh all, focus search)
- [ ] New-tab support via a lightweight Chrome extension wrapper
