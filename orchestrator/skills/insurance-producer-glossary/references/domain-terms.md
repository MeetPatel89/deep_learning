# Producer / distribution domain terms

Definitions to interpret user questions. Treat these as priors; confirm exact
codes and values against the live catalog (`lookup_glossary`, `describe_column`).

## Identity & entities

- **Producer** — an individual or business entity licensed to sell, solicit, or
  negotiate insurance. Umbrella term for agents and agencies.
- **Agent / agency / firm** — *agent* usually = individual producer; *agency* /
  *firm* = business-entity producer. Many agencies have affiliated individual
  producers.
- **NPN (National Producer Number)** — unique, non-PII identifier assigned via
  NIPR; the safest stable key for joining producer records and referring to a
  producer in output.
- **Carrier (company / insurer)** — the insurance company whose products are
  sold. Issues appointments.
- **Distributor / IMO / FMO / BGA / MGA** — intermediary organizations in the
  distribution chain that may hold the contract and a producer hierarchy.

## License

- **License** — state-issued authorization to transact insurance.
- **Resident license** — issued by the producer's home state (typically one).
- **Non-resident license** — issued by other states where the producer does
  business (typically many).
- **License class / type** — e.g. producer, adjuster, surplus lines; jurisdiction
  specific.
- **Line of authority (LOA)** — the product lines a license authorizes: e.g.
  Life, Accident & Health (A&H), Property, Casualty, Personal Lines, Variable
  Life & Variable Annuity. A license carries one or more LOAs.
- **Issue date / effective date / expiration date** — license validity window.
- **Lapsed / expired** — license past its expiration without renewal.

## Appointment

- **Appointment** — carrier's authorization for a producer to sell that carrier's
  products in a state and line. Reported to the state DOI.
- **Appointment effective / termination date** — validity window.
- **Just-in-time (JIT) appointment state** — states allowing a producer to write
  before formal appointment, with appointment filed shortly after the first
  application. Affects "appointed vs eligible" logic.
- **Termination reason** — why an appointment ended; may be administrative
  ("voluntary", "non-payment of fees") or **"for cause"** (misconduct), which
  carries regulatory reporting significance.

## Contract

- **Contract / selling agreement / producer agreement** — commercial agreement
  governing commissions, hierarchy, and the right to write business.
- **Commission schedule / comp plan** — how the producer is paid.
- **Hierarchy / upline / downline** — the reporting/override structure; a
  producer writes "under" an upline; agencies have downlines.
- **Contract status** — active, pending, terminated, suspended (catalog-defined).

## Regulatory / source systems

- **NIPR (National Insurance Producer Registry)** — central system for producer
  licensing/appointment data exchange.
- **PDB (Producer Database)** — NAIC/NIPR database of licensing info across
  states; common upstream source.
- **State DOI (Department of Insurance)** — the state regulator issuing licenses
  and recording appointments/terminations.
- **NAIC** — National Association of Insurance Commissioners; standards body.

## Common abbreviations

| Abbrev | Meaning |
| --- | --- |
| NPN | National Producer Number |
| LOA | Line of Authority |
| A&H | Accident & Health |
| P&C | Property & Casualty |
| JIT | Just-in-time (appointment) |
| IMO/FMO | Independent/Field Marketing Organization |
| BGA | Brokerage General Agency |
| MGA | Managing General Agent |
| DOI | Department of Insurance |
| CE | Continuing Education (license-renewal requirement) |
