# Output JSON schema — trader/stock-analysis.json

```json
{
  "mode": "intraday | long_term",
  "budget_inr": 50000,
  "generated_at": "2026-10-30T10:00:00+05:30",
  "disclaimer": "This is an informational analysis from public sources, not investment advice. Verify with a SEBI-registered advisor before trading or investing.",
  "stocks": [
    {
      "symbol": "RELIANCE",
      "exchange": "NSE",
      "company_name": "Reliance Industries Ltd",
      "last_price_inr": 2950.5,
      "price_as_of": "2026-10-29 (previous close, via search — confirm live price before acting)",
      "scores": {
        "news_sentiment": 72,
        "upcoming_catalysts": 80,
        "affordability": 65,
        "risk": 70,
        "total": 73.8
      },
      "score_notes": {
        "news_sentiment": "Mostly positive coverage of Q2 results beat; one article flagged debt concerns.",
        "upcoming_catalysts": "New energy capex ramp expected H1 2027 per company statements.",
        "affordability": "₹50,000 budget buys ~16 shares; allows partial diversification if split.",
        "risk": "Moderate — large-cap, but capex-heavy balance sheet."
      },
      "news_summary": "1-3 sentence summary of recent news, own words, no verbatim quotes.",
      "catalysts": [
        {"description": "New energy giga-factory commissioning", "expected_date": "H1 2027", "source": "company press release via <outlet>"}
      ],
      "filings": [
        {"type": "Board meeting / capex approval", "date": "2026-10-15", "summary": "1-line summary", "source": "BSE announcement / <outlet covering it>"}
      ],
      "timing": {
        "intraday": {
          "entry_zone_inr": "2930-2945",
          "exit_or_stop_inr": "2890 stop / 3000 target",
          "caveat": "Levels from last available search data — confirm live chart before acting; markets may have moved since this was generated."
        },
        "long_term_holding_period": "12-24 months, tied to capex ramp reflecting in earnings by FY28"
      },
      "sources_used": ["bseindia.com", "moneycontrol.com", "groww.in"]
    }
  ],
  "recommendation": {
    "ranked": ["RELIANCE", "TCS", "INFY"],
    "budget_allocation_reasoning": "1-3 sentences: concentrate vs split, and why, given scores and affordability."
  }
}
```

Notes:
- Use `null` (not a guess) for any field you could not verify via search.
- `price_as_of` must always state whether the price is live-session, previous close, or delayed, since there is no live data connector.
- Keep `news_summary` and filing summaries in your own words — never paste verbatim text from search results (copyright).
