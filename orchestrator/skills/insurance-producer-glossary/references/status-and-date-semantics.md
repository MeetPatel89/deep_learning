# Status and date semantics

Typical status families and date logic per dataset, plus the pitfalls that most
often produce wrong answers. **These are priors** — confirm the exact enum values
for your catalog with `lookup_glossary` or `describe_column`.

## Status families (typical)

| Dataset | Common statuses | Notes |
| --- | --- | --- |
| License | active, inactive, expired/lapsed, pending, suspended, revoked | "active" ≠ "not expired"; a license can be inactive without being expired. |
| Appointment | active, pending, terminated | termination carries a *reason* (voluntary vs for-cause). |
| Contract | active, pending, terminated, suspended | tied to commissions/hierarchy, not to state authority. |
| Demographics | (entity status) active, inactive | governs whether the producer record itself is current. |

Each status word can differ across datasets — always scope "active" to the
dataset the user means (see the glossary's interpretation traps).

## Date semantics

- **License validity** = `effective_date` ≤ as-of ≤ `expiration_date` **and**
  status indicates active. "Expiring soon" = `expiration_date` within a window
  from today.
- **Appointment validity** = `effective_date` ≤ as-of ≤ `termination_date`
  (or `termination_date` is null) **and** status active.
- **Contract validity** = analogous effective/termination window plus status.
- **As-of / point-in-time**: current-state questions hit Azure SQL; historical
  "as of <past date>" or trend questions usually hit the lake (ADLS) snapshots.

## High-value pitfalls

- **Soft-deletes**: rows may carry `is_active = 0` or a deleted flag — exclude
  them unless the user explicitly wants historical/terminated records. The
  catalog's `default_filters` usually encodes this; trust them.
- **Null termination/expiration** often means "still active" — do not treat null
  as "ended".
- **Status vs date disagreement**: a row can be `status = active` but past
  `expiration_date` due to upstream lag. Prefer the workflow's blessed
  definition; flag the discrepancy rather than picking silently.
- **Counting resident + non-resident together** inflates "how many licenses"
  answers — clarify which the user wants.
- **Multi-LOA licenses**: one license row may carry several LOAs; counting "life
  licenses" by row vs by LOA gives different numbers.
- **Producer vs appointment grain**: "how many appointments" (carrier×state×LOA
  rows) is not "how many appointed producers" (distinct NPNs).
