---
skill: cash-score-model
description: Apply the five-dimension Cash Score rubric to produce a 0–100 composite score for any public company.
triggers:
  - cash score
  - score this company
  - cash quality
  - rate the cash
  - cash rating
---

# Cash Score Model

## Philosophy

Money is the only game every company on the planet is playing. The scorecard is Cash — not earnings, not revenue, not adjusted EBITDA. Cash is the ultimate truth test: you either generate it or you don't.

The Cash Score is a 0–100 composite rating of cash quality. It measures five things: how much cash a company generates relative to its price (yield), how efficiently it converts accounting profits into actual cash (conversion), how tightly it manages its working capital cycle (efficiency), how consistently it generates cash year over year (stability), and how wisely it deploys the cash it generates (deployment).

---

## Scoring Framework

| Dimension | Max Points | Primary Question |
|---|---|---|
| Cash Yield | 20 | Is the company cheap relative to its cash generation? |
| Cash Conversion | 20 | Does accounting income translate to real cash? |
| Cash Efficiency | 20 | How tight is the working capital cycle? |
| Cash Stability | 20 | Is cash generation consistent and growing? |
| Cash Deployment | 20 | Is the company investing its cash wisely? |
| **Total** | **100** | |

Score interpretation: 80–100 = Elite, 65–79 = Strong, 50–64 = Average, 35–49 = Weak, 0–34 = Poor

---

## Dimension 1: Cash Yield (0–20 pts)

Measures how much free cash flow the company generates per dollar of market value. Higher FCF yield = better value proposition.

### Inputs
- `fcf_yield` (FCF / Market Cap)
- `fcf_yield_percentile` (percentile rank within peer universe)
- `p_fcf` (Price / FCF multiple)

### Scoring Table

| FCF Yield Percentile | Points |
|---|---|
| 80th–100th (top quintile) | 18–20 |
| 60th–79th | 14–17 |
| 40th–59th | 10–13 |
| 20th–39th | 6–9 |
| 0th–19th (bottom quintile) | 0–5 |

Fine-tune within each band using the absolute FCF yield:
- FCF yield > 7%: add 2 pts to the band score
- FCF yield < 1%: subtract 2 pts
- Negative FCF yield: score = 0 for this dimension

### Formula
```
D1_score = percentile_to_band_score(fcf_yield_percentile) + yield_adjustment(fcf_yield)
D1_score = max(0, min(20, D1_score))
```

---

## Dimension 2: Cash Conversion (0–20 pts)

Measures how effectively accounting profits convert into actual cash.

### Inputs
- `fcf_conversion` = FCF / Net Income
- `fcf_ebitda` = FCF / EBITDA
- `ocf_margin` = CFO / Revenue

### Scoring Table

| FCF/NI | FCF/EBITDA | OCF Margin | Points |
|---|---|---|---|
| ≥ 1.0 | ≥ 0.70 | ≥ 20% | 17–20 |
| 0.75–0.99 | 0.50–0.69 | 15–19% | 12–16 |
| 0.50–0.74 | 0.35–0.49 | 10–14% | 8–11 |
| 0.25–0.49 | 0.20–0.34 | 5–9% | 4–7 |
| < 0.25 or negative | < 0.20 | < 5% | 0–3 |

```
D2_score = 0.50 × score(fcf_conversion) + 0.30 × score(fcf_ebitda) + 0.20 × score(ocf_margin)
```

**Edge cases:**
- If Net Income is negative but FCF is positive: `fcf_conversion` = N/A; use FCF/EBITDA and OCF margin only (reweight to 60/40)
- If EBITDA is negative: use only OCF margin; apply a 30% discount to D2 max

---

## Dimension 3: Cash Efficiency (0–20 pts)

```
CCC = DSO + DIO − DPO
```

Where:
- **DSO** = Days Sales Outstanding = (AR / Revenue) × 365
- **DIO** = Days Inventory Outstanding = (Inventory / COGS) × 365
- **DPO** = Days Payable Outstanding = (AP / COGS) × 365

### Scoring Table

| CCC | Points |
|---|---|
| < −30 days | 18–20 |
| −30 to 0 days | 14–17 |
| 0 to 30 days | 10–13 |
| 30 to 60 days | 6–9 |
| > 60 days | 0–5 |

**YoY change bonus:** If CCC improved by > 5 days YoY, add 2 pts. If it worsened by > 5 days, subtract 2 pts.

---

## Dimension 4: Cash Stability (0–20 pts)

### Inputs
- `fcf_cagr_3yr` = 3-year FCF compound annual growth rate
- `fcf_volatility` = standard deviation of annual FCF / mean FCF (coefficient of variation)

**Sub-score A: FCF Growth (0–10 pts)**
| FCF CAGR | Points |
|---|---|
| > 20% | 9–10 |
| 10–20% | 7–8 |
| 5–10% | 5–6 |
| 0–5% | 3–4 |
| Negative | 0–2 |

**Sub-score B: FCF Consistency (0–10 pts)**
| FCF Volatility (CoV) | Points |
|---|---|
| < 0.15 | 9–10 |
| 0.15–0.30 | 7–8 |
| 0.30–0.50 | 5–6 |
| 0.50–0.75 | 3–4 |
| > 0.75 | 0–2 |

```
D4_score = Sub_A + Sub_B
```

---

## Dimension 5: Cash Deployment (0–20 pts)

### Inputs
- `roic` = Return on Invested Capital
- `capex_intensity` = CapEx / Revenue
- `shareholder_return_rate` = (Buybacks + Dividends) / FCF

**Sub-score A: ROIC Quality (0–8 pts)**
| ROIC | Points |
|---|---|
| > 25% | 7–8 |
| 15–25% | 5–6 |
| 10–15% | 3–4 |
| 5–10% | 1–2 |
| < 5% | 0 |

**Sub-score B: CapEx Discipline (0–6 pts)** — compare vs. peer median and own 3yr avg

**Sub-score C: Shareholder Returns (0–6 pts)**
| % FCF returned | Points |
|---|---|
| 60–100% | 5–6 |
| 30–59% | 3–4 |
| 10–29% | 1–2 |
| < 10% | 0 |
| > 100% (debt-funded) | 1 (flag) |

```
D5_score = Sub_A + Sub_B + Sub_C
```

---

## Composite Score Assembly

```
Cash_Score = D1_score + D2_score + D3_score + D4_score + D5_score
```

If any dimension is N/A: `Cash_Score = (sum of available scores / max available points) × 100`

---

## Output Schema

```json
{
  "ticker": "AAPL",
  "period": "TTM",
  "cash_score": 78,
  "grade": "Strong",
  "dimensions": {
    "cash_yield":      { "score": 15, "max": 20 },
    "cash_conversion": { "score": 17, "max": 20 },
    "cash_efficiency": { "score": 20, "max": 20 },
    "cash_stability":  { "score": 13, "max": 20 },
    "cash_deployment": { "score": 13, "max": 20 }
  },
  "bull_cash_thesis": "...",
  "bear_cash_thesis": "...",
  "flags": [],
  "scoring_version": "1.0"
}
```

---

## QC Checks

1. Sum of dimension scores equals composite Cash_Score
2. All inputs match the `cash-flow-analysis` output
3. No dimension score exceeds its maximum (20)
4. Grade label matches the composite score band
5. Bull/bear thesis cites at least one specific metric
