---
name: result-presentation-and-provenance
description: >-
  Conventions for presenting answers from the semantic layer: lead with the
  answer, render row sets as tables, disclose data freshness and truncation, cite
  the workflow and execution MCP, and never fabricate numbers. Use whenever you
  report results of a rendered workflow over the insurance license, appointment,
  contract, or demographics datasets, or when a result was truncated, empty, or
  came from a stale catalog.
---

# Result presentation and provenance

How to turn retrieved rows into a trustworthy, auditable answer. Applies after
`semantic-layer-routing` has executed a workflow.

## Lead with the answer

- State the direct answer first (the number, the yes/no, the short list), then
  supporting detail. Don't bury the result under method.
- For row sets, render a **markdown table** with the workflow's
  `output_columns` and their business meanings as headers. Keep to the columns
  that answer the question.

## Always cite provenance

End data answers with a compact provenance line, e.g.:

> Source: workflow `licenses_expiring_in_window` via Azure SQL MCP · catalog
> v`<version>` · data as of `<materialized_at>`.

Include: **workflow id**, **execution MCP** (Azure SQL or ADLS), **catalog
version**, and **freshness timestamp** when time-sensitive.

## Disclose freshness

- For time-sensitive answers (expirations, terminations, recent appointments),
  state `materialized_at` from `catalog_status()`.
- If the cache is stale (older than ~2x the sync cadence) or a tool response
  flagged staleness, **say so explicitly** and offer to re-check after a refresh.

## Disclose truncation

- If the result hit a row or byte cap (`truncated: true`), say the result is
  partial, give the cap, and offer to narrow the filter (tighter date window,
  add state/LOA/carrier) rather than implying completeness.
- Never extrapolate or invent the rows you didn't receive.

## Empty results

- If a query returns zero rows, say so plainly and state the filters applied, so
  the user can tell "truly none" from "filters too narrow." Suggest the most
  likely loosening (e.g. include non-resident, widen the date window).

## Never fabricate

- Report only values you retrieved. Do not estimate counts, infer statuses, or
  fill gaps from prior knowledge.
- If two sources disagree (e.g. status vs expiration date, or SQL vs lake),
  surface the discrepancy instead of silently choosing; offer `recon_sql_vs_lake`
  if relevant.

## PII in output

- Follow `pii-and-compliance`: prefer NPN, mask high-sensitivity fields, prefer
  aggregates. Don't widen the output beyond what the workflow returned.

## Offer the next step

- Suggest the workflow's `followups` (e.g. after a license check, offer
  `appointments_for_producer`). Keep it to one or two relevant options.

## Template

```
<direct answer>

| <col: meaning> | <col: meaning> | … |
| --- | --- | --- |
| …             | …             | … |

[only if relevant] Note: result truncated at <cap> rows — narrow by <suggestion>.
Source: workflow `<id>` via <Azure SQL|ADLS> MCP · catalog v<version> · as of <ts>.
Next: <followup workflow> ?
```
