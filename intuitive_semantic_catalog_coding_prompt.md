# Agentic_App — intuitive semantic catalog authoring

## 1. Assignment and intended outcome

Act as the lead engineer for Agentic_App. Redesign its semantic catalog authoring experience so a developer and business owner from another company team can publish useful information with the familiarity of conventional BI modeling. Deliver a repository-grounded design and implement the smallest working slice appropriate to the editor's current mode.

The platform serves agents answering questions over Azure SQL, ADLS Gen2, and eventually other information systems. Teams should contribute datasets, fields, relationships, value mappings, and business meaning using familiar source-native tools. SQL views, SQL queries, existing APIs, and document collections have appropriate implementation paths. The authoring experience should make common changes local, readable, and testable.

The current implementation has become difficult to maintain. Screens expose logical bindings, expression strategies such as `first_existing` and `per_row`, predicate IDs, operation IDs, value IDs, redundant context fields, and repeated administrative metadata. This assignment establishes the next authoring direction. Simplify those requirements while retaining the behavior existing consumers rely on and resolving genuine semantic discrepancies explicitly.

Inspect repository instructions, relevant source files, existing tests, and normal application entry points. Identify the actual model, persistence, retrieval, planner, adapter, and UI paths. Reuse them. In design-only mode, produce the design and concrete proposed changes. In implementation mode, make the focused changes and run tests. If an application repository is unavailable, produce an implementation-ready design and synthetic fixtures, with that access limitation stated.

- Every manually maintained attribute must serve a current authoring decision, runtime behavior, or applicable governance requirement.
- Put related information together so one ordinary task can be completed in one editor.
- Use familiar SQL or source-native code for complicated computations. Keep published inputs, outputs, and business meaning explicit.
- Generate technical IDs, revisions, dependency records, indexes, and effective capabilities where the system has sufficient information.
- Treat identity, grain, matching rules, permissions, and answer completeness as executable requirements.
- Introduce shared objects when independent reuse, ownership, or lifecycle gives them a purpose.
- Preserve stable references and use migration tests when consolidating existing definitions.

## 2. The small authoring model

Retain the seven proposed concepts below. Organize their editing around datasets and domains. Stable identities can exist for nested definitions, and internal storage can remain normalized.

| Concept | Ordinary authoring location | Information the human supplies |
|---|---|---|
| Entity / logical dataset | Main dataset page | Name, meaning, source implementation, population, grain, and normal output |
| Attributes / fields | Dataset Fields grid | Business labels and meanings, source mapping or calculation, optional aliases and value mapping |
| Relationships | Dataset Relationships tab | Business role, target, matching rule, expected multiplicity, and intended use |
| Mappings | Field Values panel, shared when useful | Canonical values, source codes, aliases, and applicable scope |
| Grain and identity | Dataset Record definition panel | Meaning of one row, identity fields when known, and any reviewed normalization |
| Business concept | Local description or reusable term | Meaning, aliases, and links to the dataset, field, relationship, or measure that implements it |
| Business domain | Main domain page | Purpose, datasets and terms, owner, and relevant retrieval defaults |

Add **measures** as an optional dataset field category because the supported questions include counts and aggregations. Support ordinary counts, distinct-identity counts, and approved numeric aggregations first. An independently managed metric becomes appropriate when it has shared ownership or implementations across datasets.

Keep default display fields, preferred identifier interpretation, owner inheritance, and examples as dataset settings. A short form can configure these settings. Shared vocabularies and business concepts should be available when needed while local definitions remain convenient.

### Three-layer responsibility

The editor should make the contribution of each layer visible within a coherent dataset workflow.

| Layer | Responsibility |
|---|---|
| Technical | Discover physical objects, schemas, locations, constraints, and observations with provenance |
| Logical | Bind source outputs to typed datasets and fields, and implement relationships, grain, calculations, and supported operations |
| Business | Maintain meanings, labels, synonyms, domain membership, and expected answers |

A field row can show all three responsibilities together. The system can maintain their stable references internally. A business entity may have several logical datasets, such as current licenses, license history, and license-product-line associations. An entity term can be shared when these views need one common definition.

## 3. Datasets and fields

### Logical dataset

Treat a dataset as a coherent queryable population with a declared row meaning. Its implementation can be a table, view, registered read-only SQL query, or a CSV/Parquet collection interpreted through the relevant adapter. A collection of partitions can implement one dataset. A reviewed join or transformation can also implement one dataset.

The ordinary editor should gather the information needed to understand and query the dataset.

1. A business name and description.
2. The source table, view, query, or collection.
3. What one row represents.
4. Identity fields when established, with a check that explains their scope.
5. Ordered default display fields.
6. Domain membership and owner, using inherited values where applicable.

Discover types and physical mappings where possible. Display observed facts and human assumptions distinctly. Missing identity evidence should constrain identity-dependent operations, while ordinary read-only operations can remain available when their result semantics are defined.

For registered queries, use the database's parser/planner and the existing adapter. Execute reviewed SQL through server-controlled bindings and parameters. Expose composable projections and filters only when the registered implementation supports them. Preserve authorization when views or service identities change the execution context.

### Fields

Provide a compact grid with label, description, source column or calculation, type, and default visibility. A direct source field should need little authoring. Advanced settings appear when relevant to identifiers, units, currencies, timezones, sensitivity, or business null semantics.

Calculated fields can be SQL expressions, columns of a reviewed view, or existing source-native implementations. Their executable source should have one authoritative location. The platform can derive dependency metadata to the extent it can establish it reliably, and accept reviewed dependency declarations for complex implementations.

For source types beyond SQL, retain the resource/adapter boundary. Document search and passage retrieval should advertise their actual result shape and evidence coverage. Implement the connectors needed for current workflows, and preserve a place for future capability-specific interfaces.

## 4. Grain, identity, distinctness, and normalization

Explain this section with examples in both the design and the UI. Authors need to understand the consequences before publication.

### Questions the Record definition panel asks

- What does one published row represent?
- Which fields identify that row, and within which source, tenant, jurisdiction, or time scope?
- Is this a current entity, a historical version, an event, an association, or a derived summary?
- Does the source already satisfy that grain? If several source rows represent one published record, which reviewed transformation constructs it?

Store the intended grain and its identity field references on the dataset. Validate the key where a dependent operation relies on uniqueness. Distinguish an entity identity from the key of its historical version. A proposed composite key needs evidence. A label such as “license” alone cannot establish uniqueness.

### Concrete example

Use the following synthetic population to demonstrate how changing the requested result affects grain.

| License ID | Producer | State | License number |
|---|---|---|---|
| L1 | P1 | OH | 001 |
| L2 | P1 | OH | 002 |
| L3 | P1 | PA | 003 |

The license list contains three records. Displaying only Producer and State still displays three records, including two identical-looking P1/OH rows. An explicit request for distinct Producer/State combinations returns two combinations. An explicit producer count returns one producer. Each operation has a different stated result grain.

Changing displayed columns must preserve license identity and multiplicity. The result grid must label a transformation such as “Unique State values” or “One current record per license” according to its actual meaning.

Store the tuple defining a unique-combinations operation separately from grid visibility. For unique Producer/State combinations, hiding State preserves the selected tuple and its two result rows. An explicit change to unique Producer values yields one row. Display the operation's selected fields so users can understand this behavior.

Internal license keys, policy inputs, and pagination tie-breakers retain their internal roles when the result grain changes. Apply DISTINCT or GROUP BY to the declared tuple only. Order and paginate the derived result using its resulting grain or a result-specific handle. Including hidden license IDs in the distinctness tuple would incorrectly keep three combinations in this example.

### A business identity defined by several columns

Suppose a domain expert declares the synthetic combination `(source_system, producer_profile_key, state, license_number)` to identify a business license. Record that exact tuple and its scope. Inspect the source rows to choose a suitable published representation.

| Source condition | Appropriate implementation |
|---|---|
| Exactly one row exists per approved tuple | Expose the resource directly and validate the key. |
| Repeated rows have identical retained field values | Publish a reviewed distinct projection at the agreed grain, and retain lineage to the underlying records. |
| Rows share the identity tuple and differ in status, dates, or other retained fields | Choose the intended population. For a current snapshot, use an approved effective-time or deterministic survivor rule. For history, expose versions with a version-aware key. |
| One license has several LOA or product-line associations | Model the association separately or expose a collection/detail expansion. Qualifying the license uses existence semantics. |

Declaring a key establishes an invariant. A separate reviewed query implements any required normalization. A single `distinct` switch on the dataset cannot specify how conflicting non-key values should be selected.

For counting combinations in Azure SQL, a grouped or distinct-key subquery can express the intended count. The query below assumes complete, valid keys and uses illustrative published names.

```sql
SELECT COUNT_BIG(*) AS business_license_count
FROM (
    SELECT source_system, producer_profile_key, state, license_number
    FROM published_license_rows
    WHERE producer_profile_key = @profile_key
    GROUP BY source_system, producer_profile_key, state, license_number
) AS business_licenses
```

This counts qualifying identities. Returning their status or dates still needs a defined rule for those attributes. Decide whether record filters apply to a current snapshot or represent existence of a qualifying historical version. Filter placement can change the answer, so keep those populations explicit.

Use tuple semantics for composite identity. Naive string concatenation can collide, and nulls, collations, and normalization can affect equality. Specify how incomplete identifiers are handled. A generated row number can support a particular result snapshot, while business identity requires its own definition.

## 5. Relationships and complex joins

Most relationship editing should feel like a BI join dialog. Authors choose source and target fields, add equality pairs, name the relationship, confirm expected multiplicity, and specify its purpose. Show how the join affects unmatched rows and the output grain.

The intended relationship use determines how matching records contribute to the result.

- **Qualification** keeps a base record when a related match exists.
- **Enrichment** adds a single related value while applying the approved row-preservation behavior.
- **Expansion** returns multiple related records and exposes the new result grain.
- **Anti-qualification** keeps records without a qualifying related match.

Different uses can share one matching definition. The runtime chooses a supported implementation for the requested use. An inner join and a left join have distinct preservation requirements, and a downstream filter can alter the resulting population.

The A–B–C pattern below requires the complete conjunction to determine its matching rows.

```sql
FROM TABLE_A AS a
JOIN TABLE_B AS b ON a.col1 = b.col1
JOIN TABLE_C AS c
  ON a.col2 = c.col2
 AND b.col3 = c.col3
```

Both C comparisons refer to the same C row. Model the A–B result as an input to the next join, or publish the complete reviewed query as a logical dataset. Complex matching belongs in readable SQL or an existing source-native implementation. The runtime can index the declared relationships as a graph. Authoring should remain dataset-centered.

A shortest-path search is insufficient to recover the full conjunction. Resolve the published relationship and load its declared dependencies, including prerequisites whose fields are absent from the display. Support role-specific relationships when two paths have different meanings, such as license state and residence state.

Retain current cardinality checks where they affect correctness. For an existence-based relationship, repeated equivalent witnesses can be valid. A uniqueness requirement should be tied to an operation that actually depends on it.

## 6. Canonical values and source mappings

Use a small editable mapping table attached to a field. Reuse an existing shared mapping when several datasets genuinely share its meaning. Each row can contain a canonical label, source value or values, aliases, and applicable scope.

For example, `Terminated -> T` is an illustrative status mapping. Inspect the real source before asserting the literal. Some systems may have several source codes under the same canonical status. Resolve those to a typed set of values and use an appropriate predicate. A business definition that also depends on dates belongs in a reviewed calculation or named predicate implementation.

Scope mappings by the relevant field and source. Add jurisdiction or effective-time scope only when the real mapping depends on it. A scalar one-to-many translation and a state/type-dependent codebook have different computational needs. The licensing RegEd lookup is a relational transformation with multiple matching conditions and can stay in SQL.

Use exact code, label, and curated alias matching first. Semantic retrieval can suggest candidates when needed. Ambiguous values require clarification or an approved context rule. Retrieve high-cardinality identifiers through an authorized lookup. The output must refer to actual source values.

Keep original values available for evidence and troubleshooting. A mapping row must explain whether several codes are interchangeable for filtering, grouping, or both. Case folding, whitespace normalization, and significant leading zeros require field-specific behavior.

## 7. Business concepts and contextual identifier defaults

Use business concepts as reusable meanings connected to actual datasets, fields, relationships, or measures. A locally understood field can begin with its own label and description. Promote a term when sharing it adds value. Examples include License, Producer, Profile key, Active license, and Accident and Health.

The definition of “Active license” may reference an approved SQL rule. A business concept can also be documented while its executable binding is pending. The runtime must surface that state truthfully for questions that depend on it.

### The A2345678 example

For “Fetch me licenses for A2345678,” configure a small dataset setting that says an unqualified identifier in a licensing request refers to the **holder profile key**. This setting references the actual field or registered lookup route selected in the editor. The concept Profile key gives it a label, definition, and aliases. The setting supplies the executable default.

A short form can make the default interpretation reviewable.

| Setting | Example |
|---|---|
| Context | Licenses dataset in Producer Licensing |
| Default meaning of an unqualified identifier | Holder profile key |
| Target | Approved field or lookup route for that identifier |
| Example | Fetch licenses for A2345678 |
| Visible interpretation | Holder profile key equals A2345678 |

The existing SQL uses `PRODUCER_USERID_C`. Confirm that Profile key refers to that identity domain before binding it directly. If a crosswalk is required, make it part of the reviewed lookup. An equal-looking value in two systems alone supplies insufficient identity evidence.

Resolve identifier meaning using deterministic precedence from the most explicit applicable instruction.

1. Honor an explicitly named identifier field, such as “license number A2345678.”
2. Use an unambiguous active conversational reference or user-selected field when appropriate.
3. Apply the approved default for the selected dataset and role.
4. Apply an applicable domain default where it supplies a unique target.
5. Where a unique target remains unresolved, use authorized registered identifier routes to produce candidates and ask a focused question when needed.

A shape or prefix can suggest a candidate identifier type. Exact lookup and the scoped semantic definition establish its use. Preserve significant formatting and use bound parameters. The prefix A supplies no producer-type rule.

If the approved profile-key filter finds zero licenses, return that scoped result. Changing to a license-number search requires a new interpretation that is visible to the user. Repeated license rows for one holder represent one holder identity in a lookup. Several distinct holder identities remain ambiguous.

Expose the applied identifier interpretation as a filter chip or short summary so a user can correct it easily. Keep this setting simple enough to configure with a field picker and one worked example.

## 8. Domains and bounded retrieval

A domain provides business organization, ownership, and an initial retrieval scope. Examples include Producer Licensing and Contracting. It can reference shared datasets and concepts using stable IDs. Reuse shared definitions through membership links.

Use the domain to reduce candidate retrieval, with cross-domain dependencies handled explicitly. A prompt about licenses and contracts may require both domains. Relevant mappings, shared identifier definitions, and relationship prerequisites must remain reachable even when stored outside the initially selected domain.

Use the following sequence to bound candidate retrieval and expand the dependencies needed by the selected interpretation.

1. Determine authorized candidate domains from UI context, the conversation, and business terms.
2. Retrieve a small set of relevant datasets, fields, relationships, and examples within that scope.
3. Load authoritative definitions and the dependencies required by the selected interpretation.
4. Resolve values and identifier defaults.
5. Validate and execute the supported plan.

If routing is ambiguous or incomplete, broaden the permitted candidate search or ask a question. Top-k retrieval finds candidates, and required dependencies are loaded by reference. Authority to access a resource comes from permission enforcement, independently of its domain membership.

Build searchable representations from the published definitions. Retain business descriptions and examples for retrieval while using exact IDs at execution. A large codebook or source schema can be queried on demand. Keep the model context bounded by the particular question and its required dependencies.

## 9. Measures, display defaults, and spreadsheet interactions

Measures are optional fields with explicit meanings. A count can mean rows in the published population, distinct business licenses under an approved key, or producers holding a qualifying license. Record that choice. Numeric aggregates require appropriate fields, units where relevant, and grouping semantics. The runtime should preserve metric meaning when related datasets are involved.

Configure an ordered default field list on each dataset and make subsequent projection changes predictable.

| Request or UI action | Behavior |
|---|---|
| Get matching licenses | Show the authorized default fields at license grain. |
| Show license number and state | Set the explicit projection in that order, subject to actual mappings and permissions. |
| Also include status | Add status to the current projection and preserve the query's other settings. |
| Hide or reorder a column | Change presentation while preserving records and multiplicity. |
| Count matching licenses by state | Select the approved measure and grouping field. Offer a detail drill-down. |
| Add a one-to-many field | Choose a collection, detail view, or explicit row expansion with its resulting grain. |
| Show unique combinations | Apply an explicit distinct projection with a clear result label. |
| Keep one current record per license | Apply the approved identity and version/survivor rule. |

Chat and the grid should edit the same typed query state. Apply full-dataset filters, grouping/distinctness, and ordering at the correct semantic stage before pagination. Retain deterministic tie-breakers where a stable key permits them. An aggregate filter belongs at its appropriate stage, such as HAVING, and a current-record filter must respect version selection.

Specify export scope and visible field order. Keep internal join, policy, or pagination fields out of the visible result unless requested and permitted. “All records” refers to coverage, while “all fields” refers to the selectable output schema.

## 10. Licensing implementation grounded in existing evidence

The first vertical slice can use the sources and fields established in the supplied SQL.

| Source | Known roles and fields |
|---|---|
| `dbo.PRODUCER_LICENSES` | Holder `PRODUCER_USERID_C`, jurisdiction `LIC_JURIS_ID_C`, LOA join key `AGENT_AGENCY_LIC_ID_C` |
| `dbo.PRODUCER_LOA` | Same license join key, assignment code `PROMPT_BUS_CODE` |
| `dbo.RegEd_LOA_Codes_Licenses` | Code `RegEd_LOA_Codes`, jurisdiction `[State]`, `Producer_Type`, `ProductLine` |
| `dbo.PRODUCER_KEY` | Holder `PRODUCER_USERID_C`, indicator `PRODUCER_IND_ID` |

The example request is “Get licenses for producer A9906586 and Ohio State and accident and health product line.” Its source values in the supplied SQL are A9906586, OH, and AH. There is no active-status restriction in that request.

The latest supplied definition records steward confirmation that every holder has exactly one qualifying 1001 or 1002 key. Preserve the ordered implementation in which 1001 yields Individual, otherwise 1002 yields Agency, otherwise null. Keep the exact-one-key assertion as a checked assumption. If both codes appear, ordered evaluation chooses Individual and the assertion reports a violation. Review the data scope of that assertion. The earlier SQL used the source literal Agent in the RegEd mapping, so confirm and implement the Individual-to-Agent crosswalk.

Publish Licenses and, if useful, License product lines as source-native datasets. The relationship License covers product line must find an assignment on the same license and one mapping row satisfying code, jurisdiction, applicable producer type, and line together. Existence qualification preserves each base license record once. A detail action can show multiple matching assignments.

Use this case to demonstrate the simpler dataset, field, relationship, mapping, grain, concept, and domain editors. Maintain the producer classifier in readable SQL or the existing reviewed implementation. Generate internal references around it.

Multiple SQL variants can share a semantic implementation contract after verifying population, projection, identity, multiplicity, temporal behavior, and access context. Compare actual plans and representative workload measurements when optimizing. A query with a filter written earlier or a different join spelling has no established performance advantage from syntax alone.

## 11. Publication, extension, and migration

Use the existing platform mechanisms to support this authoring model reliably.

- Stable IDs for published definitions, including nested fields and relationships. Label edits preserve their identity.
- A coherent published snapshot or the repository's existing release mechanism, with automatic revision history.
- Source and authorization bindings controlled by the server, plus typed parameter handling and bounded execution.
- Validation and examples for the behaviors each dataset advertises.
- Derived retrieval indexes and dependency information, with authoritative definitions loaded before execution.
- Adapter interfaces that can expose additional source-specific capabilities later.

Inherit owner and domain where appropriate. Derive implementation kind and supported operations from the registered implementation, logical type, and policy. Show manual overrides only for genuine exceptions. Store narrow reviewed business defaults through ordinary forms.

Audit existing custom metadata and expression objects by their consumers. Map current identifiers to the new authoring view, retain compatible references, and compare outputs before removing redundant implementation paths. Keep one authoritative source for each executable rule. Migrate the licensing slice first, including its retrieval and UI entry points.

The simplification should reduce what people author and troubleshoot. A large internal graph can remain an implementation detail if it already serves a real function. A new required publication engine, ontology language, or infrastructure service must be justified by a current use case and an explicit maintenance cost.

## 12. Required design, implementation, and verification

Ground the deliverables in the repository's structure and development conventions.

1. A concise architecture/design note explaining each object, its owner, authoring location, and generated metadata. Include an onboarding walkthrough and a diagram showing the relationship between authored definitions, retrieval, and execution.
2. A concrete schema and UI change plan grounded in current files. Show what is nested, shared, inherited, or computed. Use actual project types and clear field names.
3. A working first slice through the normal application entry point when implementation mode permits. Demonstrate both the A9906586/OH/AH query and the A2345678 default-identifier query using labeled fixtures or approved data access.
4. A grain/distinct walkthrough showing the different three-license, two-producer/state, and one-producer results. Explain composite identity and normalization in language an author can understand.
5. Tests for the specific regressions below, plus the existing relevant suite. Report observed outcomes and any unverified behavior.
6. A short migration plan with affected consumers and a demonstration that a new team can add a field, change a mapping, and trace a wrong result from one dataset editor.

### Acceptance scenarios

| Scenario | Required result |
|---|---|
| Two license rows have the same displayed Producer and State | Both rows remain until an explicit unique-combinations operation is requested. That operation uses its own declared tuple independently of hidden internal keys and later visibility changes. |
| Composite business key has repeated rows with conflicting status | The key declaration reports or constrains the conflict. A published normalized result requires the approved version/survivor/aggregation rule. |
| Null or source-colliding identity fields | The supported identity operations respect the defined scope and incomplete-key behavior. |
| Same producer has two Ohio licenses, only one with AH, plus a Pennsylvania AH license | The Ohio AH list includes exactly the qualifying Ohio license. |
| Multiple LOA assignments qualify the same license | License list and license count preserve one base record. Evidence detail can show all matching assignments. |
| Correct code, state, type, and line occur across separate mapping rows | None qualifies until one row satisfies the full conjunction. |
| A/B/C relationship with C pairs (X,Y), (X,Z), and (W,Y) | Only (X,Y) matches the complete predicate. Removing either condition fails the test. |
| Missing, mixed, duplicate, and valid producer-key records | Ordered classification and the exact-one-key assertion have independently specified outcomes. |
| Bare A2345678 in a licensing request | The published default targets Holder profile key and reports that interpretation. Explicit “license number” overrides it. |
| Same text can denote a profile key and license number | The scoped default or explicit request decides when available. An unresolved conflict leads to focused clarification. Zero matches retain the chosen scope. |
| User requests Terminated status | Resolve the approved codes for that field/source. An unrelated dataset's code mapping has no effect. |
| Request spans licenses and contracts in different domains | Retrieve the relevant domains and authorized relationship dependencies, with a bounded question-specific context. |
| Explicit, additive, and hidden column changes | Honor projection intent, preserve grain, and retain required internal predicates. |
| Dataset-wide filters/distinctness with several pages | Counts and exports match the same full authorized operation, independent of page size. |

Explain remaining business decisions, fixture assumptions, source limitations, and performance measurements honestly. Continue useful local work while an unresolved domain decision is represented explicitly. Avoid changing the business meaning merely to make a test or query succeed.

### Existing-product references

Consult these for familiar modeling idioms and source-specific behavior. Explain the specific pattern adopted in the design.

- Looker puts SQL expressions on fields and supports references to existing fields. [Field SQL reference](https://docs.cloud.google.com/looker/docs/reference/param-field-sql)
- Cube groups source, dimensions, measures, and joins within a dataset-like model. [Cube model reference](https://docs.cube.dev/reference/data-modeling/cube)
- Azure SQL distinguishes row counts from counts of distinct non-null expression values. Use the dialect's supported syntax for composite-key counts. [COUNT reference](https://learn.microsoft.com/en-us/sql/t-sql/functions/count-transact-sql?view=sql-server-ver17)

The acceptance criterion is an engineer and business owner completing the first useful dataset with a source selection, a field grid, a record definition, understandable relationships, value mappings, and example results. Every additional authoring requirement should earn its place through a concrete supported behavior.
