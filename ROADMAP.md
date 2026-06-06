# Aerie — Roadmap

Working list of what's done, what's next, and ideas on deck. Not a contract — priorities can move.

**Legend:** ✅ done · 🛠️ in progress · ⏭️ planned (next up) · 💡 idea / backlog
_Last updated: 2026-06-05_

---

## ✅ Shipped

- [x] **News Feed overhaul** — replaced per-source RSS feed cards with a **topic-driven feed + Reading List**. Add **topics** (pills); each is searched on **Google News** and merged into one **accumulating** feed (newest on top, drops after 2 days, **48h dedupe** so stories don't repeat). **👍/👎/💾** per story (like / not-interested-and-suppress / save to Reading List); the Reading List opens stories in a new tab or **✓ marks them read**. With an AI key: one-line summaries + **vote-based re-ranking**; without one it degrades to headlines + working buttons. Per-feed **refresh interval** (Off/15/30/60) and **provider/model**; reset feed / clear votes / clear Reading List. Settings schema `__v: 16`
- [x] Single-file dashboard (`home.html`, inline CSS/JS, zero dependencies)
- [x] Drag-to-move / resize-to-snap grid with auto-saved layout
- [x] Local weather (Open-Meteo) — IP / city / GPS, units toggle, last-good cache
- [x] News feeds via rss2json (+ CORS-proxy fallback), add/edit/remove any RSS/Atom
- [x] AI "For You" cards — article fetch → image + summary, 30-min per-feed cache
- [x] **Multi-provider AI** — DeepSeek, OpenAI, Anthropic, **provider + model per feed**
- [x] **Feeds polish** — type a **custom model name** per feed (beyond the curated list); optional **rss2json key** to lift the ~10-item cap; **last-updated / next-refresh** indicators (AI cards show the cache countdown, classic lists show the fetch time)
- [x] Photo of the Day — Fstoppers POTD / **NASA APOD** / Art Institute / The Met, cached per day; **🔀 shuffle** (a random recent photo — last ~14 days for Fstoppers/APOD) + **📅 reset to today**; show/hide toggle
- [x] Clock & optional search widget
- [x] **Search upgrades** — built-in engine selector (Google / Bing / Yahoo) + a custom search-URL option (Dogpile, etc.) set from the card's ⚙; fits cleanly in a 1-row-tall card
- [x] **Multiple weather cards** — one card per city, each with its own location (Auto-IP / city / GPS) set from the card's ⚙; collective show/hide + per-card cache; units stay global
- [x] **Reddit RSS images** — decode HTML-encoded ampersands in extracted image URLs (rss2json thumbnails + escaped feed content) so signed `preview.redd.it` URLs no longer 403; also parse Atom `<content>` / `media:thumbnail` in the XML fallback
- [x] **Reddit post images in AI cards** — in AI "For You" mode, read a Reddit post's actual media from the post page (the clean `i.redd.it` gif/image on `<shreddit-post content-href>`, or signed `preview.redd.it` gallery screenshots, skipping avatars), so image/gallery subs like r/coolgithubprojects show the real demo/screenshots instead of a generic linked-page card
- [x] Theming — accent color **and** background base color pickers, themed scrollbars
- [x] Aerie branding — favicon, toolbar logo + wordmark, line-art watermark
- [x] Settings persistence + schema migrations (currently `__v: 10`)
- [x] **Tabbed Settings menu** — Weather & Clock · News Feeds · Photo of the Day · Colors · AI Settings · Advanced; widget show/hide toggles relocated into their tabs + collective "News" show/hide
- [x] **Clock upgrades** — per-card ⚙ config (analog/digital, show seconds, 24-hour, time zone + subtle tz label), multiple clocks each with its own zone, collective show/hide
- [x] **Clock time-zone backgrounds** — a faint per-city skyline in the bottom-right of each clock, fading toward the readout, so its location reads at a glance. Uses a clean 4×5 sprite of white line-art skylines on a transparent background (`img/time-zone-cities-white.png`, tile order matches `CLOCK_ZONES`); shown on digital + analog, including a laptop+clock tile for "Local"
- [x] **Infinite scrolling canvas + minimizable cards** — the grid is no longer capped to the viewport: cards drag/resize past the edges and the canvas grows + scrolls on both axes (and trims back when they move in), with fixed px cells sized so the base 12×9 still fills the screen. Each card has a minimize button that collapses it to its header bar (freeing the space below); expanding or dragging a card raises it to the top (persisted stacking order). Settings schema `__v: 11`
- [x] README

---

## 🛠️ In Progress

- [ ] Final watermark artwork (being refined in Photoshop) — drop-in at `img/aerie-watermark.png`

---

## ⏭️ Planned (next up)

### News
- [ ] Better per-article images — Google News links are redirect stubs, so cards are mostly text-first today; explore resolving the real article URL (or a source-level image) for thumbnails
- [ ] Optional full-feed (not just fresh-batch) re-ranking toggle

### Repo housekeeping
- [ ] Add hero screenshot to the README (`img/aerie-screenshot.png`)
- [ ] Add a `LICENSE` file (MIT or your choice) — README already references it

---

## 💡 Ideas / Backlog

- [ ] **Drag-edge auto-scroll** — follow-up to the infinite canvas: auto-pan the canvas when a card is dragged near a viewport edge, so you can fling a card far past the edge in one gesture (today the canvas grows live and you scroll to follow). Isolated to the drag `move` handler.
- [ ] Import / export settings as a JSON file (backup + move between machines)
- [ ] Light-theme support with auto-contrast text (when a light background is chosen)
- [ ] Stocks / crypto ticker widget
- [ ] To-do / notes widget
- [ ] Calendar / agenda widget
- [ ] Keyboard shortcuts (open settings, refresh all, focus search)
- [ ] New-tab support via a lightweight Chrome extension wrapper
