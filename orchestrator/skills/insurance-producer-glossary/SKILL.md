---
name: insurance-producer-glossary
description: >-
  Insurance producer / distribution domain vocabulary for interpreting questions
  over the license, appointment, contract, and demographics datasets. Use when a
  question contains insurance terms (NPN, LOA, resident vs non-resident license,
  carrier appointment, just-in-time appointment, termination for cause, selling
  contract, hierarchy, NIPR/PDB) or when you must decide which dataset a question
  refers to. Provides definitions, the producer lifecycle, and how entities join.
  Look up live values with lookup_glossary / describe_column; this skill supplies
  the priors.
---

# Insurance producer glossary

Ground every question in the **producer lifecycle**. This skill gives you the
vocabulary and the relationships; the live catalog (`lookup_glossary`,
`describe_table`, `describe_column`) remains authoritative for exact values and
status enums.

## The four core datasets

- **Demographics** — who the producer is. Individual or agency/firm, identified
  by **NPN** (National Producer Number). Holds identity + contact data; **much of
  it is PII** (SSN/TIN, full DOB, home address). The anchor that the other three
  datasets join to.
- **License** — *state* authorization to transact insurance. A producer can hold
  many licenses (one per state, often several lines). Resident vs non-resident,
  carries **lines of authority (LOA)**, has issue / expiration dates and a
  status.
- **Appointment** — *carrier* authorization for a producer to sell that carrier's
  products in a state. Distinct from a license: you can be licensed but not
  appointed. Carriers must report appointments and terminations to state DOIs.
- **Contract** — the *commercial* agreement (a.k.a. selling/producer agreement):
  commission schedule, hierarchy/upline, and contract status. Governs how the
  producer gets paid and under whom they write.

A producer is typically **licensed** (state) → **appointed** (carrier) →
**contracted** (commercial) before they can sell and be paid.

## How they relate

- One **producer (NPN)** → many **licenses** (by state/LOA).
- One **producer** → many **appointments** (by carrier × state × LOA).
- One **producer** → one or many **contracts** (by carrier/distributor, with
  hierarchy).
- **License ⟷ appointment**: an appointment usually presupposes a valid license
  for the same state + line; a lapsed license can invalidate an appointment.

Always use the catalog's **canonical joins** — never invent a join between these
tables.

## Reference files (load on demand)

- [`references/domain-terms.md`](references/domain-terms.md) — full term
  glossary (NPN, LOA, resident/non-resident, JIT appointment, termination
  reasons, hierarchy, NIPR/PDB, etc.).
- [`references/status-and-date-semantics.md`](references/status-and-date-semantics.md)
  — typical status enums per dataset and date/"active" semantics, with the
  pitfalls that most often cause wrong answers.

## Common interpretation traps

- **"Active" is overloaded.** Active *license* ≠ active *appointment* ≠ active
  *contract*. Confirm which dataset the user means before matching a workflow.
- **"Appointed" ≠ "licensed."** A producer can be licensed in a state but not
  appointed with a given carrier (and vice-versa in just-in-time states).
- **Producer vs agency.** Questions about "agents" may mean individuals, the
  agency entity, or both. Check entity type.
- **Resident vs non-resident.** A producer has at most one resident license
  (home state) and many non-resident licenses; counts differ depending on which
  you include.
- **Expiration vs termination.** Licenses *expire / lapse*; appointments and
  contracts are *terminated* (with a reason, sometimes "for cause"). Don't
  conflate them.
