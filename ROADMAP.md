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
- [x] README

---

## 🛠️ In Progress

- [ ] Final watermark artwork (being refined in Photoshop) — drop-in at `img/aerie-watermark.png`

---

## ⏭️ Planned (next up)

### Repo housekeeping
- [ ] Add hero screenshot to the README (`img/aerie-screenshot.png`)
- [ ] Add a `LICENSE` file (MIT or your choice) — README already references it

### Features
- [ ] Custom-model text entry in the per-feed AI selector (type any model name, not just the curated list)
- [ ] Thumbnails in the classic (no-key) feed list
- [ ] Optional rss2json API key field (raise the ~10-item cap per feed)
- [ ] "Feeds last updated" / next-refresh indicator
- [ ] Import / export settings as a JSON file (backup + move between machines)

---

## 💡 Ideas / Backlog

- [ ] Light-theme support with auto-contrast text (when a light background is chosen)
- [ ] Stocks / crypto ticker widget
- [ ] To-do / notes widget
- [ ] Calendar / agenda widget
- [ ] Per-widget settings (e.g. feed item count override, weather detail level)
- [ ] Keyboard shortcuts (open settings, refresh all, focus search)
- [ ] New-tab support via a lightweight Chrome extension wrapper

---

## 📝 To triage (your additions)

_Drop new items here and we'll slot them into the sections above as we go._

- [ ] …
- [ ] …
- [ ] …
