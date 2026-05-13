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

Take the structured JSON output from `cash-score-model` and produce two artifacts:

1. `./out/cash-score-<TICKER>.xlsx` — six-tab Excel scorecard
2. `./out/cash-score-<TICKER>.md` — one-page Markdown narrative

Both files staged for analyst review. Neither distributed externally without sign-off.

See the vertical-plugins source at `plugins/vertical-plugins/financial-analysis/skills/cash-scorecard/SKILL.md` for the full output spec, tab layout, color coding, Markdown template, and QC checklist.
