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
- [x] **Clock time-zone backgrounds** — a faint per-city skyline in the bottom-right of each clock, fading toward the readout, so its location reads at a glance. Skylines are sliced from the source sheet, navy baked to transparent, and repacked into a clean even sprite (`img/tz-skylines.png`); shown on digital + analog, skipped for "Local"
- [x] README

---

## 🛠️ In Progress

- [ ] Final watermark artwork (being refined in Photoshop) — drop-in at `img/aerie-watermark.png`

---

## ⏭️ Planned (next up)

### Feeds
- [ ] Thumbnails in the classic (no-key) feed list (the image is already fetched per item — just surface it)

### Repo housekeeping
- [ ] Add hero screenshot to the README (`img/aerie-screenshot.png`)
- [ ] Add a `LICENSE` file (MIT or your choice) — README already references it

---

## 💡 Ideas / Backlog

### Infinite canvas + minimizable cards
_Pack many windows close together and toggle them, instead of one very tall start page._
- [ ] **Infinite / scrolling canvas** — today the viewport's width & height cap the usable area and how many widgets fit; instead, let the canvas **scroll vertically *and* horizontally**, growing on demand as cards are placed past the browser edges.
- [ ] **Minimize button per card** — collapse a card to just its header bar (hide the body content); click again to expand.
- [ ] **Expand → bring to front** — re-expanding a card (any time its body becomes visible again) immediately gives it the **top z-index**, so tightly-overlapping windows can be toggled open/closed cleanly.
  - _Notes: the layout is currently a viewport-filling 12×9 CSS grid (`body{overflow:hidden}`, `clamp()`-bounded drag/resize). This shifts to a scrollable canvas whose grid grows past the viewport — `firstFreeSpot`, the drag/resize clamps, and "Reset layout" all assume fixed bounds and would need to follow it. Minimize = a persisted per-card `collapsed` flag (render header only); bring-to-front = a rising z-index counter in state so stacking order survives reload._

- [ ] Import / export settings as a JSON file (backup + move between machines)
- [ ] Light-theme support with auto-contrast text (when a light background is chosen)
- [ ] Stocks / crypto ticker widget
- [ ] To-do / notes widget
- [ ] Calendar / agenda widget
- [ ] Keyboard shortcuts (open settings, refresh all, focus search)
- [ ] New-tab support via a lightweight Chrome extension wrapper
