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
```
FCF (SBC-adjusted) = Standard FCF − SBC expense
```
Flag when SBC/Revenue > 5%.

### 3.2 Capitalized Software Development Costs
```
Adjusted CapEx = Reported CapEx + Capitalized Software Dev Costs
```

### 3.3 Working Capital Distortions
- Compare CCC vs. 3-year average
- Flag if WC change > ±15% of CFO

### 3.4 Lease Payments (Post-ASC 842)
```
FCF (lease-adjusted) = Standard FCF − Operating Lease Payments
```

---

## 4. Key Derived Metrics

```
Operating CF Margin    = CFO / Revenue
FCF Margin             = FCF / Revenue
FCF Conversion         = FCF / Net Income
FCF / EBITDA           = FCF / EBITDA
FCF Yield              = FCF / Market Cap
P/FCF                  = Market Cap / FCF
FCF CAGR (3yr)         = (FCF_t / FCF_t-3)^(1/3) − 1
FCF YoY Growth         = (FCF_t / FCF_t-1) − 1
FCF Volatility (σ)     = StdDev of annual FCF / mean FCF (3yr window)
```

---

## 5. Output Schema

Return a validated JSON object matching the cash-flow-analysis output schema. All monetary values in absolute USD. Ratios as decimals. Mark any unverified field as `null` with an entry in `unsourced_fields`.

---

## 6. Guardrails

- **Never compute FCF from earnings alone.**
- **Negative FCF is valid data** — do not substitute zero.
- **Financials and banks**: flag `sector_exception: true`.
- **All figures must cite a specific source.**
