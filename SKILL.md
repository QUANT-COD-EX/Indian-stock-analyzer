---
name: indian-stock-analyzer
description: Analyzes NSE/BSE-listed Indian stocks via web search — current news, upcoming government schemes (PLI, budget allocations), company projects/capex, and BSE/NSE corporate announcements — then scores and ranks stocks, recommends entry/exit timing (intraday) or ideal holding period (long-term), and picks the best stock(s) for a given budget. Use this whenever the user asks about Indian stocks, NSE or BSE tickers, "should I buy X", comparing multiple Indian stocks against a budget, Indian stock entry/exit timing, or mentions trading/investing in Indian equities — even if they don't explicitly ask for a "report" or "analysis". Always trigger for requests naming Indian stock tickers or companies alongside words like buy, invest, budget, compare, entry, exit, or hold.
---

# Indian Stock Analyzer (NSE/BSE)

Produces a structured, data-backed analysis of one or more NSE/BSE-listed stocks using web search only (no live market-data connector is available). Output is a JSON file other skills/sessions can reuse, plus a readable summary in chat.

This is a **standalone** skill — it does not depend on or call the `trader-*` plugin skills (those are US-market, connector-driven). It is for Indian equities, web-search driven.

## Hard requirements — always

1. **Not financial advice.** Every report — inline and in the JSON's `disclaimer` field — must state this is informational analysis from public sources, not investment advice, and that the user should verify with a SEBI-registered advisor before acting. Never state a recommendation as certain.
2. **Ask mode first.** Before researching, always ask the user to choose between **Intraday** and **Long-term** using `ask_user_input_v0`, unless they already stated it unambiguously in their message (e.g., "day trade", "swing today" → intraday; "invest for 2 years" → long-term). If ambiguous, ask.
3. **Web search only** for prices, news, schemes, filings. Never fabricate a price, date, or filing. If a number can't be found via search, mark it `null` with a note, don't guess.
4. **Source priority** (cite what you actually used in `sources_used`):
   - Primary/official: BSE (bseindia.com), NSE (nseindia.com), PIB/government ministry press releases, SEBI, Union Budget documents
   - Broker research: Groww, Zerodha Varsity/Pulse, Upstox, Angel One
   - Mainstream financial news (for "current news"): Economic Times, Moneycontrol, LiveMint, Business Standard
   - Avoid low-quality aggregator/blog content with no byline or primary sourcing.
5. **Write the JSON file** to `trader/stock-analysis.json` in the working directory (create the `trader/` folder if absent) on every run, then present a human-readable summary in chat from the same data. If `present_files` is available, present the JSON file too.

## Workflow

### Step 1 — Gather inputs
From the user's message, extract:
- Stock(s): ticker or company name(s). Resolve to NSE/BSE symbol via search if only a company name is given (e.g., "Reliance" → `RELIANCE` NSE / `500325` BSE).
- Budget (₹), if already stated in the message. If not, it's gathered in Step 2 — budget is always asked for, single stock or multiple, since affordability is one of the four score components. If the user declines to give one, proceed with affordability's weight redistributed into the other three components, same as before.

### Step 2 — Ask mode and budget
Call `ask_user_input_v0` with up to two questions in one call:
- "Is this for intraday (day) trading, or long-term investing?" — options: `Intraday`, `Long-term`
- "What is your budget?" — always ask this, even for a single stock with no comparison, since affordability is part of the score. Skip only if the user already stated a budget figure in their message.
Skip the mode question only if the user's own wording already makes the mode unambiguous — state your inferred mode in the response if you skip it. Budget is still asked (or confirmed) separately even when mode is skipped.

### Step 3 — Research (per stock, via web_search / web_fetch)
Run separate searches per stock for each of:
1. **Current price & recent move** — last traded price, today's/recent % change, 52-week range (broker sites or news).
2. **Current news** — last 1–4 weeks of company-specific news.
3. **Corporate announcements/filings** — BSE/NSE announcements: results, board meetings, capex approvals, contracts, fundraises, promoter actions. Search `"<company> BSE announcement"` / `"<company> NSE corporate announcement"` and fetch the BSE/NSE page if accessible; fall back to broker-site or news coverage of the same filing if the exchange site blocks fetch.
4. **Upcoming government schemes** — PLI scheme eligibility, budget allocations, sector policy relevant to the company's sector (search PIB/ministry sources).
5. **Upcoming company projects** — new plants, capacity expansion, capex plans, order wins, product launches — with expected timelines if stated.
6. **Analyst/broker view** — target price or outlook from Groww/Zerodha/Upstox/Angel One research if available.

Scale search depth to stock count: 1 stock → thorough (6–10 searches); 3+ stocks → 3–5 focused searches each, prioritizing recency.

**Search time horizon — tie to the mode chosen in Step 2.** Indian broker/charting sites commonly expose price history as tabs: `1D`, `5D`, `1M`, `6M`, `YTD`, `1Y`, `5Y`, `Max`. Mirror that convention when choosing search windows:
- **Intraday** → use the `1D`, `5D`, `1M` windows only (e.g., `"<company> share price today"`, `"<company> share price 5 day"`, `"<company> stock October 2026"` using the actual current month/year). Discard or down-weight anything older than ~1 month; it isn't relevant to a day trade. If price results conflict or carry different dates, prefer the most recently dated one and explicitly flag the price as potentially stale rather than presenting it as current (search has no live feed).
- **Long-term** → use the remaining windows: `6M`, `YTD`, `1Y`, `5Y`, `Max` (e.g., `"<company> stock performance 1 year"`, `"<company> 5 year return"`, `"<company> all-time high low"`, `"<company> YTD performance"`). This surfaces multi-year catalysts, historical volatility, and whether current news is a blip or part of a longer trend — don't rely solely on last-week news for a long-term call.

### Step 4 — Score each stock (0–100)
Compute four sub-scores, each 0–100, then a weighted total. Show your component scores, don't just assert a final number.

| Component | Weight | What it measures |
|---|---|---|
| News sentiment & momentum | 30% | Net tone of recent news/announcements (positive catalysts vs. negative/risk items found) |
| Upcoming catalysts | 30% | Concrete, dated or near-term schemes/projects/contracts that could move the stock — stronger if dated and specific, weaker if speculative |
| Affordability (budget fit) | 20% | Only if budget given: how well the budget allows a meaningful position (whole shares bought, ability to diversify across the given list) — if no budget given, redistribute this weight proportionally into the other three |
| Risk level | 20% (inverted: lower risk → higher score) | Volatility implied by recent price swings, sector/regulatory risk, leverage/debt flags if visible in news |

Document the reasoning for each component briefly (1–2 lines) so the score isn't a black box.

### Step 5 — Timing recommendation
- **Intraday mode**: Based on the latest available price/levels found via search (broker charting pages, news), suggest a plausible entry zone (e.g., "near today's support around ₹X, confirmed on a breakout above ₹Y") and an exit/stop zone. Explicitly caveat that intraday levels are time-sensitive and the search data may be stale by market open — recommend the user confirm live levels on their broker terminal before acting. Do NOT claim real-time accuracy.
- **Long-term mode**: Give an **ideal holding period estimate** per stock (e.g., "12–24 months, tied to PLI scheme payout timeline" or "3–5 years for capex-led capacity doubling to reflect in earnings"), tied explicitly to the catalyst timeline found in Step 3 (scheme/project completion dates) rather than a generic default.

### Step 6 — Rank & recommend (multi-stock + budget)
If multiple stocks were given with a budget: rank by total score, then state which stock(s) the budget best supports — factoring in share price vs. budget (whole-share affordability) and whether splitting the budget across top-ranked stocks for diversification beats concentrating it in the single highest scorer. Explain the tradeoff; don't just pick one silently.

### Step 7 — Output

**JSON** (write to `trader/stock-analysis.json`, full schema in `references/schema.md`):
- `mode` (intraday | long_term), `budget`, `generated_at`, `disclaimer`
- `stocks[]`: each with `symbol`, `exchange`, `last_price`, `price_as_of`, `scores` (news, catalysts, affordability, risk, total), `score_notes`, `news_summary`, `catalysts` (array with description + expected date if known), `filings` (array), `timing` (entry/exit for intraday, or `holding_period` for long-term), `sources_used`
- `recommendation`: ranked list + budget allocation reasoning (if applicable)

**Chat summary**: A concise human-readable version of the same — per-stock score + 1-line reasoning, the timing/holding call, the budget recommendation, and the disclaimer. Don't dump the raw JSON into chat; summarize it, and mention the file was saved if `present_files` is used.

See `references/schema.md` for the exact JSON field names and an example.
