# Morning Walk Evaluation

A mobile-first tool for scoring grocery department conditions during morning store walks. Tracks endcap facing, aisle facing, metro/side stack displays, and misc deductions with per-miss notes to identify repeat problem areas and patterns by closer.

Built as a single HTML file powered by React (CDN) with a Google Sheets backend via Apps Script.

## How It Works

**Frontend:** Static `index.html` served from GitHub Pages. React 18 + Babel loaded from CDN at runtime. No build step, no dependencies to install.

**Backend:** Google Apps Script `doPost()`/`doGet()` web app attached to a Google Sheet. The sheet acts as the database. All data lives in a single `WalkData` tab.

## Scoring

| Section | Base Points | Method |
|---|---|---|
| Endcaps | 18 | 18 minus missed count |
| Aisles | 12 | 12 minus missed count |
| Metro & Side Stacks | 0 | Negative deduction from total |
| Misc (x3) | 0 | −1 each when marked |

**Base max = 30.** Overall score = endcap score + aisle score − metro deductions − misc deductions.

Each miss entry has an optional note field for identifying which specific endcap, aisle, or display was missed. These notes aggregate on the dashboard to surface repeat offenders.

## Features

- **New Walk tab:** Date, who closed, per-miss notes with add/remove, misc toggle deductions, live running score
- **History tab:** Reverse-chron list, tap for full detail view with edit/delete
- **Dashboard tab:** Filter by All / Day / Range / Week / Month
  - Overall, Ends, and Aisles score rings
  - Best/worst walk stats
  - By Closer breakdown (avg score per person)
  - Repeat Misses frequency table
  - Score trend chart

## Setup

### Google Sheet + Apps Script

1. Create a new Google Sheet
2. Extensions → Apps Script
3. Paste the contents of `Code.gs` into the editor
4. Deploy → New deployment → Web app
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Copy the deployed URL

When updating `Code.gs` later: Deploy → Manage deployments → edit existing deployment (pencil icon) → set version to **New version** → Deploy. This preserves the URL.

### Frontend

1. Clone this repo
2. Open `index.html` and verify the `API` constant on line ~7 contains your Apps Script URL
3. Push to GitHub
4. Settings → Pages → Source: main branch → Save
5. Access at `https://<username>.github.io/<repo-name>/`

### iPhone Home Screen

1. Open the GitHub Pages URL in **Safari**
2. Share button → scroll down → **Add to Home Screen**
3. Launches fullscreen with dark status bar

## Sheet Structure

The `WalkData` tab is auto-created on first request. Columns:

```
ID | Date | ClosedBy | EndsMissed | EndsDetail | MetroMissed | MetroDetail |
AislesMissed | AislesDetail | Misc1Active | Misc1Text | Misc2Active |
Misc2Text | Misc3Active | Misc3Text | TotalScore | TotalPct | SavedAt
```

Detail columns (`EndsDetail`, `MetroDetail`, `AislesDetail`) store JSON arrays of `{note: "..."}` objects. Miss counts are derived from array length on write.

## Technical Notes

- `Content-Type: text/plain` on POST requests is intentional. Apps Script cannot handle CORS preflight (OPTIONS), so this keeps the request "simple" per the CORS spec and avoids the preflight entirely.
- Dates are stored as `YYYY-MM-DD` strings. The Apps Script formats Sheets Date objects back to this format on read to prevent serialization issues.
- Apps Script cold starts can take 1-3 seconds. The UI shows saving/loading states to account for this.
- Repeat Misses matching is case-insensitive string comparison. Consistent naming in notes ("Aisle 7" every time, not "aisle seven" sometimes) produces the best aggregation.
