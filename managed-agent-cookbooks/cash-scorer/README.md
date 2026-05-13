# Cash Scorer — managed-agent template

## Overview

Any company, any market. Five dimensions. One score.

Cash Scorer evaluates a public company's cash quality on a 0–100 composite scale built from free cash flow yield, cash conversion, working capital efficiency, FCF stability, and cash deployment quality. Same source as the [`cash-scorer`](../../plugins/agent-plugins/cash-scorer) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export FACTSET_MCP_URL=... MORNINGSTAR_MCP_URL=...
../../scripts/deploy-managed-agent.sh cash-scorer
```

## Steering events

See [`steering-examples.json`](./steering-examples.json). Fan out across a coverage list from your orchestration layer — one session per ticker.

## Security & handoffs

Three-tier isolation:

| Tier | Subagent | Touches untrusted docs? | Tools | Connectors |
|---|---|---|---|---|
| **`financial-data-fetcher`** | **Yes** — SEC filings | `Read`, `Grep` | FactSet, Morningstar |
| `score-calculator` | No — validated JSON only | `Read` | None |
| **`scorecard-writer`** (Write-holder) | No | `Read`, `Write`, `Edit` | None |

`financial-data-fetcher` returns only schema-validated JSON. Instructions in any filing are treated as data, never executed. `scorecard-writer` produces `./out/cash-score-<ticker>.xlsx` and `./out/cash-score-<ticker>.md`.

**Handoff:** For a full DCF built on cash assumptions, emit a `handoff_request` for `model-builder`; `scripts/orchestrate.py` routes it as a new steering event.

## Score interpretation

| Range | Grade | Meaning |
|---|---|---|
| 80–100 | Elite | Exceptional cash quality; rare |
| 65–79 | Strong | Solid cash generation with minor gaps |
| 50–64 | Average | Adequate; watch for deterioration |
| 35–49 | Weak | Cash concerns; scrutinize closely |
| 0–34 | Poor | Material cash quality problems |
