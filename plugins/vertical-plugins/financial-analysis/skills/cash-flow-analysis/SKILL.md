---
skill: cash-flow-analysis
description: Extract, validate, and normalize free cash flow (FCF) and related cash metrics from public company filings and financial data providers.
triggers:
  - cash flow
  - free cash flow
  - FCF
  - operating cash flow
  - capex
  - cash from operations
---

# Cash Flow Analysis

## Purpose

Extract and normalize the raw cash flow inputs needed to compute a company's Cash Score. All figures must be sourced from verified data — FactSet, Morningstar, or the company's SEC filings — and tagged with their source reference before being passed downstream.

---

## 1. Free Cash Flow (FCF) — Canonical Definition

```
FCF = Cash from Operations (CFO) − Capital Expenditures (CapEx)
```

**Variants and when to use them:**

| Variant | Formula | Use case |
|---|---|---|
| **Levered FCF** | CFO − CapEx | Standard; default for all scoring |
| **Unlevered FCF (FCFF)** | EBIT × (1−t) + D&A − ΔNWC − CapEx | Cross-company comparison; normalize out capital structure |
| **FCF to Equity (FCFE)** | CFO − CapEx + Net Borrowings | PE or buyout context |

Always use **Levered FCF** as the default unless the analyst specifies otherwise.

---

## 2. Where to Find Each Line Item

### From SEC Filings (10-K / 10-Q)

| Line Item | Statement | XBRL Tag |
|---|---|---|
| Cash from Operations | Cash Flow Statement | `us-gaap:NetCashProvidedByUsedInOperatingActivities` |
| Capital Expenditures | Cash Flow Statement | `us-gaap:PaymentsToAcquirePropertyPlantAndEquipment` |
| Depreciation & Amortization | CF Statement or Notes | `us-gaap:DepreciationDepletionAndAmortization` |
| Stock-Based Compensation | CF Statement (non-cash add-back) | `us-gaap:ShareBasedCompensation` |
| Working Capital Changes | CF Statement | Sum of operating WC line items |
| Net Income | Income Statement | `us-gaap:NetIncomeLoss` |

### From FactSet
- **CFO**: `FF_CFO_NET` (trailing twelve months available)
- **CapEx**: `FF_CAPEX` (use negative convention — absolute value)
- **FCF**: `FF_FREE_CF` (pre-computed; validate against manual calc)
- **FCF Yield**: `FF_FCF_YIELD` (FCF / Market Cap)

### From Morningstar
- **Normalized FCF**: available in the financial summary; uses Morningstar's adjusted CapEx
- **Cash Flow Statement**: accessible via the financials endpoint; 5-year history standard

---

## 3. Normalization Adjustments

Apply these adjustments before scoring. Each adjustment must be documented with its rationale.

### 3.1 Stock-Based Compensation (SBC)
SBC is a real economic cost even though it's a non-cash add-back in the CF statement. For tech companies especially:

```
FCF (SBC-adjusted) = Standard FCF − SBC expense
```

Flag when SBC/Revenue > 5% — significant dilution risk.

### 3.2 Capitalized Software Development Costs
Some companies capitalize software dev costs as CapEx rather than expensing them. This inflates CFO and understates "true" CapEx.

```
Adjusted CapEx = Reported CapEx + Capitalized Software Dev Costs
```

Check the investing activities section for capitalized software line items.

### 3.3 Working Capital Distortions
One-time working capital swings (large inventory builds, unusual receivables collections) can distort trailing FCF.

- Compare CCC (Cash Conversion Cycle) vs. 3-year average
- Flag if WC change > ±15% of CFO

### 3.4 Lease Payments (Post-ASC 842)
Since 2019, operating lease payments flow through financing activities (not operating). For fair comparison of pre/post-ASC 842 periods:

```
FCF (lease-adjusted) = Standard FCF − Operating Lease Payments
```

Only apply when explicitly comparing across the 2019 accounting change boundary.

---

## 4. Key Derived Metrics

Compute all of the following and pass them in the output JSON:

```
Operating CF Margin    = CFO / Revenue
FCF Margin             = FCF / Revenue
FCF Conversion         = FCF / Net Income  (flag if > 1.5× or < 0)
FCF / EBITDA           = FCF / EBITDA
FCF Yield              = FCF / Market Cap
P/FCF                  = Market Cap / FCF  (inverse of FCF Yield)
FCF CAGR (3yr)         = (FCF_t / FCF_t-3)^(1/3) − 1
FCF YoY Growth         = (FCF_t / FCF_t-1) − 1
FCF Volatility (σ)     = Standard deviation of annual FCF / mean FCF (3yr window)
```

---

## 5. Peer Universe Construction

For FCF Yield percentile ranking, construct a peer universe:

1. Pull the company's GICS sub-industry from FactSet
2. Retrieve all peers with Market Cap within 0.2×–5× of the subject company
3. Minimum 5 peers; maximum 20 peers
4. Compute FCF Yield and P/FCF for each peer
5. Rank the subject company; record percentile (0–100)

---

## 6. Output Schema

Return a validated JSON object:

```json
{
  "ticker": "AAPL",
  "period": "TTM",
  "source_date": "2025-Q1",
  "cfo": 118543000000,
  "capex": 10959000000,
  "fcf": 107584000000,
  "sbc": 11688000000,
  "fcf_sbc_adjusted": 95896000000,
  "revenue": 391035000000,
  "net_income": 93736000000,
  "ebitda": 130000000000,
  "market_cap": 3000000000000,
  "derived": {
    "ocf_margin": 0.303,
    "fcf_margin": 0.275,
    "fcf_conversion": 1.148,
    "fcf_ebitda": 0.827,
    "fcf_yield": 0.036,
    "p_fcf": 27.9,
    "fcf_cagr_3yr": 0.072,
    "fcf_yoy_growth": 0.082,
    "fcf_volatility": 0.12
  },
  "peers": {
    "universe_size": 12,
    "fcf_yield_percentile": 62,
    "p_fcf_percentile": 38
  },
  "ccc": {
    "dso": 28.4,
    "dio": 8.1,
    "dpo": 105.2,
    "ccc": -68.7
  },
  "adjustments_applied": ["sbc_excluded"],
  "data_sources": {
    "cfo": "FactSet FF_CFO_NET",
    "capex": "FactSet FF_CAPEX",
    "peers": "FactSet GICS sub-industry screen"
  }
}
```

All monetary values in absolute USD. Ratios as decimals. Mark any unverified field as `null` with an entry in `unsourced_fields`.

---

## 7. Guardrails

- **Never compute FCF from earnings alone.** Always use the actual cash flow statement.
- **Negative FCF is valid data** — do not substitute zero. Score accordingly.
- **Pre-revenue companies**: set `fcf_conversion` to `null`; note in output.
- **Financials and banks**: CFO is not meaningful for these; flag `sector_exception: true` and use sector-appropriate metrics (NIM, ROE, ROA) instead.
- **All figures must cite a specific source.** No estimates or interpolation without a `[ESTIMATED]` flag.
