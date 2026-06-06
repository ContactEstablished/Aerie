<p align="center">
  <img src="img/aerie-wordmark.png" alt="Aerie" width="280">
</p>

<p align="center">
  <em>Your browser's home base — live weather, AI‑summarized headlines, and a daily photo, on a grid you arrange yourself.</em>
</p>

<p align="center">
  <img alt="single file" src="https://img.shields.io/badge/build-none%20·%20single%20file-5b8cff">
  <img alt="vanilla js" src="https://img.shields.io/badge/stack-vanilla%20HTML%2FCSS%2FJS-7b6bff">
  <img alt="no framework" src="https://img.shields.io/badge/dependencies-0-22c55e">
  <img alt="license" src="https://img.shields.io/badge/license-MIT-06b6d4">
</p>

<!-- ─────────────────────────────────────────────────────────────
     SCREENSHOT GOES HERE
     Drop a screenshot at img/aerie-screenshot.png (or change the path)
     and uncomment the block below.
────────────────────────────────────────────────────────────── -->
<!--
<p align="center">
  <img src="img/aerie-screenshot.png" alt="Aerie dashboard" width="960">
</p>
-->

---

## Table of Contents

- [What is Aerie?](#what-is-aerie)
- [Highlights](#highlights)
- [Features in Detail](#features-in-detail)
- [Quick Start](#quick-start)
- [Make It Your Chrome Home Page](#make-it-your-chrome-home-page)
- [The Settings Panel](#the-settings-panel)
- [Enabling AI Summaries & Ranking](#enabling-ai-summaries--ranking)
- [Arranging the Grid](#arranging-the-grid)
- [Theming](#theming)
- [How It Works](#how-it-works)
- [Data Sources](#data-sources)
- [Caching & Local Storage](#caching--local-storage)
- [Privacy & Security](#privacy--security)
- [Browser Support](#browser-support)
- [Troubleshooting](#troubleshooting)
- [Project Structure](#project-structure)
- [Customizing & Hacking](#customizing--hacking)
- [Roadmap Ideas](#roadmap-ideas)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## What is Aerie?

**Aerie** is a personal start page — the screen your browser opens to. The name comes from an *aerie*: the high cliff‑top nest of a bird of prey, a vantage point for surveying everything below. That's the idea: one glance and you've taken in the day — the weather where you are, the top headlines (optionally distilled by AI into a single sentence each), and a striking photo to wake up to.

It is a **single, self‑contained HTML file**. No build step, no framework, no server, no `npm install`. All the CSS and JavaScript live inline in `home.html`; the only other files are the branding images in `img/`. Open it from disk and it just runs.

Everything is **configurable from a Settings panel** and **persists in your browser** — your layout, feeds, units, colors, and API key are remembered between sessions. The layout itself is a **drag‑and‑drop, resize‑to‑snap grid**, so you can shape the page to your taste.

> **Why a single file?** Because a home page should be trivial to install, trivial to back up, and impossible to break with a dependency update. Copy one file (plus the `img/` folder) and you're done.

---

## Highlights

- 🦅 **One‑glance dashboard** — weather, news, and a daily photo in a single view.
- 🧩 **Drag‑and‑drop grid** — move widgets by their header, resize from the corner; everything **snaps to an invisible grid**.
- 📰 **Topic‑based news feed** — tell Aerie the **topics** you care about (pills you add/remove); it searches **Google News** for each and builds one merged feed. **👍/👎/💾** on every story: like it, dismiss it, or save it to your **Reading List**.
- 🤖 **AI‑summarized & personalized (optional)** — bring your own **DeepSeek, OpenAI, or Anthropic** key for a one‑sentence summary per story, and **vote‑based re‑ranking** that surfaces what you've liked. Pick the **provider and model**. Without a key, the feed still shows headlines + working votes/save.
- 🌤️ **Local weather, no API key** — auto‑locates you by IP (or set a city / use GPS), powered by Open‑Meteo. Add **multiple cards for multiple cities**.
- 🖼️ **Photo of the Day** — Fstoppers' editor‑picked photo by default, plus **NASA's Astronomy Picture of the Day** or a random masterpiece from the Art Institute of Chicago or The Met. **Shuffle** through recent days, or reset to today.
- 🎨 **Fully themeable** — independent **accent** and **background** color pickers, themed scrollbars, and a subtle layered gradient + watermark for depth.
- ⏱️ **Smart caching** — the news feed refreshes on your chosen interval (and accumulates), the photo caches for the day, weather shows the last‑known reading instantly while it refreshes.
- 🔒 **Local‑only & private** — your settings and API key never leave your browser except to call the services you've configured directly.
- 🪶 **Zero dependencies** — pure HTML/CSS/JS, ~1 file.

---

## Features in Detail

### Weather
- Powered by **[Open‑Meteo](https://open-meteo.com/)** — free, no API key.
- **Multiple weather cards** — add a card per city. Use **＋ Add weather** in Settings, then set each card's location independently from its **⚙** button.
- Each card resolves its **location** one of three ways:
  - **Auto (by IP)** — default; no permission prompt. Tries `ipwho.is`, then `geojs.io`, then `ipapi.co` for resilience.
  - **City** — type a city; geocoded via Open‑Meteo's geocoding API. City cards auto‑title to the city name.
  - **Precise (GPS)** — uses the browser's geolocation (asks permission).
- Shows current temperature, condition (with emoji), "feels like," daily high/low, humidity, and wind.
- **Units** (Fahrenheit · mph, or Celsius · km/h) are a single global setting applied to every card.
- The **last successful reading is cached per card** and shown instantly on load (marked *offline* if a refresh fails), so you're never staring at a spinner.

### News Feed (topic search) & Reading List
Instead of managing individual RSS feeds, you tell Aerie the **topics** you want to follow and it builds a single, personalized feed. Two side‑by‑side widgets:

- **News Feed** (left) — add **topics** as pills in **Settings → News** (type a word/phrase, press Enter; ✕ to remove). For each topic Aerie searches **Google News** (its per‑query RSS endpoint) and takes the top **N** stories (default 5), merging them into one feed.
- **Reading List** (right) — articles you **💾 save** from the feed, to read later.

Each story card has three actions:
- **👍 like** — records that you like it (the story stays; feeds future ranking).
- **👎 not interested** — hides it immediately and stops it being recommended again.
- **💾 save** — copies it to the **Reading List** (the card stays, marked *Saved ✓*).

In the Reading List, click a story to **open it in a new tab**, or hit **✓ Read** to remove it.

**How it stays fresh:**
- The feed **accumulates** — each refresh *adds* newly‑found stories on top and keeps the ones already there, dropping anything older than **2 days** (or that you disliked/saved).
- A **48‑hour dedupe ledger** means the same article isn't grabbed twice within two days.
- **Auto‑refresh** every **Off / 15 / 30 / 60 min** (your choice); **⟳** fetches new stories onto the feed on demand.
- Fetched through **[rss2json](https://rss2json.com/)** (CORS‑friendly, no key) with a **CORS‑proxy fallback** (`allorigins` → `codetabs`). Add an optional **rss2json API key** to lift the ~10‑items‑per‑request free‑tier cap.

> **On images:** Google News article links are redirect URLs that can't be resolved quickly from the browser, so cards are mostly **text‑first** (headline + summary + source); a thumbnail appears only when the RSS item carries one.

### AI: summaries & vote‑based ranking (optional — DeepSeek, OpenAI, or Anthropic)
Add an API key for one of **[DeepSeek](https://platform.deepseek.com/)**, **[OpenAI](https://platform.openai.com/)**, or **[Anthropic](https://console.anthropic.com/)** (Settings → **AI**), then pick the **provider + model** for the feed in Settings → **News** (curated models per provider plus a **✎ Custom…** free‑text option). With a key, on each refresh Aerie:
1. Sends the newly‑found stories in **one batched call** for a **single factual sentence** each (built from the headline + Google News's related‑coverage blurb — articles aren't individually fetched).
2. **Re‑ranks** that batch with a second call that weighs your **👍/👎 history**, so stories resembling what you've liked lead and ones resembling dislikes sink. Ranking never drops a story and fails safe to the original order.

All providers are called **directly from your browser** (no backend); Anthropic uses its direct‑browser‑access header. **Without a key**, the feed degrades gracefully: headlines, sources, timestamps, and the 👍/👎/💾 buttons all still work — just no summaries or ranking. The feed caches by **topics + count + provider + model** and rebuilds when any of those change.

### Photo of the Day
- **Fstoppers — Photo of the Day** (default): the editors' daily pick, via its RSS feed.
- **NASA — Astronomy Picture of the Day**: the day's APOD via NASA's official JSON API (the ~weekly video days are skipped).
- **Art Institute of Chicago** or **The Met**: a random public‑domain artwork each time.
- **Shuffle 🔀 / Today 📅** — the **🔀** button in the card header swaps in a random photo from the source's recent days (the last ~14 for Fstoppers and APOD; a fresh random for the museums). **📅** appears after a shuffle to jump back to today's pick. Shuffles are *transient* — they never overwrite the day's cache, so a reload always returns to today's photo.
- **Cached for the whole local day** — it loads once and stays put until tomorrow. The ⟳ button forces a fresh one on demand.

### Clock & Search
- **Clock** widget: large time, full date, and a time‑of‑day greeting. Per‑card **⚙** options (analog/digital, seconds, 24‑hour, time zone), and you can add multiple clocks for different zones.
- **Time‑zone skylines** — pick a city zone and a faint line‑art skyline of that city sits in the **bottom‑right** of the card, fading toward the readout, so a wall of clocks reads at a glance. Shown on both digital and analog faces; "Local" stays clean (no skyline).
- **Search** widget (optional): a search bar with a built‑in **engine selector** — Google, Bing, or Yahoo — plus a **custom search‑URL** option (e.g. Dogpile) set from the card's **⚙**. Fits cleanly in a single‑row card.
- Both can be toggled on/off in Settings.

### The Grid
- A **12 × 9 snap grid** fills the viewport.
- **Move** a widget by dragging its **header**; **resize** by dragging the **bottom‑right corner**. Both snap to grid cells in real time, and the faint grid lines brighten while you drag.
- The layout is **saved automatically** and restored on next load.

### Theming & Branding
- **Accent color**: 8 presets + a custom picker. Recolors highlights, buttons, scrollbars, the background glow, and the feed accent bar.
- **Background color**: a separate base‑tone picker. A subtle **gradient fade** and the **accent glow** are layered on top so the background keeps depth rather than going flat.
- **Themed scrollbars** that match your accent.
- **Aerie branding** baked in: favicon, toolbar logo + wordmark, and a faint line‑art **watermark** centered behind the widgets.

---

## Quick Start

1. **Get the files.** Clone or download this repo:
   ```bash
   git clone https://github.com/<your-username>/aerie.git
   ```
   You only need `home.html` and the `img/` folder living next to each other.

2. **Open it.** Double‑click `home.html`, or open it in Chrome via `File → Open`. It loads from a `file://` URL and works immediately — no server required.

3. *(Optional)* **Set it as your home page** so it opens with every new window. See the next section.

> **Tip:** Because Aerie is just a local file, you can keep it anywhere — a Dropbox folder, a USB stick, wherever. Keep `home.html` and `img/` together and it'll work.

---

## Make It Your Chrome Home Page

Chrome can open Aerie as your **Home button** target and as your **startup page**. (Replacing the *New Tab* page specifically requires an extension — that's a Chrome restriction, not an Aerie one.)

1. Copy the file's full path, formatted as a URL. On Windows it looks like:
   ```
   file:///C:/path/to/aerie/home.html
   ```
2. Chrome → **⋮ → Settings → On startup → Open a specific page or set of pages → Add a new page** → paste the URL above.
3. *(Optional)* **Settings → Appearance → Show home button**, then enter the same URL so the 🏠 button opens Aerie too.

To preview without changing settings, just open the file directly:
```bash
# Windows
start chrome "file:///C:/path/to/aerie/home.html"
```

### Optional: serve it locally
Everything works from `file://`, but if you prefer an `http://` origin (some setups treat it more permissively), serve the folder with any static server:
```bash
# Python
python -m http.server 8777
# then visit http://localhost:8777/home.html
```

---

## The Settings Panel

Click **⚙ Settings** (top‑right) to open the configuration modal. Changes preview live where applicable and are saved with **Save** (or discarded with **Cancel**).

| Section | What you can configure |
| --- | --- |
| **Weather** | Units (°F·mph or °C·km/h), show/hide, and **＋ Add weather** for more cities. Each card's location (Auto‑IP / City / GPS) is set from its **⚙**. |
| **News** | Manage **topics** (pills); set **articles per topic**, **refresh interval**, and the feed's **provider + model** (with a **✎ Custom…** model option); optional **rss2json key**; **reset feed / clear votes / clear Reading List**. |
| **AI Settings** | Enable AI; paste/clear a **DeepSeek / OpenAI / Anthropic** key. (The news feed's provider & model live in the **News** tab.) |
| **Photo of the Day** | Source: Fstoppers POTD, NASA APOD, Art Institute of Chicago, or The Met. |
| **Accent Color** | 8 preset swatches + custom color picker. |
| **Background Color** | 8 preset base tones + custom color picker (gradient + glow preserved). |
| **Widgets** | Toggle Weather, Photo of the Day, Clock, and the Search bar. |
| **Advanced** | Swap the CORS proxy; **Reset layout**; **Reset everything**. |

Per‑widget controls live in each widget's header: **⟳** refreshes (and busts that widget's cache), and **✕ / –** removes a feed or hides a utility widget.

---

## Enabling AI Summaries & Ranking

AI is **optional** — without a key the news feed still shows headlines, sources, timestamps, and the 👍/👎/💾 buttons. Aerie supports three providers, all called directly from the browser:

| Provider | Get a key | Default model offered |
| --- | --- | --- |
| **DeepSeek** | [platform.deepseek.com](https://platform.deepseek.com/) | `deepseek-chat` |
| **OpenAI** | [platform.openai.com](https://platform.openai.com/) | `gpt-4o-mini` |
| **Anthropic** | [console.anthropic.com](https://console.anthropic.com/) | `claude-haiku-4-5` |

1. In Aerie: **⚙ Settings → AI Settings** → check **"Enhance the news feed with AI"** → paste a key into **one** of the DeepSeek / OpenAI / Anthropic fields.
2. In **Settings → News**, set the feed's **Provider** and **Model** (choose *Off* to keep plain headlines), plus **articles per topic** and the **refresh interval**.
3. **Save.** On the next refresh the feed gains one‑line summaries and vote‑based ranking.

**Cost & rate:** each refresh makes at most **two** calls — one to summarize the newly‑found stories, one to re‑rank them by your votes — and only when there are new stories, throttled by the refresh interval. The default models (`gpt-4o-mini`, Claude Haiku, `deepseek-chat`) are the cheap/fast tier. Clearing the key drops the feed back to headlines.

> **Note on OpenAI errors:** OpenAI doesn't attach CORS headers to its `401` responses, so an *invalid* OpenAI key shows up as a generic "failed to fetch" in that card rather than a clean error message. A valid key works normally. (DeepSeek and Anthropic return clean error messages.)

---

## Arranging the Grid

- **Move:** click and drag a widget's **title bar** to a new spot — it snaps to grid cells.
- **Resize:** drag the **bottom‑right corner** of a widget in or out — it snaps to cells.
- **Add topics:** **Settings → News** — add the topics you want to follow as pills.
- **Hide/remove:** use the **–/✕** button in a widget header, or the **Widgets** toggles in Settings.
- **Start over:** **Settings → Advanced → Reset layout** restores the default arrangement (keeping your feeds and preferences).

Your arrangement is saved automatically.

---

## Theming

Aerie ships with a dark, glassy aesthetic and two independent color controls:

- **Accent** drives interactive color — buttons, links, scrollbars, the background glow, and the red‑style feed accent bar.
- **Background** sets the base tone. Aerie derives a slightly lighter shade for a **diagonal gradient fade** and keeps the **accent radial glow** layered on top, so the page has depth instead of a flat fill.

Both offer eight curated presets plus a full custom color picker, and both preview live before you save.

> **Note:** The UI is tuned for **dark** backgrounds (light text on translucent dark cards). The presets keep you in safe territory; a very light custom background may reduce text/grid‑line contrast.

---

## How It Works

- **One file.** `home.html` contains all markup, an inline `<style>`, and an inline `<script>`. No bundler, no external JS/CSS.
- **State** lives in a single object persisted to `localStorage`, with a small **schema‑migration** routine (`__v`) that upgrades older saved layouts in place when you update Aerie.
- **The grid** is a CSS Grid (`12 × 9`). Each widget stores `{col, row, w, h}`; dragging/resizing recomputes those from pointer position against the grid's measured cell size and re‑applies `grid-column` / `grid-row`. Faint grid lines are painted with layered CSS gradients.
- **Networking** is plain `fetch`:
  - Services that allow cross‑origin requests (Open‑Meteo, rss2json, the AI providers, the museums, IP‑geo services) are called **directly**.
  - Feeds/pages that don't are routed through a **public CORS proxy** (`allorigins`, with `codetabs` as fallback). A small global limiter caps simultaneous proxy requests, and a retry‑with‑backoff smooths transient rate limits.
- **News feed** searches Google News per topic (rss2json/proxy) → merges + dedupes against a 48h ledger → one batched call to the chosen provider for summaries (DeepSeek/OpenAI via chat‑completions, Anthropic via the Messages API) → a second call re‑ranks by your votes → accumulates into the cached feed.
- **Resilience first:** every data path has a fallback and shows cached or partial data rather than failing hard.

---

## Data Sources

All free and key‑less, **except the AI providers** (optional, your key):

| Purpose | Service | Key required |
| --- | --- | --- |
| Weather | [Open‑Meteo](https://open-meteo.com/) (forecast + geocoding) | No |
| IP geolocation | `ipwho.is` → `geojs.io` → `ipapi.co` | No |
| Reverse geocoding | [BigDataCloud](https://www.bigdatacloud.com/) | No |
| News search | [Google News](https://news.google.com/) RSS search (per topic) | No |
| RSS → JSON (primary) | [rss2json](https://rss2json.com/) | No |
| CORS proxy (fallback) | `api.allorigins.win` → `api.codetabs.com` | No |
| AI summaries | [DeepSeek](https://platform.deepseek.com/) · [OpenAI](https://platform.openai.com/) · [Anthropic](https://console.anthropic.com/) | **Yes (optional)** |
| Photo of the Day | [Fstoppers POTD](https://fstoppers.com/potd) · [NASA APOD](https://apod.nasa.gov/apod/astropix.html) | No (APOD uses NASA's public `DEMO_KEY`) |
| Artwork | [Art Institute of Chicago](https://api.artic.edu/docs/) · [The Met](https://metmuseum.github.io/) | No |
| Search | Google · Bing · Yahoo · custom URL | No |

> **rss2json note:** The free tier returns up to ~10 items per request and does **not** support the `count` parameter (that needs a paid key). Aerie omits `count` unless you add an **rss2json API key** (News tab) — with a key it sends `count` to raise the cap; without one it stays on the safe free‑tier path. If a request needs more items or rss2json can't read it, Aerie falls back to the CORS proxy.

---

## Caching & Local Storage

Aerie stores everything client‑side. Keys used in `localStorage`:

| Key | Contents | Refresh policy |
| --- | --- | --- |
| `myhome.v1` | Main state: layout, widgets (incl. the News Feed's topics + provider/model + refresh), per‑weather‑card location, units, theme, AI settings (incl. provider API keys), proxy, optional rss2json key. Versioned via `__v` (currently **16**). | On every change |
| `myhome.news.cache` | The built News Feed (accumulated stories), keyed by topics + count + provider + model. | Older than the feed's refresh interval; **⟳** fetches new |
| `myhome.news.votes` | Your 👍/👎 per article (with title/source/topic) — drives re‑ranking. | On each vote |
| `myhome.news.saved` | The Reading List (saved articles). | On save / mark‑read |
| `myhome.news.seen` | 48‑hour dedupe ledger so a story isn't grabbed twice within 2 days. | Pruned at 48h each build |
| `myhome.feeds` | Legacy per‑feed AI cache (unused by the topic feed; kept for migration). | Older than the AI **cache minutes** (default 30) |
| `myhome.wx` | Last successful weather reading per card (keyed by widget id). | On each successful fetch |
| `myhome.potd` | The day's photo (per source). | New local day, or manual ⟳ |

To wipe everything and start fresh: **Settings → Advanced → Reset everything** (or clear the site's storage in DevTools).

---

## Privacy & Security

- **Local‑first.** Your layout, preferences, and any provider API keys live only in this browser's `localStorage`.
- **Direct calls only.** Aerie talks to the services listed above directly from your browser. Nothing is sent to any Aerie‑owned backend — there isn't one.
- **Your API keys** are each sent only to their own provider (`api.deepseek.com`, `api.openai.com`, `api.anthropic.com`), only when a feed using that provider refreshes.
- **Heads‑up:** since keys are stored with your settings, **don't share your `home.html`/exported storage** after entering a key. Use a provider's **Clear** button before sharing.
- **CORS proxies** are third‑party public services; feed URLs (and article page requests) pass through them. They are **not** sent your API keys. You can change the proxy in Settings.

---

## Browser Support

- Built and tested for **Google Chrome** (recent versions) on Windows.
- Uses modern web features (CSS Grid, custom properties, Pointer Events, `backdrop-filter`, optional chaining, `??=`). Any current Chromium‑based browser (Edge, Brave) should work; Firefox/Safari are largely fine but untested.
- `file://` is treated as a secure context in Chrome, so geolocation and `fetch` work from a local file.

---

## Troubleshooting

**A feed won't load.**
Aerie falls back from rss2json to a CORS proxy automatically, but public proxies do get rate‑limited. Try the feed's ⟳ button, or switch the proxy in **Settings → Advanced**. Confirm the feed URL is a valid RSS/Atom link.

**Everything fails behind a VPN.**
Some VPN exit nodes are blocked by the public proxy/RSS/IP services. Toggle the VPN off, or switch to a different server, and reload.

**Weather is blank or wrong.**
The IP‑based location can be imprecise. Set a **City** (or enable **Precise/GPS**) in **Settings → Weather & Location**. If all IP providers are momentarily down, Aerie shows the last cached reading.

**Summaries / ranking aren't appearing.**
Check that **"Enhance the news feed with AI"** is on (AI Settings tab), the feed has a **Provider** selected (not *Off*) in the **News** tab, and that provider's **key** is filled in. A bad key surfaces an error (for OpenAI specifically, an invalid key reads as a generic "failed to fetch"). The feed caches for the refresh interval — hit **⟳** to fetch new stories and rebuild.

**Console shows `429 / Too Many Requests` from a proxy.**
That's a public proxy throttling under load (worse on a cold load with many feeds, or if a feed has a blank URL). It usually self‑resolves on the next refresh; the rss2json primary path avoids the proxy for most feeds.

**The photo doesn't change on reload.**
That's intended — Photo of the Day is cached for the day. Use the widget's ⟳ for a new one.

---

## Project Structure

```
aerie/
├─ home.html               # the entire app (HTML + inline CSS + inline JS)
├─ README.md
└─ img/
   ├─ aerie-logo.png       # original square app icon (opaque)
   ├─ aerie.png            # original wordmark (opaque)
   ├─ aerie-bg-img.png     # original line-art (for the watermark)
   ├─ aerie-icon.png       # processed: transparent app icon  → favicon + toolbar
   ├─ aerie-wordmark.png   # processed: transparent "aerie" wordmark → toolbar/README
   ├─ aerie-watermark.png  # processed: transparent light line-art → background watermark
   ├─ time zone-background.png  # source sheet: 4×5 grid of 17 city tiles (skyline + label)
   └─ tz-skylines.png      # processed: skyline-only, navy→transparent → clock time-zone backdrops
```

---

## Customizing & Hacking

Because it's one file, tweaking is direct — open `home.html` and search for these:

- **Default layout & widgets:** `defaultState()` — grid size, default widgets (incl. the News Feed's seed topics), and starting positions.
- **Watermark strength:** the `.watermark img { opacity }` rule.
- **Grid line subtlety:** the `--line` CSS variable.
- **Color presets:** `PRESET_ACCENTS` and `PRESET_BGS` arrays.
- **CORS proxies:** the `PROXIES` array.
- **News engine:** `buildNewsFeed()` (topic search + accumulate + dedupe), `googleNewsUrl()`, `aiSummarize()` / `rankByTaste()` / `RANK_SYS`, and the `AI_PROVIDERS` / `MODEL_OPTIONS` / `DEFAULT_MODELS` constants (add models or providers here).
- **Branding:** swap the files in `img/` (keep the same names, or update the `<link>`/`<img>` references near the top of the file).

---

## Roadmap

Planned work and ideas live in **[ROADMAP.md](ROADMAP.md)** — what's done, what's in progress, and what's on deck.

---

## License

Released under the **MIT License** — see [`LICENSE`](LICENSE). (Swap this out if you prefer a different license.)

---

## Acknowledgments

- Weather by **[Open‑Meteo](https://open-meteo.com/)**.
- Feed parsing by **[rss2json](https://rss2json.com/)**; CORS proxying by **[AllOrigins](https://allorigins.win/)** and **[CodeTabs](https://codetabs.com/)**.
- AI summaries by **[DeepSeek](https://www.deepseek.com/)**, **[OpenAI](https://openai.com/)**, and **[Anthropic](https://www.anthropic.com/)**.
- Photography from **[Fstoppers](https://fstoppers.com/)** and **[NASA APOD](https://apod.nasa.gov/)**; artwork from the **[Art Institute of Chicago](https://www.artic.edu/)** and **[The Metropolitan Museum of Art](https://www.metmuseum.org/)**.
- Geolocation by **ipwho.is**, **geojs.io**, **ipapi.co**, and **[BigDataCloud](https://www.bigdatacloud.com/)**.

<p align="center"><sub>Built as a single HTML file, with no dependencies. 🦅</sub></p>
