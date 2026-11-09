<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=28&pause=1000&color=2E86DE&center=true&vCenter=true&width=600&lines=Indian+Stock+Analyzer;NSE+%2B+BSE+%7C+News+%2B+Schemes+%2B+Filings;Built+as+a+Claude+Skill" alt="Typing SVG" />

<br/>

![NSE](https://img.shields.io/badge/Exchange-NSE-1f6feb?style=for-the-badge)
![BSE](https://img.shields.io/badge/Exchange-BSE-ff6b00?style=for-the-badge)
![Claude Skill](https://img.shields.io/badge/Claude-Skill-cc785c?style=for-the-badge&logo=claude&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)

**A Claude skill that analyzes NSE & BSE listed Indian stocks using live web search -- news, government schemes, company projects, and exchange filings -- then scores, ranks, and recommends based on your budget.**

</div>

---

## WARNING: Disclaimer

> This skill produces **informational analysis only**, assembled from public web sources. It is **not investment advice**, is **not built or reviewed by a SEBI-registered advisor**, and prices/data may be stale or inconsistent since there is no live market-data feed. Always verify current prices and consult a qualified, SEBI-registered advisor before trading or investing.

---

## What it does

| Capability | Description |
|---|---|
| News analysis | Pulls recent company news for sentiment and momentum |
| Government schemes | Checks PLI scheme eligibility, budget allocations, sector policy (PIB/ministry sources) |
| Upcoming projects | Surfaces capex plans, new plants, capacity expansion, contract wins |
| Exchange filings | Checks BSE/NSE corporate announcements -- results, board meetings, fundraises |
| Scoring & ranking | Scores each stock 0-100 across news sentiment, catalysts, affordability, and risk |
| Timing calls | Entry/exit zones for intraday, or an ideal holding period for long-term investing |
| Budget-aware | Given multiple stocks + a budget, ranks and recommends an allocation |
| Structured output | Writes a reusable JSON report alongside a readable chat summary |

---

## Installation

This is a **Claude Skill** -- a folder of instructions Claude reads to perform a task consistently. It works with any Claude surface that supports Skills (Claude.ai with Skills enabled, Claude Code, Claude Cowork, etc.).

### Option A -- Claude.ai (web/desktop/mobile)

1. Download this repo as a ZIP: click **Code -> Download ZIP** on GitHub, or run:
   ```bash
   git clone https://github.com/<your-username>/indian-stock-analyzer.git
   ```
2. In Claude.ai, go to **Settings -> Capabilities -> Skills** (or wherever your account's Skills manager is).
3. Upload the `indian-stock-analyzer` folder (or its ZIP) as a new Skill.
4. Start a chat and ask something like *"Should I buy Reliance Industries on NSE?"* -- Claude will automatically detect and use the skill.

### Option B -- Claude Code / local skills folder

1. Clone the repo:
   ```bash
   git clone https://github.com/<your-username>/indian-stock-analyzer.git
   ```
2. Copy the `indian-stock-analyzer` folder into your Skills directory, e.g.:
   ```bash
   cp -r indian-stock-analyzer ~/.claude/skills/
   ```
   *(path may differ depending on your setup -- check your Claude Code docs for the exact Skills directory)*
3. Restart/reload Claude Code so it picks up the new skill.

### Option C -- Manual reference

Don't want to install it as a formal Skill? You can also just open `SKILL.md` in a chat with Claude and say *"follow these instructions"* -- it's plain Markdown, no special tooling required.

---

## Usage

Just ask naturally -- no special syntax needed:

```
Should I buy TCS on NSE?
Compare SBI, HDFC Bank and Federal Bank for a Rs 50,000 budget
Is Reliance Industries a good long-term investment?
What's a good entry point for Tata Motors today?
```

Claude will:
1. Ask whether this is **Intraday** or **Long-term**, and your **budget**
2. Research news, filings, schemes, and projects via web search
3. Score and rank the stock(s)
4. Give you entry/exit timing (intraday) or a holding-period estimate (long-term)
5. Save a structured report to `trader/stock-analysis.json`

---

## Example output (abridged)

```json
{
  "mode": "long_term",
  "budget_inr": 50000,
  "stocks": [
    {
      "symbol": "FEDERALBNK",
      "exchange": "NSE",
      "scores": { "news_sentiment": 82, "upcoming_catalysts": 70, "affordability": 90, "risk": 55, "total": 74.6 },
      "timing": { "long_term_holding_period": "12-24 months, riding continued earnings momentum..." }
    }
  ],
  "recommendation": {
    "ranked": ["FEDERALBNK", "SBIN", "HDFCBANK"],
    "budget_allocation_reasoning": "..."
  }
}
```

Full schema: [`references/schema.md`](references/schema.md)

---

## Repo structure

```
indian-stock-analyzer/
|-- SKILL.md              # The skill definition Claude reads
|-- references/
|   `-- schema.md          # Output JSON schema + worked example
|-- README.md              # You are here
`-- .gitignore
```

---

## Data sources

| Priority | Source type | Examples |
|---|---|---|
| 1 | Primary/official | BSE, NSE, PIB, SEBI, Union Budget documents |
| 2 | Broker research | Groww, Zerodha Varsity, Upstox, Angel One |
| 3 | Financial news | Economic Times, Moneycontrol, LiveMint, Business Standard |

No live market-data connector is used -- all data comes from real-time web search, so always cross-check prices before acting.

---

## Contributing

Issues and PRs welcome -- especially around improving price-data reliability (search results for Indian stocks are often inconsistent across sources/dates) and expanding scheme/filing coverage.

## License

MIT -- see [`LICENSE`](LICENSE).

---

<div align="center">
<sub>Built for learning and research. Not a substitute for professional financial advice.</sub>
</div>
