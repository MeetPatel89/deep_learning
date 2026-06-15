---
name: producer-data-workflows
description: >-
  Maps common insurance producer business questions to semantic-layer workflows
  and their parameters, and extracts parameters (state codes, lines of authority,
  statuses, carriers, date windows, NPNs) from natural language. Use when a user
  asks a recurring operational question about licenses, appointments, contracts,
  or demographics (e.g. "is X licensed in TX", "appointments expiring in 30
  days", "contract status by carrier", "terminations last quarter") and you need
  to pick the right workflow and fill it. Pair with semantic-layer-routing.
---

# Producer data workflows

Bridge between a business question and the Semantic MCP. This skill helps you (a)
guess the right workflow before/with `match_workflow`, and (b) extract clean
parameters that pass `validate_parameters` on the first try. **Always confirm the
real workflow id and parameter spec with `match_workflow` / `describe_workflow`**
— ids below are indicative, mapped to the seeds in the plan doc.

## Question → workflow map (indicative)

| The user is asking about… | Likely workflow family | Key parameters |
| --- | --- | --- |
| Is a producer licensed in a state / line? | `entity_lookup_by_id` + license filter | `npn`, `state`, `loa` |
| Licenses expiring / lapsing soon | `licenses_expiring_in_window` (count/range) | `state?`, `loa?`, `window_days` or `start/end` |
| Active appointments for a producer | `appointments_for_producer` | `npn`, `as_of?`, `carrier?` |
| Appointments / terminations in a period | `count_by_status_in_range` | `dataset=appointment`, `status`, `start`, `end`, `group_by?` |
| Terminations "for cause" last quarter | `count_by_status_in_range` + reason filter | `status=terminated`, `reason=for_cause`, period |
| Contract status by carrier | `count_by_status_in_range` / `top_n_by_metric` | `dataset=contract`, `group_by=carrier` |
| Producer 360 / full context | `entity_lookup_by_id` | `npn` |
| Status change history for a producer | `entity_timeline` | `npn`, `dataset`, `start`, `end` |
| How many appointed producers in a state | `count_by_status_in_range` (distinct NPN) | `dataset=appointment`, `state`, `as_of` |
| Period-over-period (new appointments) | `period_over_period` | `metric`, `period`, `prior` |
| SQL vs lake reconciliation | `recon_sql_vs_lake` | `table`, period |
| Data freshness for a feed | `freshness_check` | `table` |
| Value distribution / sanity check | `value_distribution` | `table`, `column` |

If a question doesn't fit any row, hand back to `semantic-layer-routing`'s
no-match fallback.

## Parameter extraction cheatsheet

- **State** → normalize to the 2-letter USPS code (Texas → `TX`). "All states" →
  omit the state param (don't pass a wildcard the workflow doesn't define).
- **Line of authority (LOA)** → map phrases to catalog enums: "life" → `life`,
  "health"/"A&H" → `accident_health`, "P&C" → expand to both `property` and
  `casualty` if the workflow takes a list; otherwise ask which.
- **Status** → map the user's word to the dataset's enum (see the glossary's
  status-and-date reference). "Lapsed" → license `expired`/`lapsed`;
  "cancelled" → appointment/contract `terminated`.
- **Carrier** → resolve to the catalog's carrier identifier; if the name is
  ambiguous (multiple legal entities), ask one disambiguating question.
- **Producer** → prefer **NPN**. If given only a name, expect possible multiple
  matches and surface them rather than guessing.
- **Date windows** → resolve relative phrases to explicit dates and pass the
  shape the workflow expects:
  - "next 30/60/90 days" → `window_days` or `start=today, end=today+N`.
  - "last quarter", "QTD", "YTD", "MTD" → resolve against the catalog's fiscal vs
    calendar definition (`lookup_glossary` for `dates`).
  - Always enforce `end >= start`.
- **Resident vs non-resident** → if unspecified for a license count, ask which to
  include; do not silently sum both.

## Worked mappings

1. *"Is producer 1234567 licensed for health in Florida?"*
   → `entity_lookup_by_id` (or a license-check workflow); params `npn=1234567`,
   `state=FL`, `loa=accident_health`. Answer yes/no + the license row.

2. *"How many appointments did we terminate for cause last quarter, by carrier?"*
   → `count_by_status_in_range`; `dataset=appointment`, `status=terminated`,
   `reason=for_cause`, period = last calendar quarter, `group_by=carrier`.
   Confirm "quarter" = calendar vs fiscal via glossary.

3. *"Show non-resident life licenses expiring in the next 60 days in CA."*
   → `licenses_expiring_in_window`; `state=CA`, `license_class=non_resident`,
   `loa=life`, `window_days=60`. Note the resident/non-resident choice was
   explicit — good.

## Reminders

- These mappings are **hints**. The catalog is authoritative — validate before
  rendering, and let `match_workflow` override a guess.
- Never fabricate a workflow id; if the mapped family doesn't exist in
  `list_workflows`, treat it as a no-match.
