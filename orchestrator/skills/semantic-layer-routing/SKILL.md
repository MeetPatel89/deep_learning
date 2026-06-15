---
name: semantic-layer-routing
description: >-
  Protocol for answering insurance data questions through the Semantic MCP
  instead of writing raw SQL. Use whenever the user asks about producer
  licenses, carrier appointments, distribution contracts, or producer
  demographics, or requests any data pull, count, list, trend, or lookup over
  the Azure SQL / ADLS datasets. Covers the match -> validate -> render ->
  execute loop, choosing the execution MCP, disambiguation, parameter
  self-correction, no-match fallback, and freshness checks.
---

# Semantic-layer routing

Answer data questions by selecting a vetted **workflow** from the Semantic MCP
and dispatching its rendered query to the right execution MCP. Do not author SQL
from scratch when a workflow exists.

## The loop

```
match_workflow  ->  validate_parameters  ->  render_workflow  ->  execute  ->  answer
   (Semantic)          (Semantic)              (Semantic)        (SQL/ADLS)
```

### 1. Understand
Read the question in producer-lifecycle terms (license / appointment / contract /
demographics). If a business word is ambiguous (e.g. "active", "appointed",
"terminated"), call `lookup_glossary(term)` before matching — the same word means
different things on different tables.

### 2. Match
Call `match_workflow(question)`. Use the returned confidence:

- **Single strong match** → continue with it.
- **Top-K are close together** → do not guess. Ask the user **one** crisp
  disambiguation question that distinguishes the candidates (see below).
- **Nothing acceptable** → go to "No-match fallback".

If you already know the workflow id (e.g. a follow-up), you may skip straight to
`describe_workflow(id)` to confirm its parameters.

### 3. Fill & validate
Extract parameters from the question. Then **always** call
`validate_parameters(id, params)` before rendering — it is the firewall against
parameter hallucination.

- If validation returns structured errors, **self-correct and re-validate**:
  - enum miss → map the user's word to an allowed value via the glossary;
  - bad date order → swap or re-parse `start_date` / `end_date`;
  - missing required → infer a sensible default *only* if the workflow allows
    one; otherwise ask the user for exactly that value.
- Do not call `render_workflow` until validation passes.

### 4. Render
Call `render_workflow(id, params)`. It returns
`{ rendered_query, target_mcp, target_tool_call, output_schema,
postprocessing_hints }`.

- Treat `rendered_query` as **immutable**. Never tweak the SQL, add columns, or
  change filters by hand — that defeats governance and breaks reconciliation.
- The `target_mcp` tells you where to send it.

### 5. Execute
Dispatch `target_tool_call` to the indicated MCP:

- `target_mcp: azure_sql` → the **Azure SQL MCP** (operational/current state:
  current license status, active appointments, contract terms).
- `target_mcp: adls` → the **ADLS Gen 2 MCP** (history, point-in-time snapshots,
  large extracts, SQL-vs-lake reconciliation).

Pass parameters through exactly as rendered. One execution call per rendered
query.

### 6. Answer
Hand off to `result-presentation-and-provenance`: lead with the answer, render
row sets as tables, and cite the workflow id, execution MCP, catalog version, and
freshness. Offer the workflow's `followups`.

## Disambiguation (one question, not many)

When candidates are close, ask the single question that splits them. Examples:

- License vs appointment: *"Do you mean the producer's state **license** status,
  or their **appointment** with a specific carrier?"*
- Point-in-time vs current: *"As of today, or as of a specific date?"*
- Producer vs agency: *"The individual producer, or the agency/firm they write
  under?"*

Prefer a multiple-choice phrasing so the user can answer in one word.

## No-match fallback

1. State plainly that no vetted workflow matched.
2. If the question is **definitional** ("what does LOA mean?", "which columns are
   on the license table?"), answer with `describe_table`, `describe_column`,
   `describe_metric`, or `lookup_glossary` — no execution needed.
3. If it is a genuine data gap, offer to log it for a new workflow.
4. Only write free-form SQL if it is explicitly enabled for the session, and even
   then announce that you are leaving the governed path.

## Freshness

Call `catalog_status()` and surface `catalog_version` + `materialized_at` when:

- the answer is time-sensitive (expirations, terminations, recently added
  appointments), or
- a tool response warns the cache is stale (older than ~2x the sync cadence).

If stale, answer with the data you have **and** flag the staleness explicitly.

## Anti-patterns (do not do these)

- Writing or editing SQL when a workflow exists, or editing a rendered query.
- Inventing table/column/join/status names instead of looking them up.
- Skipping `validate_parameters` and rendering on a guess.
- Executing against the wrong MCP (e.g. asking Azure SQL for deep history that
  lives in the lake).
- Asking the user three clarifying questions when one would do.
- Reporting numbers without provenance or freshness on time-sensitive answers.

## Worked example

User: *"Which producers have a resident life license in Texas that expires in the
next 60 days?"*

1. `lookup_glossary("resident license")`, `lookup_glossary("life LOA")` if unsure.
2. `match_workflow(...)` → `licenses_expiring_in_window` (high confidence).
3. params: `{ state: "TX", license_class: "resident", loa: "life",
   window_days: 60 }` → `validate_parameters` → ok.
4. `render_workflow(...)` → `target_mcp: azure_sql`.
5. Execute via Azure SQL MCP.
6. Answer as a table (producer NPN, name, license #, expiration date), cite
   workflow + catalog version + `materialized_at`, and offer follow-up
   `appointments_for_producer`.
