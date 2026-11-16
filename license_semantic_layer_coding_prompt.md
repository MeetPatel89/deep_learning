# Agentic_App coding assignment — license lookup through the semantic catalog

Work in the existing Agentic_App repository as a senior data and application engineer. Design and implement a narrow, testable Azure SQL slice of our semantic platform around this user request:

> Get licenses for producer A9906586 and Ohio State and accident and health product line.

A licensing analyst should receive a list of matching license records and be able to inspect why a license qualified. A business steward should be able to review the meaning in plain language. An engineer should be able to change the SQL implementation without changing that meaning or the agent-facing interface.

Read the repository instructions and trace the current catalog, planner, adapter, permissions, result grid, and tests. Identify existing extension points by actual path. Reuse them. If the editor is in design-only mode, deliver the design, proposed change list, and executable test specification. In implementation mode, carry the slice through the existing public entry point and run the relevant tests. If there is no application repository, provide the implementation-ready design and fixtures. Treat this as a focused extension of the three-layer architecture; a new graph database, agent framework, or general SQL optimizer is unnecessary for this slice.

## 1. Start from the four real query variants

The source facts below are sufficient to work without the original screenshots. All four queries filter for producer user ID `A9906586`, license jurisdiction `OH`, and RegEd product line `AH`. They have no active-status or termination-date filter. Use typed parameters and approved value mappings in the implementation. Treat “Accident and Health” as one candidate label.

| Role | Source and observed SQL references |
|---|---|
| License, `l` | `dbo.PRODUCER_LICENSES`. Holder is `PRODUCER_USERID_C`, license jurisdiction is `LIC_JURIS_ID_C`, and the LOA join uses `AGENT_AGENCY_LIC_ID_C`. |
| Authority assignment, `loa` | `dbo.PRODUCER_LOA`. Join to the license on `AGENT_AGENCY_LIC_ID_C`. Its authority code is `PROMPT_BUS_CODE`. |
| Authority-to-product mapping, `m` | `dbo.RegEd_LOA_Codes_Licenses`. Match `m.RegEd_LOA_Codes` to the assignment code, `m.[State]` to the license jurisdiction, and `m.Producer_Type` to the applicable producer type. Filter `m.ProductLine` to the requested line. |
| Producer classification evidence, `k` | `dbo.PRODUCER_KEY`. Match `k.PRODUCER_USERID_C` to the license holder. The variants interpret `k.PRODUCER_IND_ID` differently. |

Explain these differences in the design and test them before promoting any SQL to a default binding:

| Variant | Observed behavior |
|---|---|
| 1 | Joins license to LOA, then left joins the mapping by code and state. In `WHERE`, the mapping type must equal `CASE WHEN EXISTS` any key row for this producer with indicator `'1001'` `THEN 'Agent' ELSE 'Agency' END`. Uses `SELECT *`. |
| 2 | Uses the same existence-based classifier, with the producer-type comparison in the mapping's `ON` clause. Both variants keep `m.ProductLine IN ('AH')` in `WHERE`. |
| 3 | Inner joins the producer-key rows and classifies **each row** as Agent for `'1001'` or Agency for `'1002'`, with an implicit null otherwise. Uses `SELECT *`, including the joined key fields. |
| 4 | Applies the producer and jurisdiction filters inside a derived license table, uses variant 3's row classifier, and selects the license, LOA, and mapping columns. It omits the producer-key columns. |

For this filtered query, unmatched mapping rows cannot pass the product-line predicate. Variants 1 and 2 therefore have the same matching rows. Moving a predicate between `ON` and `WHERE` needs fresh review if a later feature must retain unmapped licenses. Variants 3 and 4 have comparable joined rows at a common projection. Narrowing the projection preserves repeated matches. The two classifier families can produce different license eligibility as well as different row counts.

## 2. Write the Business and Logical layer constitution

Produce a repository design note that an engineer and a licensing steward can review together. For each object give its purpose, an example, owner, stable identifier, binding, and acceptance condition. Use one worked request throughout. Include a small diagram showing which fields from which roles participate in the mapping, plus a trace from business intent to final result.

**Business layer.** Define License record, License holder, License jurisdiction, Product line, the relationship **License covers product line**, the scoped policy **Producer type for LOA mapping**, and the **License list** display view. A proposed definition for steward review is: “A license qualifies when at least one authority recorded for that license maps to the requested product line in the license's jurisdiction for the applicable producer type.” Include a counterexample involving two different Ohio licenses held by the same producer. Define whether the population includes historical versions and what one list item represents. The SQL supports a license-level interpretation via LOA. A producer's product contract is a different business relationship.

**Logical layer.** Bind the above concepts to executable resources, typed properties, value domains, predicates, and a versioned relationship. Record the base grain, scoped record identity, role of each input, expected multiplicities, required comparisons, treatment of missing matches, and evidence for uniqueness. The shared LOA join column is observed; its global uniqueness is unverified. If a larger key or as-of scope is necessary, specify the evidence and how it is enforced. Keep the classification policy a named dependency; a reviewed source view or existing query builder is a valid first implementation.

**Technical layer.** Reference the four discovered tables and columns. Distinguish observed joins, declared source constraints, tested data properties, and approved business rules. Assign business terminology in the Business layer and executable behavior in the Logical layer. Show how a business definition can be authored before its production binding is approved.

## 3. Represent the complete three-way relationship

Support this general pattern using the repository's representations:

```sql
FROM TABLE_A AS a
JOIN TABLE_B AS b ON a.col1 = b.col1
JOIN TABLE_C AS c ON a.col2 = c.col2 AND b.col3 = c.col3
```

Model the A–B result as an input to the next join, or define a named relationship with bound roles and a required conjunction. In either representation both comparisons target **the same C row**. Give the object a stable ID and record its dependencies. A graph projection can help discover the object; selecting an ordinary shortest path could omit one comparison. It is fine for the database optimizer to reorder equivalent inner joins.

Apply this pattern to the license, its own LOA assignment, and one RegEd mapping row. The producer classification adds a fourth dependency. That mapping row must satisfy **code, state, producer type, and product line together**. A field hidden in the result can still be required to qualify a license. Preserve the dependency even if the user selects only a license field.

## 4. Specify the agent-to-runtime boundary and reference SQL

Show a typed plan like the following, adapted to actual repository types:

```yaml
resource: license_record
projection: {mode: default, view: license_list}
filters:
  - {field: license.holder, equals: resolved_producer}
  - {field: license.jurisdiction, equals: resolved_ohio}
  - exists: {relationship: license.covers_product_line, value: resolved_accident_and_health}
```

Walk through value resolution, authorization, loading the published binding and its dependencies, validation of grain and projection, parameter binding, execution, and answer formatting. The agent proposes semantic IDs and intent. Deterministic code owns classification, required predicates, field access, and SQL compilation. Resolve the producer identifier exactly in its scoped domain. Index the relationship once for retrieval; store Ohio, the producer ID, and the product line as request values. Trace the release, binding, classifier revision, and projection used.

Use the following reference shape **only under the existence-based classification policy of variants 1–2**. This is valid as a fixture assumption until an existing approved rule or the business owner settles the policy. Types and the production display list come from the repository or source metadata.

```sql
SELECT l.AGENT_AGENCY_LIC_ID_C, l.PRODUCER_USERID_C, l.LIC_JURIS_ID_C
FROM dbo.PRODUCER_LICENSES AS l
WHERE l.PRODUCER_USERID_C = @producer_user_id
  AND l.LIC_JURIS_ID_C = @jurisdiction
  AND EXISTS (
      SELECT 1
      FROM dbo.PRODUCER_LOA AS loa
      JOIN dbo.RegEd_LOA_Codes_Licenses AS m
        ON m.RegEd_LOA_Codes = loa.PROMPT_BUS_CODE
       AND m.[State] = l.LIC_JURIS_ID_C
      WHERE loa.AGENT_AGENCY_LIC_ID_C = l.AGENT_AGENCY_LIC_ID_C
        AND m.ProductLine = @product_line
        AND m.Producer_Type = CASE
            WHEN EXISTS (
                SELECT 1
                FROM dbo.PRODUCER_KEY AS k
                WHERE k.PRODUCER_USERID_C = l.PRODUCER_USERID_C
                  AND k.PRODUCER_IND_ID = '1001'
            ) THEN 'Agent'
            ELSE 'Agency'
        END
  )
```

The outer `EXISTS` yields each qualifying base row once. If the base contains multiple versions of a license, its approved population definition must determine which versions are eligible. Expose a separate evidence drill-down when the user asks which LOA or mapping qualified a license. Review the missing-key fallback in the adapter's actual access context: a restricted view of key rows can also affect an existence-based classifier.

## 5. Choose visible columns by purpose and grain

Define an ordered display profile per result type using semantic field IDs. Justify each default in terms of what a licensing analyst needs to identify or review a row. Separate output fields from fields used internally for matching, security, filtering, sorting, and stable pagination. Validate access to each requested operation. Bind only verified source fields; examples such as license number or status require real mappings.

| User intent | Required behavior |
|---|---|
| “Get licenses …” or “get all matching records” | Use the authorized default license-list fields. Show applied scope in the UI. “All records” changes coverage, not projection. |
| “Show license number and state” | Return exactly those authorized fields in order when bound. Report an unavailable field explicitly. |
| “Also include status” / “hide status” | Amend the current projection. Retain filters and grain. Hiding or reordering fields preserves row count. |
| “Count matching license records” / “by state” | Return the measure, or state plus measure. Reuse the same qualification contract. Allow drill-down to the list. |
| “Show which LOAs qualified each license” | Offer evidence per license or an explicitly labeled expanded result. A one-to-many field can change grain. |
| “Show unique states” / “one row per license” | The first operates on projected state values. The second requires a certified license identity and, when relevant, a version-selection rule. |

Use one query state for chat and grid edits. Apply dataset-wide filters, sorting, and explicit distinct/grouping to the full authorized result with the correct semantics before pagination. Export the stated scope and visible field order. Avoid silent `DISTINCT` when a narrower projection makes genuinely different records look identical. If the repo has no grid, demonstrate these rules at the planner/API boundary and document the UI contract.

## 6. Decide when several SQL implementations can share a binding

Give a concrete decision table for the four variants, the reference list query, and any existing repository implementation. Address these questions in order:

1. **Business equivalence.** Do they implement the same population, classification, null and missing-data behavior, jurisdiction, and time scope?
2. **Result equivalence.** At an identical authorized projection, do they return the same identities, multiplicities, and field values? Compare raw joined evidence and license lists at their respective grains. Equal totals, a screenshot, or a set-only comparison can hide differences.
3. **Runtime eligibility.** Are required fields, source coverage, permissions, and freshness available? Record the conditions for using a precomputed mapping or alternate source.
4. **Maintenance and retrieval.** Prefer one named relationship and a readable parameterized default. Store reviewed question–SQL examples with the logical model. Ranking four SQL strings independently is unnecessary. Tie an alternate to the same semantic contract only after its preconditions and tests are documented.
5. **Measured performance.** Compare eligible candidates with the same parameters, projection, and workload. Include zero-match, sparse, and high-volume cases. Use actual Azure SQL plans, logical reads, CPU, duration, and Query Store when available. Distinguish SQL spelling from physical execution strategy. If measurements are unavailable, give a reproducible benchmark and label it unmeasured. Recommend an alternate only for an identified operational benefit.

Discuss a relevant precedent for complete join dependencies (Looker), logical-name verified queries (Snowflake), and plan comparison (Azure SQL Query Store). Explain what each contributes to this design. These are precedents, not required products. Primary references:

- [Looker join dependencies](https://docs.cloud.google.com/looker/docs/reference/param-explore-join-required-joins)
- [Snowflake verified queries](https://docs.snowflake.com/en/user-guide/views-semantic/verified-query-repository)
- [Azure SQL Query Store usage scenarios](https://learn.microsoft.com/en-us/sql/relational-databases/performance/query-store-usage-scenarios?view=azuresqldb-current)

## 7. Acceptance tests and handoff

Use synthetic data with independently specified expected results. Run the fixture through the existing request path where available. Use labels such as L1 only in fixtures.

| Fixture | Expected assertion |
|---|---|
| Producer P has two Ohio licenses L1 and L2. Only L1 has a qualifying LOA. P also has a Pennsylvania license with a qualifying LOA. Another producer has a matching Ohio license. | For P, Ohio, and AH, the list is exactly L1. The request introduces no status test. Repeat with different values to catch hard-coded sample literals. |
| Two assignments or two mapping rows qualify L1. | The license list and record count each include L1 once. The evidence view retains both qualifying matches. |
| Correct code, Ohio, producer type, and AH each appear among decoy mapping rows; no single row satisfies all four. Include an assignment with no mapping. | No license qualifies through these rows. A separately requested unmapped-license report can preserve the unmatched base row. |
| General A–B–C fixture: A yields X, B yields Y for the same join key, and C has (X,Y), (X,Z), (W,Y). | Only (X,Y) qualifies. Removing either comparison must break the test. Duplicating (X,Y) distinguishes raw detail from qualification. |
| One assignment and one AH mapping for each type. Key cases: one 1001, one 1002, none, one unknown or null, both codes, and repeated 1001. | Ordered as listed, the existence-based classifier matches Agent, Agency, Agency, Agency, Agent, Agent. The row-based variant yields Agent, Agency, no match, no match, both types, and two Agent evidence matches. Exercise a mapping available only for Agency as well: mixed codes then change license eligibility, not just detail count. |
| Two qualifying licenses share every displayed value. Switch from the default list to an explicit narrower list, add/remove a field, then request unique visible combinations. | Both records remain through ordinary projection edits. Only the explicit unique-combinations operation collapses them. Filters, counts, exports, and pagination agree on scope. Hidden dependencies remain effective. |

Finish with the design note, the smallest working code/configuration change appropriate to the current mode, test commands and observed outcomes, and any unverified assumptions. Show the worked prompt, resolved semantic plan, generated SQL with separate parameter bindings, visible result schema, and fixture result. Identify the business owner decision on missing/mixed producer types and the evidence required to certify license identity. Where those decisions remain open, keep making progress with a labeled fixture policy and mark the production binding as pending.
