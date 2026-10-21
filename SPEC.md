# KeBox Sentiment tab: data feed spec

The **Property Fear & Greed Index** is a daily sentiment reading for the Australian residential property market. It's built from that day's property news and the weekend auction clearance rate. This repo publishes it as one JSON file for the KeBox **Sentiment** tab.

## The feed

| | |
|---|---|
| File | `sentiment.json` in the root of this repo |
| URL to fetch | `https://raw.githubusercontent.com/<GITHUB-USER>/<REPO>/main/sentiment.json` |
| Updates | Weekdays at about 9:50am Brisbane time (UTC+10). The index holds over weekends and public holidays. |
| Auth | None. It's a public file, so the app needs no API key to read it. |
| Owner | Updated automatically by Claude every weekday. Don't edit it by hand. |

**How the app should use it**
- Fetch it fresh each time the Sentiment tab opens, and on pull-to-refresh. Don't cache it for longer than an hour.
- Never hard-code or copy the data into the app. Always read the live file.
- If the fetch fails, show the last successfully loaded copy with its `asOf` date and a small "couldn't refresh" note.
- Check `schemaVersion`. It's `1` now. New fields may be added at any time, and the app should ignore any it doesn't know. Fields are never renamed or removed without bumping `schemaVersion`.

## Fields

The values below are examples. See `sentiment.json` for real data.

```jsonc
{
  "schemaVersion": 1,
  "name": "Property Fear & Greed Index",
  "region": "Australia",
  "generatedAt": "2026-10-21T12:40:00Z",   // when the file was built (UTC)
  "asOf": "2026-10-21",                     // the market day the reading is for (Brisbane date)

  "index": {
    "value": 27,                            // 0–100, the headline number
    "exact": 26.81,                         // unrounded, for charts
    "band": { "label": "Fear", "key": "fear" },
    "previous": 28,                         // previous recorded day's value
    "change": -1                            // value − previous
  },

  "today": {                                // the day's news on its own, before smoothing
    "date": "2026-10-21",
    "negative": 7, "positive": 2,           // article counts
    "newsScore": -26,                       // sum of article points
    "newsIndex": 24                         // 50 + newsScore, clamped 0–100 (null if no articles)
  },

  "averages": { "month": 33, "year": 40 }, // average index this calendar month / year

  "clearance": {
    "latest": {                             // most recent weekend auction result (null if none yet)
      "reportedOn": "2026-10-19",
      "weekend": "17–18 Oct",
      "capitals": 51.2,                     // combined capital cities clearance rate, %
      "brisbane": 24.0,                     // Brisbane clearance rate, % (can be null)
      "changeCapitals": 2.4,                // vs previous recorded weekend, percentage points
      "changeBrisbane": 1.1,                // (missing if either week has no Brisbane figure)
      "kind": "preliminary",                // "preliminary" | "final"
      "source": "Cotality via …",
      "url": "https://…"
    },
    "history": [ /* same shape as latest, minus the change fields, oldest first */ ]
  },

  "headlines": [                            // the day's scored articles, most negative first
    {
      "title": "…", "source": "…", "url": "https://…",
      "summary": "One line on why it matters to buyers and sellers",
      "sentiment": "neg",                   // "neg" | "pos"
      "impact": "normal",                   // "major" | "normal" | "minor"
      "points": -6,                         // what it added to newsScore
      "geo": { "level": "national" }        // or { level:"city", state:"QLD", city:"Brisbane" } etc.
    }
  ],

  "history": [                              // every recorded day, oldest first
    { "date": "2026-07-01", "index": 47, "band": "neutral", "newsIndex": 41, "newsScore": -9 }
  ],

  "bands": [                                // ranges are inclusive, on the rounded value
    { "key": "extreme_fear",  "label": "Extreme Fear",  "min": 0,  "max": 20 },
    { "key": "fear",          "label": "Fear",          "min": 21, "max": 39 },
    { "key": "neutral",       "label": "Neutral",       "min": 40, "max": 60 },
    { "key": "greed",         "label": "Greed",         "min": 61, "max": 79 },
    { "key": "extreme_greed", "label": "Extreme Greed", "min": 80, "max": 100 }
  ],

  "links": { "fullTracker": "https://claude.ai/artifact/8CeMPRjb5qGmkNDkrKFhUm" },
  "disclaimer": "Based on media headlines and property market reporting. It is not a market forecast, valuation or financial advice."
}
```

A real sample built from live data is in this repo as `sentiment.json`.

## What the Sentiment tab should show

Build it natively in KeBox's own style. The "Portfolio Pulse" card is the reference.

1. **Gauge card (top).** A small eyebrow label "Market pulse". The band label as the title (e.g. "Fear"). `index.value` in a "/ 100" badge. A horizontal bar running red → yellow → green with a ring marker at `index.value`%, labelled FEAR on the left and GREED on the right. Under it, one line: "Down 1 from 28" (from `change` / `previous`). Then a small muted line: "7 negative · 2 positive · News Index 24".
2. **Trend chart.** A line of `history[].index` with the fear/greed bands shaded behind it. Offer 1M / 3M / All ranges. Tapping a point shows that day's value and band.
3. **Clearance rates.** The latest capitals and Brisbane figures with their change arrows and the weekend label. Add a small line chart of `clearance.history` (capitals and Brisbane as two lines) with 55% / 65% guide lines (under 55% = buyer's market, over 65% = seller's market).
4. **Today's headlines.** A list from `headlines`: points (red if negative, green if positive), title, source and summary. Tapping opens `url` in the in-app browser.
5. **Footer.** The `disclaimer` text, **always visible**, and "Updated {asOf}". Optionally add a "See full tracker" link to `links.fullTracker`.

Band colours to match the tracker: extreme fear `#D8434F`, fear `#EE6A6A`, neutral `#EFB95B`, greed `#8FE07F`, extreme greed `#3FCF8E`.

## How the index is calculated (for the "How it works" screen)

- Each article scores **impact × sentiment**: Major ×3, Normal ×2, Minor ×1; negative −3, positive +2. Bad news carries more weight than good.
- **News Index** = 50 + the day's total, kept between 0 and 100.
- On days with a new weekend clearance result, it's blended in at 20%: clearance reading = (rate − 40) × 2.5, so 60% clearance = neutral.
- **Fear & Greed Index** = 70% × the previous day's index + 30% × today's reading. It rolls over from day to day rather than resetting, so it moves with sustained news, not one-off headlines.

The app should **display** these numbers, not recalculate them. The feed is the single source of truth.

## Questions or changes

Need a field added or changed? Open a **GitHub issue** in this repo describing what and why. Mick will have Claude pick it up and update the feed and this spec.
