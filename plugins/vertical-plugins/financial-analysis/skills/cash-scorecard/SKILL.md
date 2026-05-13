---
skill: cash-scorecard
description: Format a completed Cash Score JSON into a published scorecard — an Excel workbook (.xlsx) and a Markdown narrative (.md).
triggers:
  - generate scorecard
  - produce scorecard
  - write scorecard
  - format cash score
  - cash report
---

# Cash Scorecard

## Purpose

Take the structured JSON output from `cash-score-model` and produce two human-readable artifacts:

1. `./out/cash-score-<TICKER>.xlsx` — a multi-tab Excel scorecard
2. `./out/cash-score-<TICKER>.md` — a one-page Markdown narrative

Both files are staged for analyst review. Neither is distributed externally without sign-off.

---

## 1. Excel Scorecard (.xlsx)

### Workbook Structure (6 tabs)

| Tab | Name | Purpose |
|---|---|---|
| 1 | **Summary** | Composite score gauge + five dimension bars + key metrics table |
| 2 | **Cash Yield** | D1 detail: FCF yield, P/FCF, peer comp table |
| 3 | **Cash Conversion** | D2 detail: FCF/NI, FCF/EBITDA, OCF margin trend (3yr) |
| 4 | **Cash Efficiency** | D3 detail: CCC waterfall (DSO, DIO, DPO), YoY change |
| 5 | **Cash Stability** | D4 detail: FCF history bar chart + CAGR + volatility |
| 6 | **Cash Deployment** | D5 detail: ROIC vs. WACC, CapEx/Revenue, returns table |

### Tab 1 (Summary) — Required Elements

```
Row 1:  Company name, Ticker, Period, Score Date
Row 3:  CASH SCORE: [score]/100  [grade badge]
Row 5:  Score gauge — horizontal bar 0–100, color-coded
Row 8:  Dimension breakdown table
Row 15: Key metrics panel (FCF, FCF Yield, FCF Margin, FCF/NI, CCC)
Row 22: Bull cash thesis (text box)
Row 28: Bear cash thesis (text box)
Row 34: Data sources and disclaimer
```

### Color Coding

| Score Range | Grade | Hex |
|---|---|---|
| 80–100 | Elite | #1A7A4A |
| 65–79 | Strong | #4CAF50 |
| 50–64 | Average | #FFC107 |
| 35–49 | Weak | #FF5722 |
| 0–34 | Poor | #D32F2F |

---

## 2. Markdown Scorecard (.md)

### Template

```markdown
# Cash Score: {TICKER} — {SCORE}/100 ({GRADE})
*Period: {PERIOD} | Generated: {DATE}*

---

## The Verdict

{TICKER} earns a Cash Score of **{SCORE}/100 ({GRADE})**. {ONE_SENTENCE_SUMMARY}.

## Scorecard

| Dimension | Score | Key Metric |
|---|---|---|
| Cash Yield | {D1}/20 | FCF Yield: {fcf_yield}%, P/FCF: {p_fcf}× |
| Cash Conversion | {D2}/20 | FCF/NI: {fcf_ni}×, OCF Margin: {ocf_margin}% |
| Cash Efficiency | {D3}/20 | CCC: {ccc} days |
| Cash Stability | {D4}/20 | FCF CAGR (3yr): {fcf_cagr}%, Volatility: {fcf_vol} |
| Cash Deployment | {D5}/20 | ROIC: {roic}%, % FCF returned: {shareholder_return}% |

## The Bull Case on Cash

{bull_cash_thesis}

## The Bear Case on Cash

{bear_cash_thesis}

---
*All figures sourced from {primary_source}. For review purposes only — not for external distribution without analyst sign-off.*
```

### Writing Standards for Thesis Sections

- **Bull cash thesis**: 2–3 sentences. Lead with the strongest cash quality signal. Cite at least one specific metric.
- **Bear cash thesis**: 2–3 sentences. Identify the dimension with the lowest score. Include at least one risk scenario.
- **One-sentence summary**: Non-hedging. Take a position.

---

## 3. QC Before Writing

1. All `{PLACEHOLDER}` tokens are resolved
2. Numbers formatted consistently (percentages as "x.x%", multiples as "x.x×")
3. The composite score in the header matches the sum of dimension scores
4. Bull and bear theses are non-contradictory
5. Both artifacts have been produced: `.xlsx` and `.md`

Write both files atomically — produce both or neither.

---

## 4. Output Paths

```
./out/cash-score-{TICKER}.xlsx
./out/cash-score-{TICKER}.md
```

Lowercase ticker. Replace `/` or `.` with `-` (e.g., `brk-b` not `brk.b`).
