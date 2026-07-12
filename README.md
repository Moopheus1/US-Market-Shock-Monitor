# US Market Shock Monitor

> **This dashboard answers two different questions, on two different timescales — don't mix them up.**
>
> **Part I — "Are we in a risky macro regime?"** Strategic positioning (cash levels, hedges, sector allocation) over a 2–4 week horizon.
>
> **Part II — "Is today likely to open higher or lower?"** Tactical entry/exit timing for the current session.

## What This Dashboard Is For

| Question | Answer Source | Part |
|----------|---------------|------|
| **"Are we in a risky macro regime?"** | Yield curve, CPI, credit spreads, unemployment, Fed balance sheet, VIX, breadth | **Part I** — slow-moving, regime-defining indicators, FRED data (1-day lag) |
| **"Is today going to be red?"** | S&P/Nasdaq/Dow/Russell futures, real-time VIX/DXY/oil, MOO imbalance, buy/sell volume | **Part II** — real-time/near-real-time via Yahoo Finance and manual entry |

Part I tells you the **structural risk picture** as of yesterday's close — use it to answer *"should I be 20% cash or 60% cash this month?"*, not *"should I sell at 9:45am?"* Part II is built for that second question instead.

The two parts can and do disagree with each other. That's not a bug — a calm macro regime (Part I) with a rough premarket session (Part II) is a completely normal, informative combination, not a contradiction to resolve.

---

## Part I: The Signpost Model

**This dashboard does not average its 14 indicators into one blended score.** An earlier version did, and it was wrong for a specific reason: averaging lets 13 calm indicators dilute one indicator that's actually screaming. The current model counts discrete triggers instead — the same way Bank of America's public "7 of 10 bear-market signposts triggered" commentary works.

### How a signpost triggers

Each of the 14 FRED indicators is checked against up to three independent conditions. **Any one of them firing counts as that indicator's signpost being triggered:**

- **LEVEL** — is the indicator currently sitting in its own red zone?
- **TREND** — has it moved fast toward red recently (≥0.15 on a 0–1 zone-position scale, over a 5-observation lookback), even from a calm level? Computed from real historical FRED values, not invented.
- **VIX and treasury yields only, a third condition:**
  - VIX uses a **separate stress-only zone** for signpost purposes (green ≤17 / yellow 17–25 / red ≥25). Its card display keeps a different, deliberately contrarian zone (green = high VIX, i.e. "buying opportunity" for entry timing) — that framing is fine for a card you click into for entry decisions, but it actively broke crash detection when used for the composite score (see Known Limitations below).
  - Treasury 10Y/2Y have a **flight-to-safety collapse watch**: a ≥15% relative *drop* in yield over the lookback window triggers a signpost, independent of which zone the level sits in. A sudden yield collapse (investors fleeing to Treasuries) is itself a stress signal, not just a rise in yields.

### Score and thresholds

The header shows a straight count, e.g. `6/14`, converted to a percentage of available indicators (not all 14 always have fresh data — see Data Availability below).

| Signpost % | Badge | Action |
|---|---|---|
| <20% | accumulate | Overweight equities, buy dips aggressively |
| 20–39% | maintain | Hold positions, add hedges (puts/VIX calls) |
| 40–59% | reduce risk | Raise cash to 20–30%, avoid new equity positions |
| 60%+ | derisk | Sell aggressively, raise cash >50%, go defensive |

Click any indicator chip above the grid to jump straight to that card and see exactly why it triggered (level, trend, or — for VIX/yields — the stress/collapse condition).

### Backtest results (why these specific numbers)

The model above was tested against **2015–2026, 2,896 trading days, out-of-sample** (only data that would have actually been available as of each historical date was used — no lookahead). Full methodology and raw findings are in the conversation history that built this dashboard; summary:

- **The original 65% derisk threshold never fired once in 11 years** — the historical maximum was 58.3% (Sept 2023). It was recalibrated to 60%, which the post-fix model actually reaches.
- **Before the VIX/yield fixes above**, the model badly undersold two real crashes: COVID peaked at only 45.5% ("maintain"), the 2018 Q4 selloff peaked at 27.3% ("maintain"). Root cause for COVID: VIX hit 82.69 during the actual panic, which the *unmodified* contrarian zone read as calm/green — actively fighting crash detection.
- **After the fixes:** COVID's peak signpost reading rises to 72.7% (correctly "derisk"), 2022's peak rises to 63.6% (correctly "derisk").
- **Known, unfixed limitation:** the 2018 Q4 selloff still only reaches 27.3% even after the fixes. That correction wasn't well-represented by any of the 12 available FRED series simultaneously — it was mostly a growth/rate-fear equity correction, not a classic vol-spike or flight-to-safety event. Closing this gap needs indicator categories this dashboard doesn't have (credit market breadth, equity positioning, liquidity stress) — it's a data-coverage gap, not a threshold problem.
- **What did validate:** the "reduce risk" bucket shows a real, if modest, forward-return edge over calmer buckets (lower hit rate, weaker 4–8 week returns). The top-end/derisk signal and fast-crash detection were the broken parts, not the underlying concept.

**Do not treat any threshold in this model as backtested to a rigorous statistical standard.** Sample sizes shrink a lot once you account for autocorrelation (the "reduce risk" bucket's 355 raw days collapse to 48 independent episodes), and the trend/level thresholds themselves (0.15, 5-observation lookback, VIX's 17/25 split, the 15% yield-collapse threshold) are judgment calls informed by the backtest, not fitted to it.

---

## Part II: Pre-Market Pulse & Bias Score

Real-time-ish data (15-minute refresh during the relevant window) feeding a weighted bias score, separate from Part I entirely.

### What feeds the bias score automatically

`fetch-yahoo.yml` pulls ES=F, NQ=F, YM=F, RTY=F futures plus ^VIX, ^TNX, DX-Y.NYB, and CL=F spot, every 15 minutes from 11:00–14:45 UTC weekdays (covers your ~8pm SGT session through the US open). `fetch-breadth.yml` separately computes an automated buy/sell volume proxy (see below) from SPY/QQQ/DIA/IWM, every 15 minutes from 13:00–17:59 UTC weekdays (first 4 hours of the trading day).

### Buy/sell volume proxy

There is no free, automatable source for literal tick-level buyer/seller-tagged trade volume (that requires a paid NBBO quote feed). This proxy instead classifies each 1-minute bar for SPY/QQQ/DIA/IWM as "up" or "down" volume based on whether that bar closed above or below the prior bar's close, then sums across the window. **This is a bar-direction approximation (the same family as OBV/UVOL-DVOL), not tick-level aggressor-side volume** — labeled as such directly in the code and on the panel so it's never mistaken for something more precise than it is.

### MOO imbalance (manual entry)

Market-on-open imbalance (S&P 500 / Nasdaq 100 / Dow 30 / Mag 7, in millions $) is **not automatable**. The only aggregated source found (FinancialJuice, posted to X/Telegram) is proprietary and scraping it would violate their ToS — the same reasoning applies here as to any other paywalled/ToS-restricted data source. You paste the numbers in yourself each morning from whatever you're already watching; only the S&P 500 figure feeds the bias score (weight 12), the other three are shown as a breadth read (broad-based vs. narrow) but aren't separately scored, since they overlap heavily with the S&P 500 figure and would double-count the same flow.

### Bias score thresholds

| Score | Badge | Action |
|---|---|---|
| 0–14 | strong bear | Futures down + VIX up, one-sided — reduce risk, raise cash, add hedges |
| 15–39 | bearish | Futures net negative — favor shorts, tighten stops on longs |
| 40–60 | neutral | Mixed/weak signals — wait for first-hour confirmation before sizing |
| 61–84 | bullish | Futures net positive — favor longs, tighten stops on shorts |
| 85–100 | strong bull | Futures up + VIX down, one-sided — consider adding risk, size up longs |

---

## Reference-Only Panels (Not Scored Into Either Part)

Sit above both Part I and Part II at the top of the page, since they're context for both questions rather than belonging to one. **None of these feed `calcComposite()` or `computeSignposts()`.**

### NAAIM Active Manager Positioning
Weekly average equity exposure reported by NAAIM member firms. **Read this the same way as VIX's card display — it's contrarian, not monotonic:** extreme HIGH exposure = crowding/euphoria = contrarian derisk warning; extreme LOW/negative exposure = capitulation = contrarian buy signal; the middle of the range is unremarkable. Higher does **not** simply mean riskier.

⚠️ **NAAIM announced this data moves to a subscription model effective August 1, 2026.** The panel will show "unavailable" after that date unless a paid replacement source is wired in. `fetch-naaim.yml` fails gracefully rather than breaking anything else, and its file header has full removal instructions if you decide not to pay for a replacement.

### Gold Futures (GC=F)
Uses futures, not the GLD ETF — GLD only trades US market hours and would miss overnight moves relevant to Part II's "today" question; GC=F trades ~23 hrs/day. **Backtested against three known stress events and found unreliable as a hedge:** worked in 2018 Q4 (+6.6% while SPY fell −19.7%), but went flat-to-down in both COVID (−3.6%) and the 2022 selloff (−7.3%). Shown as context, not signal.

### Bitcoin (BTC-USD)
Trades 24/7, no ticker-swap needed for Part II relevance. **Backtested against the same three events — this is not a hedge.** BTC fell harder than SPY in all three: −38.1% vs. SPY's −19.7% (2018 Q4), −33.4% vs. −34.1% (COVID), −58.8% vs. −25.4% (2022). It behaves like a high-beta risk asset that amplifies the same move equities make, not a diversifier. Never read a BTC move as a safe-haven confirmation the way gold sometimes can be.

### What was tested and rejected as unusable
For transparency: $TICK and CBOE put/call ratio were investigated as free market-breadth proxies and rejected — the free CBOE put/call CSV feed has been dead since October 2019 (still returns HTTP 200, but the data inside stops in 2019), and Yahoo's `C:TICK`/`C:TRIN` composite tickers resolve as symbols but carry zero actual price history. AAII's sentiment survey is free to view but the site actively blocks automated requests. None of these are wired into the dashboard.

---

## FRED Data Lag Reference

| Indicator | FRED Lag | What You Actually Get |
|-----------|----------|----------------------|
| **VIX** | 1 day | Yesterday's close |
| **Yield Curve** | 1 day | Yesterday's Treasury close |
| **Credit Spreads** | 1 day | Yesterday's bond market |
| **CPI** | ~2 weeks | Last month's inflation |
| **Unemployment** | ~1 month | Last month's jobs |
| **Fed Balance Sheet** | Weekly | Last Wednesday's snapshot |
| **M2 Money Supply** | Weekly | Last week's money supply |

## FOMC Reference Links

The FOMC section links out to the Federal Reserve's own meeting calendar/statements page and to the CME FedWatch Tool for rate-decision probabilities. Neither is pulled into the dashboard automatically — CME doesn't offer a free public API for FedWatch data, so this is a link-out, not embedded data.

---

## Data Pipelines (GitHub Actions)

| Workflow | Schedule | Writes | Notes |
|---|---|---|---|
| `fetch-fred.yml` | 11:30 & 21:00 UTC daily | `data/{vix,dxy,yield10y2y,oil,cpi,unemployment,credit,fedbal,consumer,treasury10y,treasury2y,m2,sp500,nasdaq}.json`, `data/meta.json` | Requires `FRED_API_KEY` secret. `meta.json` tracks `fetchedCount`/`failedSeries` so a bad key or partial failure is visible instead of hidden behind a "fresh" timestamp. `sp500`/`nasdaq` are fetched but not displayed — index price levels don't fit the zone-based framework (see index.html comments). |
| `fetch-yahoo.yml` | Every 15 min, 11:00–14:45 UTC weekdays | `data/pulse.json` | ES=F/NQ=F/YM=F/RTY=F futures + ^VIX/^TNX/DX-Y.NYB/CL=F. Fetched server-side (GitHub Actions runner) specifically because Yahoo's chart API has no CORS headers permitting browser-origin requests from GitHub Pages — this can never work as a client-side fetch. |
| `fetch-breadth.yml` | Every 15 min, 13:00–17:59 UTC weekdays | `data/breadth.json` | SPY/QQQ/DIA/IWM 1-min bar-direction volume classification. |
| `fetch-naaim.yml` | Thu 19:00 UTC + Fri 15:00 UTC backup | `data/naaim.json` | Free access ends ~Aug 1, 2026 per NAAIM's own announcement. Appends to a rolling history rather than overwriting, so trend data survives even after the source goes paywalled. |
| `fetch-gold.yml` | 12:00 & 21:30 UTC weekdays | `data/gold.json` | GC=F futures. |
| `fetch-crypto.yml` | 12:00 & 21:30 UTC weekdays | `data/bitcoin.json` | BTC-USD. |

All six fail gracefully — a failed fetch writes `_fetchFailed: true` and preserves the last good data rather than crashing or overwriting with garbage. None of them require anything beyond `FRED_API_KEY` (only `fetch-fred.yml` needs it; the rest hit free, keyless public endpoints).

### Reliability note

GitHub's own cron scheduler has, in practice, sometimes failed to fire scheduled runs at all (observed: workflows sitting with zero scheduled executions despite correct cron syntax). If a workflow's data goes stale, don't assume the code is broken — check the Actions tab's "Triggered via" column first. If it says "Manually run" for every recent execution and never "Scheduled," the fix is a redundant external trigger (e.g. cron-job.org calling the GitHub REST API's `workflow_dispatch` endpoint), not a code change.

---

## UI Notes

- **Signal framework boxes** for both Part I and Part II are collapsible (click to close, click again to reopen) and load **expanded by default**.
- **Clicking an indicator card toggles its detail panel** — click again on the same card to close it, no need to hunt for the X button. The detail panel physically relocates in the page to sit directly under whichever card was clicked, instead of always rendering at the bottom of the full grid.
- **Light theme.** All colors, including several backgrounds that were hardcoded hex values rather than theme variables, were converted — check contrast yourself after any future edits, since hardcoded colors won't follow a future theme change automatically.
- Font sizes were increased uniformly (+2px across every rule in the stylesheet, including the large score numbers) for readability.
- Treasury 10Y/2Y zone thresholds were recalibrated from a pre-2022 rate regime (green up to 3.0%) to reflect the post-2022 rate environment (green up to 4.25%/4.0%) — see index.html comments for the reasoning. Revisit these again if rates move to a durably different regime.

## Setup Instructions

### 1. Get a FRED API Key (free)

1. Go to https://fred.stlouisfed.org/docs/api/api_key.html
2. Request a key (instant, no approval needed)
3. Copy the key

### 2. Add the Key to GitHub Secrets

1. In your repo → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `FRED_API_KEY`
4. Value: your FRED API key
5. Click **Add secret**

(Only `fetch-fred.yml` needs this. The other five workflows hit free, keyless endpoints.)

### 3. Commit These Files

```bash
git add .github/workflows/fetch-fred.yml
git add .github/workflows/fetch-yahoo.yml
git add .github/workflows/fetch-breadth.yml
git add .github/workflows/fetch-naaim.yml
git add .github/workflows/fetch-gold.yml
git add .github/workflows/fetch-crypto.yml
git add index.html
git commit -m "Add signpost model, Part II pulse, reference panels"
git push
```

### 4. Trigger the First Run of Each Workflow

Each workflow needs one manual trigger to seed its `data/*.json` file — none of this appears automatically until that first run:

1. Go to **Actions** tab in your repo
2. For each of the six workflows: click it → **Run workflow** → **Run workflow**
3. Wait ~1 minute per workflow
4. Refresh your repo — you should see a `data/` folder populated with JSON files
5. Your site will now display live data instead of demo data

### 5. How It Works

```
GitHub Actions (6 workflows, various schedules) → free APIs (FRED/Yahoo/NAAIM) → data/*.json → GitHub Pages
                                                 ↑
                                          FRED_API_KEY only (secure, GitHub secret)
```

- The FRED API key never leaves GitHub's secure environment; the other five workflows need no key at all
- The browser loads local JSON files — no CORS issues, no exposed keys
- If any pipeline's data is missing, the dashboard falls back to demo data or shows "unavailable" — it doesn't crash

## File Structure

```
.
├── .github/
│   └── workflows/
│       ├── fetch-fred.yml       # FRED macro data, 2x daily
│       ├── fetch-yahoo.yml      # Pre-market futures/VIX/DXY/oil, every 15min in-window
│       ├── fetch-breadth.yml    # Automated buy/sell volume proxy, every 15min in-window
│       ├── fetch-naaim.yml      # NAAIM positioning, weekly (free until ~Aug 1 2026)
│       ├── fetch-gold.yml       # Gold futures (GC=F), 2x daily
│       └── fetch-crypto.yml     # Bitcoin (BTC-USD), 2x daily
├── data/                        # Generated by workflows (don't commit manually)
│   ├── meta.json
│   ├── vix.json / dxy.json / ... (14 FRED series)
│   ├── pulse.json
│   ├── breadth.json
│   ├── naaim.json
│   ├── gold.json
│   └── bitcoin.json
├── index.html                   # Dashboard (reads data/*.json)
└── README.md
```

## Troubleshooting

**Site still shows "demo mode" for Part I**
- `fetch-fred.yml` hasn't run yet. Trigger it manually: Actions → Fetch FRED Data → Run workflow.

**Part II pulse bar / breadth panel show "unavailable"**
- `fetch-yahoo.yml` / `fetch-breadth.yml` haven't run yet, or ran outside their scheduled window (they only fire during specific UTC hours — see the pipeline table above). Trigger manually to seed the first data.

**NAAIM panel shows "unavailable" and it's on/after Aug 1, 2026**
- Expected — NAAIM's free access ended per their own announcement. See `fetch-naaim.yml`'s file header for full removal instructions if you're not replacing it with a paid source.

**A workflow's data looks stale (last updated more than a day or two ago)**
- Check the Actions tab's "Triggered via" column. If every recent run says "Manually run," the scheduled cron isn't firing — see the Reliability Note above.

**Workflow fails**
- For `fetch-fred.yml`: check `FRED_API_KEY` is set correctly in repo Settings → Secrets, and check `meta.json`'s `failedSeries` field.
- For the other five: check the Actions log — they hit free public endpoints, so failures are usually a source-side change (page structure, rate limiting) rather than a credentials issue.

## Data Sources

- [FRED](https://fred.stlouisfed.org/) (Federal Reserve Bank of St. Louis) — free for public use, requires a free API key
- [Yahoo Finance](https://finance.yahoo.com/) — undocumented public chart API, no key required, fetched server-side to avoid CORS
- [NAAIM](https://naaim.org/programs/naaim-exposure-index/) — free until ~Aug 1, 2026 per their own announcement
- [Federal Reserve FOMC Calendar](https://www.federalreserve.gov/monetarypolicy/fomccalendars.htm) and [CME FedWatch Tool](https://www.cmegroup.com/markets/interest-rates/cme-fedwatch-tool.html) — linked, not embedded
