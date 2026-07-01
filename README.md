# 2026 World Cup Knockout Bracket 🏆

A single-file, zero-dependency web app that tracks the **2026 FIFA World Cup knockout stage** (Round of 32 → Final) with auto-updating live scores. Built for mobile, written in Traditional Chinese (Hong Kong), and deployed on GitHub Pages.

**Live site:** https://chi429.github.io/worldcup-bracket/

---

## Overview

The page renders the full knockout bracket as horizontally-swipeable columns — 32強 → 16強 → 8強 → 4強 → 決賽 — plus a top-scorers / assists board. On load it fetches live match data from a free, keyless API and merges it over hand-verified fallback results, so the page is **never blank or broken** even when the API is down.

Key characteristics:

- **Single file.** Everything (HTML, CSS, JS, data) lives in `index.html`. No build step, no framework, no package manager.
- **Keyless live data.** Fetches from `https://worldcup26.ir/get/games` — no API key or auth required.
- **Graceful degradation.** Each match carries a known-good fallback result. If the API is unavailable, slow, or missing a match, the fallback is shown and a status indicator explains why.
- **Mobile-first.** Sticky header, swipe navigation with scroll-snap, tap-to-expand team journeys, and live countdown timers.
- **Timezone-aware.** All kickoff times are displayed in Hong Kong time (港時) regardless of the viewer's location.

---

## Features

| Feature | Description |
|---|---|
| Live score updates | Auto-fetches on page load; manual refresh via the 重新整理 button |
| Status indicator | Green dot = live data loaded; amber dot = showing saved fallback |
| Bracket navigation | Swipe or tap the round tabs (32強 / 16強 / 8強 / 4強 / 決賽 / 射手榜) |
| Focus panel | Highlights in-progress matches and the next 4 upcoming fixtures (by HK time) |
| Match states | Distinct styling for finished, live, today, penalty-shootout, and TBD matches |
| Team journeys | Tap a team name to expand its result history through the tournament |
| Countdown timers | Live "starts in X" counters, refreshed every 30 seconds |
| Scorers & assists | Top-5 goal and assist leaderboards (dated snapshot from HKET) |

---

## Project Structure

```
worldcup-bracket/
├── index.html                      # The entire application (HTML + CSS + JS + data)
├── README.md                       # This file
├── .gitignore                      # Excludes local secrets from version control
└── GIT_PUSH_CREDENTIALS.local.md   # Local-only push credentials (gitignored, never committed)
```

Because the app is a single static file, there is no `src/`, no build output, and no dependency directory. This is intentional — it keeps hosting trivial (any static host works) and the app fully self-contained.

---

## How It Works

### 1. Data model

The bracket is defined as a `MATCHES` array in `index.html`. Each match object looks like:

```js
{
  id: 80,                              // matches the API's match id (73–104)
  ko: "2026-07-01T12:00:00-04:00",     // kickoff time (ISO, venue timezone)
  round: 0,                            // 0=R32, 1=R16, 2=QF, 3=SF, 4=Final/3rd
  label: "7/1 · 阿特蘭大",             // display label (date · city)
  home: { name: "英格蘭", flag: "🏴..." },
  away: { name: "剛果(金)", flag: "🇨🇩" },
  fb:   { today: true },               // fallback: last known-correct result
  next: "➝ 16強第4場 對 墨西哥"        // where the winner advances
}
```

Two lookup tables map the API's numeric team IDs to Traditional Chinese names (`ZH`) and flag emoji (`FLAG`).

### 2. Live merge logic

`resolveMatch()` starts from the fallback, then overrides it with live API values **only when the API has meaningful score/status data**. This prevents an incomplete API response from wiping out a correct saved result. Match status is derived from the API's `finished` and `time_elapsed` fields.

### 3. Rendering

`render()` builds all five round columns plus the leaderboard, `renderFocus()` builds the focus panel, and `tickCd()` updates countdown timers on a 30-second interval. Everything is rendered client-side from the merged data.

### 4. Failure handling

`loadLive()` wraps the fetch in try/catch. On success it renders live data and sets the status to green; on any failure it renders fallback-only and sets the status to amber with an explanatory footer.

---

## Local Development

No tooling required. Because `fetch()` is used, open it through a local server rather than `file://`:

```bash
# from the project folder
python3 -m http.server 8000
# then visit http://localhost:8000
```

Edit `index.html` directly — all markup, styles, logic, and data are in that one file.

### Updating results manually

To change a fallback result, edit the relevant match's `fb` object in the `MATCHES` array:

- `fb: {}` — not yet played (TBD)
- `fb: { today: true }` — scheduled for today
- `fb: { hs: 2, as: 1, done: true, winner: "home" }` — finished result
- add `pk: "PK 3:4 · ..."` for penalty-shootout notes

---

## Deployment

The app is hosted on **GitHub Pages** from the `main` branch. Any push to `main` republishes the site automatically (typically within a minute).

```
Repo:  https://github.com/chi429/worldcup-bracket
Pages: https://chi429.github.io/worldcup-bracket/
```

To publish a change: edit `index.html`, commit, and push to `main`. See `GIT_PUSH_CREDENTIALS.local.md` for the exact push workflow (that file is gitignored and must never be committed).

> **Note on caching:** GitHub Pages and browsers cache aggressively. After a push, hard-refresh (`Cmd/Ctrl+Shift+R`) or use a private window to see changes.

---

## Known Limitations

- **Emoji flags render per-device.** Subdivision flags such as England (🏴󠁧󠁢󠁥󠁮󠁧󠁿) fall back to a plain black flag on some older systems that lack the glyph. If reliable flags across all devices are required, switch to SVG/PNG images.
- **Third-party API.** Live data depends on `worldcup26.ir` remaining available; the fallback layer covers outages but won't reflect brand-new results until the API returns or the fallback is edited.
- **Leaderboard is a manual snapshot.** Scorer/assist figures are hand-entered from HKET and dated via `SCORER_ASOF`, not live.

---

## Tech Stack

Plain HTML5, CSS3 (custom properties, flexbox, scroll-snap), and vanilla JavaScript (`fetch`, `Intl.DateTimeFormat`). No dependencies, no build step, no backend.
