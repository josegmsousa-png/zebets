# ZéBets — Project Context

## Overview
Personal betting tracker web app. Single `index.html` file hosted on **GitHub Pages**, with a **Supabase PostgreSQL** backend.

- **Live URL:** `https://josegmsousa-png.github.io/zebets/`
- **GitHub repo:** `github.com/josegmsousa-png/zebets`
- **File:** single `index.html` — all CSS/JS inline, no build step

---

## Tech Stack
- **Frontend:** Vanilla HTML/CSS/JS, Chart.js, DM Sans + DM Serif Display fonts
- **Backend:** Supabase REST API (direct from browser)
- **Charts:** Chart.js v4.4.0
- **Hosting:** GitHub Pages (static)

---

## Supabase
- **URL:** `https://vgsvhixbvosimiemooiw.supabase.co`
- **Anon key:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InZnc3ZoaXhidm9zaW1pZW1vb2l3Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzY5NzMzMTgsImV4cCI6MjA5MjU0OTMxOH0.a3DiUpZUaOEtuEHg66RSKtcXCiHlN6LiRYkx1TFoPss`
- **Table:** `bets`
- **RLS:** public access enabled

### Supabase Table Schema
```sql
bets (
  id uuid primary key,
  date text,           -- format: "M/D/YYYY" e.g. "5/1/2026"
  bet text,            -- selection name
  game text,           -- "Team A - Team B"
  odd numeric,
  stake numeric,
  outcome text,        -- 'W', 'L', or '' (pending)
  sport text,          -- 'basketball', 'soccer', 'nfl', 'mlb', 'other'
  created_at timestamptz,
  acca_id text,        -- shared UUID for accumulator legs
  acca_combined_odd numeric,
  acca_stake numeric,
  acca_leg integer,    -- 1, 2, 3...
  acca_total_legs integer
)
```

---

## Seed Balances
```js
const SEED_BALANCE = { "4/2/2026": 35.0, "5/1/2026": 30.0, "6/1/2026": 30.0 };
```
- April started at €35, May at €30, June at €30
- Total invested: €95
- Each month shows "Started at €X · Total €95"

---

## Features Implemented

### Stats Cards (6 cards)
- **Total Bets** — monthly count (big) + total all-time underneath
- **Current Balance** — running balance from seed, shows pending stake
- **Win Rate** — all-time %, with W/L counts
- **ROI** — monthly % (big, coloured) + all-time underneath
- **P&L** — monthly €X + staked this month + all-time underneath
- **Avg Odd** — monthly average + all-time underneath

### Charts
- **Daily Balance** — line chart per month
- **Win/Loss Ratio** — donut chart with % in center, monthly data

### Bet Table
Columns: Date | Sport | Bet | Game | Odd | Stake (€) | Outcome | P&L | Actions

- Sort: date desc, same game+date grouped together, then by `created_at`
- Team logos from ESPN CDN (NBA, NFL, MLB, Soccer) + embedded logos
- Edit (✎) and Delete (✕) per row
- Outcome: W / L / — badges

### Accumulators (Acca)
- Stored as multiple rows sharing same `acca_id`
- Each leg has its own outcome (set via edit modal)
- Acca header outcome **derived** from legs: all W → W, any L → L, else pending
- Combined odd = product of legs, rounded to 2dp (matches Betano)
- Table display: header row (Date | ⚡ | Double/Treble/N-Fold | — | combined odd | stake | badge | P&L | ✕) then leg rows indented
- Balance/P&L uses `getAccaOutcome()` not individual leg outcome

### Months
- Tab per month, "+ Next Month" dashed button
- Empty months show ✕ to delete
- Balance carries forward month-to-month

### Themes
- Dark (default: `#0d0f14`) / Light (warm parchment `#f5eedc`)
- Toggle 🌙/☀️ in header, saved to localStorage

### Mobile
- ≤640px: table hidden, replaced with bet cards
- Acca cards show legs with sport icon + bet + odd + outcome tick
- Forms: 2-column grid (Date|Sport, Odd|Stake, full-width for Bet/Game)
- Modals only close via Cancel/Save buttons (not by clicking outside)

### Favicon
- Browser: SVG with dark background, italic gradient Z + orange basketball dot
- iOS: PNG 180×180 rendered version (same design)

---

## Logo System

### NBA (ESPN CDN)
```js
'team name': 'ABBR'  // → espncdn.com/combiner/i?img=/i/teamlogos/nba/500/ABBR.png
```
All 30 NBA teams + EuroLeague teams using football club logos where available:
- Olympiacos → ESPN soccer `435`
- Real Madrid Baloncesto → ESPN soccer `86`
- Barcelona basketball → ESPN soccer `83`

### NFL (ESPN CDN)
All 32 NFL teams

### MLB (ESPN CDN)
All 30 MLB teams

### Soccer (ESPN CDN with `e_` prefix OR api-sports)
- `e_XXX` → `espncdn.com/i/teamlogos/soccer/500/XXX.png`
- Numeric ID → `media.api-sports.io/football/teams/XXX.png`

**Leagues covered:** Premier League, La Liga, Bundesliga, Ligue 1, Serie A, Eredivisie, Primeira Liga, Liga Portugal 2, Segunda División (Spain), English Championship/League One

### Country Flags (national teams, any sport)
- `COUNTRY_FLAGS` map: normalized country name → ISO 3166 code (~190 entries, English + Portuguese names)
- URL: `flagcdn.com/w80/{code}.png` (England/Scotland/Wales/N. Ireland use `gb-eng` etc.)
- **Exact-match only** (no fuzzy) to avoid collisions with club names
- Checked in `getTeamLogo` right after `EMBEDDED_LOGOS`, before sport-specific maps
- Rendered with `object-fit: cover` (flags are rectangular → circular crop); club logos keep `contain`

### Embedded Logos (base64, in `EMBEDDED_LOGOS` object)
- Penafiel, Nice, Famalicão, Torreense
- **Important:** base64 strings MUST use backtick delimiters not single quotes (base64 can contain `'` characters which break JS string literals)

### Logo Lookup Priority
1. `EMBEDDED_LOGOS` (exact + fuzzy, accent-normalized)
2. `COUNTRY_FLAGS` (exact match only)
3. Sport-specific map (NBA/NFL/MLB/SOCCER)
4. Fuzzy match (min 8 chars to avoid false positives)

### Key IDs (verified ESPN)
| Team | ID |
|------|-----|
| Benfica | e_211 |
| Sporting CP | e_2250 |
| FC Porto | e_212 |
| Braga | e_2994 |
| Estoril | e_12216 |
| Famalicão | embedded |
| Arsenal | e_359 |
| Real Madrid | e_86 |
| Barcelona | e_83 |
| Atletico Madrid | e_1068 |
| Olympiacos FC | e_435 |
| Rayo Vallecano | e_101 |
| Real Oviedo | e_92 |

---

## Key JS Functions

| Function | Purpose |
|----------|---------|
| `computeDailyBalances(monthKey)` | Runs from SEED_BALANCE, carries forward. Uses `getAccaOutcome()` for acca P&L |
| `getAccaOutcome(accaId)` | Derives W/L/pending from individual leg outcomes. **Must be defined before `computeDailyBalances`** |
| `render()` | Main render — tabs, stats, charts, table |
| `renderTable(bets)` | Desktop table + mobile cards. Handles singles and acca groups |
| `normalizeKey(str)` | Lowercases, strips accents, normalizes spaces. **Must be at top of script** |
| `getSoccerLogo(teamName)` | Checks EMBEDDED_LOGOS first, then SOCCER_LOGOS |
| `getTeamLogo(teamName, sport)` | Routes to correct logo source by sport |
| `getAccaOutcome(accaId)` | All W → W, any L → L, else '' |
| `setLegOutcome(legId, outcome)` | Updates single leg, re-renders |
| `saveBet()` | Add/edit bet. For acca legs: preserves acca fields, recalcs combined odd |
| `saveAcca()` | Creates multiple rows sharing acca_id |
| `deleteAcca(accaId)` | Deletes all legs |
| `toggleTheme()` | Switches dark/light, redraws charts |

---

## Known Gotchas / Previous Bugs

1. **`getAccaOutcome` must be defined BEFORE `computeDailyBalances`** — otherwise ReferenceError crashes the whole page
2. **`normalizeKey` must be defined at top of script** — used in SOCCER_LOGOS lookup before function declarations are reached
3. **Base64 in EMBEDDED_LOGOS must use backtick delimiters** — base64 can contain `'` which breaks JS string literals
4. **No duplicate `const` declarations in `render()`** — strict JS throws on these
5. **Acca P&L uses `getAccaOutcome()` not leg.outcome** — individual leg outcomes are per-leg, acca outcome is derived
6. **`totalBetsCount` must be declared before `avgOddAll`** which uses it
7. **Fuzzy logo matching min 8 chars** — shorter keys like `'sporting'` match wrong teams
8. **Acca combined odd rounded to 2dp** — matches Betano's rounding (e.g. 1.25×1.55=1.94 not 1.9375)
9. **Modal overlays have no onclick close** — removed to prevent accidental close when selecting text

---

## File Update Workflow
1. Make changes to `index.html` locally
2. `git add` → `git commit` → `git push` (repo connected to `origin/main`, Git Credential Manager handles auth)
3. GitHub Pages auto-deploys in ~1 min
4. Hard refresh: `Ctrl+Shift+R`

---

## Sports Dropdown Options
- 🏀 Basketball (`basketball`)
- 🏈 Football (`nfl`)
- ⚽ Soccer (`soccer`)
- ⚾ Baseball (`mlb`)
- 🎯 Other (`other`)
