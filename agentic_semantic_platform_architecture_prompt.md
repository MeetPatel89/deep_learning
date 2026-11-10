# Master Prompt: Extensible Semantic Platform for Agentic Data Applications

**Purpose:** Use this prompt to develop an implementation-ready architecture, semantic model, and phased delivery plan. The platform should begin with a small, useful implementation while preserving the contracts needed to support additional data systems, semantic capabilities, and agent workflows over time.

**Prompt version:** 1.0  
**Default mode:** Architecture and design, not automatic implementation or deployment.  
**Primary deliverables:** A detailed design specification, architecture decision records, representative contracts and interfaces, a phased implementation plan, and a test strategy.

---

## 1. Your role and the assignment

Act as a principal architect with expertise in semantic modeling, data integration, query planning, agentic applications, information retrieval, governance, and enterprise software evolution.

Design a **semantic platform over heterogeneous information systems that agents can use to answer business questions**. Do not frame the application narrowly as natural-language-to-SQL. SQL generation is one possible execution capability, not the organizing principle of the platform.

The central requirement is:

> We will not implement every capability in V1 or V2. Nevertheless, the foundational model, interfaces, ownership boundaries, and versioning strategy should allow the application to evolve for years without routinely replacing its semantic foundation.

Do not promise that no redesign will ever be necessary. Instead, identify the stable contracts that reduce migration cost, isolate change, and preserve business meaning when implementations change.

Balance two competing risks:

- **Under-design:** Hard-coding SQL assumptions, physical schema details, agent-framework dependencies, or UI shortcuts into the permanent semantic model.
- **Over-design:** Building a universal ontology engine, distributed query optimizer, graph database platform, or elaborate plugin system before concrete use cases justify them.

For every important recommendation, explain **what it is, why it exists, how it works, what belongs in V1, and what can be deferred**. Prefer concrete examples and tradeoffs over lists of architectural buzzwords.

### Working rules

1. Use the project requirements in this prompt as the starting point. Where repositories, existing specifications, or uploaded files are available, inspect them before claiming compatibility. Do not claim to have inspected unavailable material.
2. Distinguish **provided facts**, **proposed decisions**, **illustrative examples**, and **unresolved assumptions** throughout the design.
3. Ask focused questions when an answer would materially change the architecture or business semantics. Where answers are unavailable, continue with an explicit provisional assumption or a safe unsupported state rather than inventing business facts.
4. Separate questions that block implementation from those that only block production certification. Do not let unanswered details prevent useful architectural progress.
5. Do not treat an LLM instruction as a security boundary, correctness guarantee, or substitute for an enforced contract.
6. Do not introduce an external service, framework, database, or deployment topology without explaining why a simpler alternative is insufficient.
7. Do not implement, deploy, provision infrastructure, or modify source systems unless separately authorized.

## 2. Project context and intended evolution

The application is still developing. Initial systems include Azure SQL and Azure Data Lake Storage Gen2. Its eventual scope may include relational databases, lake tables, files, NoSQL collections, VM file systems, document repositories, APIs, and other information resources.

Representative source objects include tables, views, stored procedures, Delta tables, CSV, Parquet, JSON, XML, text, and business documents. The platform should distinguish these resources without pretending that all of them support the same operations.

Business users should be able to maintain business concepts in an approachable interface. Technical teams should connect those concepts to executable logical models and physical sources. Agents should resolve user questions through those approved semantic definitions rather than improvising schema meaning, joins, identifiers, or business rules.

The platform should ultimately support questions that combine structured computation with document evidence, while allowing V1 to implement only a well-defined subset.

## 3. Architectural constitution: first principles

Use the following principles as design constraints. Challenge a constraint only with a clear reason, explicit tradeoff, and replacement proposal.

| Principle | Architectural implication |
|---|---|
| Business meaning must survive physical migrations. | Business concepts have stable identities independent of databases, tables, files, labels, and connector names. |
| Meaning, implementation, and observation are different. | Keep Business, Logical, and Technical semantic responsibilities separate. |
| Correctness requires more than relevant retrieval. | Resolve candidates, validate typed plans, enforce contracts, and test underlying data assumptions. |
| The model may propose; the runtime authorizes. | LLM output is untrusted input to governed resolution and execution services. |
| Metadata is not automatically verified truth. | Distinguish discovered, declared, inferred, tested, and approved assertions. |
| A graph-shaped domain does not mandate graph storage. | Use typed relationships and graph traversal where useful; choose persistence independently. |
| A relationship is not automatically an executable join. | Business links, join contracts, lineage, bindings, and evidence links have different edge types and validation rules. |
| Retrieval representations are replaceable. | Search indexes, embeddings, graph projections, and context packages are derived from authoritative records. |
| Definitions and implementations evolve together. | Publish compatible, versioned semantic releases and retain dependency information. |
| No supported answer is better than a confidently incorrect answer. | Ambiguity, unsupported operations, stale contracts, and missing coverage have explicit outcomes. |
| Security applies to metadata as well as data. | Enforce visibility before restricted metadata, passages, or results enter model context. |
| Extensibility comes primarily from boundaries. | Prefer a small number of stable contracts over speculative frameworks and infrastructure. |

Explain the limits of these principles. For example, deterministic validation establishes conformance to approved contracts; it cannot prove that an incorrect business definition or stale source assumption is true.

## 4. Preserve three semantic layers and separate cross-cutting services

Use this as the conceptual starting point:

```text
USER QUESTION
     |
     v
AGENT / PLANNER
     |
     v
BUSINESS ONTOLOGY
  What does the business mean?
     |
     | governed business-to-logical bindings
     v
LOGICAL SEMANTIC MODEL
  What operations are meaningful and computationally safe?
     |
     | governed logical-to-technical bindings
     v
TECHNICAL CATALOG
  What resources physically exist, and what is known about them?
     |
     v
SOURCE SYSTEMS
  SQL | lake | NoSQL | files | documents | APIs

Cross-cutting services:
  Governance and security
  Retrieval and semantic resolution
  Planning, validation, compilation, and execution
  Evidence, lineage, evaluation, and observability
```

This diagram shows semantic dependency, not a requirement for every request to visit three separate services. The implementation may be a modular monolith with shared storage and libraries.

Do not turn governance, retrieval, or orchestration into additional semantic layers. They operate across the three layers.

Define both the **authoring/publication lifecycle** and the **question-answering lifecycle**. Explain how the published model is consumed at runtime without exposing partially edited definitions.

## 5. Technical layer: observed and declared resource metadata

### Responsibility

The Technical Catalog records source resources and the evidence available about their physical structure and access characteristics.

Represent, where relevant:

- Sources, databases, schemas, tables, views, columns, collections, files, document collections, endpoints, procedures, and functions.
- Data types, nullability, keys, constraints, indexes, partitions, file formats, schema versions, and source-native identifiers.
- Discovery method, discovery time, scan scope, connector identity/version, and technical metadata provenance.
- Resource permissions, sensitivity classifications, supported access methods, and operational limits.
- Profiling and quality observations, including their time, sampling scope, and limitations.

A declared SQL foreign key, a tested uniqueness assertion, an inferred NoSQL property, and a manually supplied mapping are not equivalent evidence. Model their distinctions. A constraint may also be disabled, untrusted, scoped, or not enforced by its underlying system.

### Boundaries

Do not promote a source column name into a business definition automatically. Do not infer that matching column names imply compatible identity domains or safe joins. Do not equate a sampled schema with an exhaustive schema.

Manual configurations should be supported when discovery is incomplete, but their provenance must remain distinct from source-observed metadata. A new scan must not silently overwrite a business-approved logical rule.

Distinguish cataloging a collection from materializing every record as a metadata node. The catalog should not accidentally become a copy of all operational data.

For VM file systems and file repositories, specify connector-scoped roots, identities, and access policies. Do not grant an agent unrestricted filesystem access as a substitute for a source adapter.

### Required design output

Provide a minimal resource model and explain how it represents SQL tables, a lake dataset, a semi-structured collection, and a document collection without forcing them into identical schemas or capabilities.

## 6. Logical layer: computable semantic contracts

### Responsibility

The Logical Semantic Model defines how information can be interpreted, related, filtered, computed, retrieved, and combined independently of any one physical representation.

At minimum, discuss:

| Construct | What it defines |
|---|---|
| Logical entity or record resource | A modeled population with explicit identity and grain. |
| Logical attribute | A typed property, its null semantics, and its implementation. |
| Relationship contract | Permitted traversal or combination, with direction, cardinality, predicates, and constraints. |
| Rule or predicate | A reusable executable interpretation of a business condition. |
| Metric or measure | What is counted or computed, over which population and grain. |
| Value domain | Allowed values, canonical identities, labels, aliases, and source mappings. |
| Document/evidence collection | What can be searched, retrieved, cited, and linked to business entities. |
| Resource capability | Which operations a particular logical binding can actually perform. |
| Transformation or derived resource | A controlled derivation with input dependencies and output contracts. |

Not every construct requires a separate service or table in V1. Explain the smallest implementation that preserves the distinctions.

### Foundational semantics

Make the following explicit wherever applicable:

**Grain and identity.** State what one record represents. Distinguish a database row, an entity instance, an event, a historical version, and a derived aggregate. Keys must include relevant scope such as source, tenant, state, or time when needed.

**Population.** Define which records belong to a resource. A metric over all license records is different from a metric over distinct real-world licenses or current license versions.

**Time.** Separate event time, effective time, ingestion time, and query evaluation time. Specify timezone and as-of behavior where relevant. Do not require a complete bitemporal engine in V1 unless the use case demands it.

**Types and units.** Represent the supported subset of dates, timestamps, decimals, identifiers, units, and currencies. Define unsupported conversions rather than silently coercing values.

**Nulls and missingness.** A missing field, an explicit null, an unknown business state, and an unmatched relationship are not necessarily the same thing.

**Aggregation.** Record measure additivity, permitted dimensions, distinctness rules, and the grain at which computation is valid.

**Evidence and trust.** Distinguish contractual expectations from the evidence that those expectations still hold in the source.

The Logical layer should not become a dumping ground for business descriptions or arbitrary executable strings. Identify which definitions should use typed expressions, which may reference reviewed source-native implementations, and how both are governed.

## 7. Business layer: an independent business ontology

### Responsibility

The Business layer represents how the organization thinks, not how its databases happen to be organized.

Business users should be able to maintain labels, descriptions, aliases, definitions, ownership, and business relationships through a simple interface. The underlying model must not be permanently limited to two UI categories named “tables” and “columns.”

Distinguish these concepts:

| Business construct | Example |
|---|---|
| Entity type | Producer, License, Appointment, Contract |
| Property | License State, Contract Product Category |
| Relationship | Producer holds License |
| Rule or business condition | Active License |
| Metric | Active License Record Count |
| Controlled value | Pet Product Category |
| Subject area or domain | Producer Compliance |
| Data product | A governed Producer Compliance information product |
| Event, potentially later | License Terminated |
| Policy, potentially later | Restrictions on use of particular information |

A subject area is an organizational grouping, not an entity instance. A data product includes a governed delivery or consumption contract; it is not merely another name for a glossary folder. An entity type is distinct from the individual records that instantiate it.

### Meaning versus executable implementation

Avoid competing sources of truth:

- The Business layer owns the definition and intent of “Active License.”
- The Logical layer implements that definition as an approved executable rule for a particular population and context.
- A governed binding establishes that the implementation realizes the business definition.

The same separation applies to metrics. A business metric describes what the organization intends to measure; its logical implementation specifies the calculation, grain, filters, and compatible dimensions.

Do not assume that a single enterprise-wide definition of “active” applies to licenses, appointments, contracts, customers, and every jurisdiction. Support scoped definitions and explicit ambiguity.

A concept may exist before it has an executable implementation. Represent that state as **defined but unbound**, not as a fake implementation or a reason for the agent to guess.

### V1 user experience

Propose a simple business-facing interface, potentially starting with entity and property cards, while preserving richer typed semantics underneath. Show how rules, metrics, and value concepts can become visible when needed without reclassifying all existing objects.

## 8. Bindings are governed first-class records

Do not model all mappings as a single `logical_entity_id` field on a business concept.

Support explicit bindings between:

1. Business concepts and logical resources, properties, metrics, or rules.
2. Logical constructs and technical resources or reviewed implementations.

A binding should be able to express:

- Stable identity, revision, endpoints, and binding type.
- Context: domain, tenant where relevant, jurisdiction, population, use case, or effective period.
- Property mappings, transformations, and referenced implementations.
- Ownership, approval state, provenance, dependencies, and validation evidence.
- Source authority or precedence when multiple implementations are available.
- Preconditions and supported capabilities.

Multiple bindings do not automatically mean union the sources, pick the newest source, or let the LLM choose arbitrarily. Define deterministic selection rules and explicit ambiguity outcomes.

Explain how a business concept such as `Contract` can survive migration from one SQL table to multiple sources, and how different logical implementations can coexist without claiming they are interchangeable.

Also distinguish a **schema-level binding** from **record-level entity resolution**. Binding a Producer concept to two systems does not establish that producer record `123` in each system identifies the same real-world producer.

## 9. Graph modeling without premature graph infrastructure

The semantic model is naturally graph-shaped. Its nodes and edges may include:

```text
Business Entity --has_property--> Business Property
Business Entity --business_relationship--> Business Entity
Business Concept --implemented_by--> Logical Construct
Logical Resource --source_binding--> Technical Resource
Logical Resource --relationship_contract--> Logical Resource
Derived Resource --depends_on--> Input Resource
Evidence Passage --mentions_or_supports--> Entity or Claim
Subject Area --organizes--> Business Concept
```

However, these edges have different meanings. A lineage edge cannot be used as a join edge. A document mention is not proof of a business relationship. Traversing a business relationship does not guarantee an executable implementation exists.

### Required recommendations

Evaluate a relational metadata store with typed tables, validated extension payloads, and indexed relationship records as the default starting point. Explain when an in-memory adjacency graph or a derived graph projection is sufficient.

Do not reduce the entire platform to an unconstrained `nodes(id, json)` and `edges(id, json)` model. A common identity registry can coexist with strongly typed records and constraints.

Explain the distinction among:

- The **semantic metadata graph** of types, contracts, bindings, and dependencies.
- An optional **instance graph** of actual producers, licenses, documents, and other records.
- The **workflow graph** used to orchestrate agent/runtime steps.

None of these automatically requires the others or the same storage technology.

A dedicated graph database should have an evidence-based adoption trigger, such as traversal workload, latency, graph size, operational requirements, or features that the current store cannot reasonably provide. Explain the migration boundary rather than recommending a graph database merely because the word “relationship” appears.

## 10. Relationship contracts must govern behavior, not just connectivity

Design a relationship contract that can express the relevant subset of:

- Source and target logical types, roles, direction, and identity domains.
- Join or matching keys, including composite, tenant, and temporal keys.
- Expected cardinality and the evidence supporting it.
- Allowed purposes: enrichment, qualification, anti-qualification, traversal, or aggregation support.
- Row-preservation requirements and permitted physical join strategies.
- Required predicates, including their scope and placement semantics.
- Optionality, missing-match behavior, and null handling.
- Fan-out risk, permitted aggregation behavior, and disallowed operations.
- Temporal matching rules and uniqueness expectations.
- Security, execution-location, and capability constraints.
- Dependencies, version, certification status, and test coverage.

### Critical distinctions

**Enrichment** adds related information while preserving an explicitly defined population. A contract may require a left outer join, with right-side eligibility predicates in the join condition.

**Qualification** retains base records only when a related condition holds. A semi-join or `EXISTS` is often the appropriate implementation.

**Anti-qualification** retains base records only when no qualifying related record exists. Its null semantics must be intentional; do not casually substitute `NOT IN` for a correctly scoped anti-join.

**Aggregation** computes at an explicit grain. A permitted relationship does not automatically make arbitrary aggregation over its joined rows safe.

Do not impose “always use LEFT JOIN” as a universal policy. Enforce left preservation where the contract requires it; use qualification semantics when the user is intentionally restricting the population.

Conversely, checking that the SQL contains `LEFT JOIN` is insufficient. A null-rejecting downstream filter or subsequent inner join can remove preserved records. Multiple matching target rows can also multiply records even with a left join.

### Illustrative contract shape

Use the following as a design example, not as an existing product schema or a certified relationship in the supplied data:

```yaml
id: relationship.base_to_reference
revision: 1
status: draft
source_resource: logical.base_record
target_resource: logical.reference_record

matching:
  predicates:
    - op: equals
      source_attribute: reference_key
      target_attribute: reference_key

multiplicity:
  expected: many_to_one
  required_target_uniqueness: [reference_key]
  evidence_state: not_yet_validated

uses:
  enrichment:
    enabled: true
    preserve: filtered_source_population
    required_join_type: left_outer
    relationship_predicate_scope: match_condition
  qualification:
    enabled: true
    lowering: exists_or_semi_join

aggregation:
  count_base_records_after_fanout: forbidden

certification_requirements:
  - compatible_identity_domains
  - target_uniqueness_verified
  - approved_business_relationship
```

Specify the actual schema and invariants your implementation would use. Avoid a configuration language so flexible that it becomes an unreviewed programming language.

The plan validator must consider entire paths, not just individual edges. A shortest path is not necessarily a valid path. Direction, roles, grain, temporal compatibility, and required intermediate preservation all matter.

## 11. Value resolution and identity resolution are separate services

Define a value resolver for requests such as:

```text
resolve_dimension_value(
    dimension_id = "contract.product_category",
    text = "Pet"
)
```

It should retrieve an authoritative value or mapping record, not invent a database literal.

Use a tiered strategy where appropriate: exact identifier, normalized canonical label, curated alias, lexical/fuzzy matching, then semantic retrieval and reranking. Small curated vocabularies may not need vectors at all.

Return explicit outcomes such as `resolved`, `ambiguous`, `not_found`, `not_authorized`, or `unavailable`, while ensuring user-visible errors do not leak restricted existence information.

For resolved values, preserve canonical identity, relevant source-specific codes, mapping revision, matching method, and evidence. A vector similarity score is not a calibrated probability of correctness. An exact label may still be ambiguous across scopes.

Record-level entity resolution is a separate concern. Explain how cross-system Producer identifiers would be matched using approved keys, crosswalks, or a governed resolution process. Do not infer identity solely from equal strings or similar names.

For both services, specify when clarification is required and when deterministic resolution is allowed.

## 12. Unstructured and semi-structured information must have native semantics

Do not model every PDF as a table or assume every JSON object has a stable relational schema.

For document resources, distinguish:

- Collection, document identity, document version, section, and passage/chunk.
- Physical location and content version from the stable logical identity of the document.
- Metadata fields from extracted claims and inferred entity references.
- Document date, event date, effective date, and ingestion date.
- Searchable content from citable source evidence.

A retrieved passage should retain the resource ID, version, location within the resource, relevant timestamps, access context, and extraction provenance needed to explain the answer. An extracted statement or entity link is an assertion with evidence, not automatically a certified business fact.

Describe resource capabilities such as lexical search, semantic search, metadata filtering, passage retrieval, extraction, and entity linking. No collection should advertise a capability that its current binding cannot actually support.

### Evidence and completeness

Distinguish:

- Exact structured computation over a defined population.
- Search results drawn from a covered corpus.
- Probabilistic extraction or entity linking.
- Narrative synthesis from retrieved evidence.

Top-k retrieval cannot justify claims such as “all matching producers,” an exact count, or “no documents mention this” unless an additional exhaustive, validated process establishes the necessary coverage. A count of extracted mentions is not automatically a count of real-world incidents.

Explain how coverage, freshness, source authority, conflicts, and incomplete evidence are represented and communicated. Keep document text untrusted: instructions embedded inside retrieved content must not change tool permissions, system policy, or execution rules.

## 13. Retrieval is an accelerator, not the canonical semantic store

Use an authoritative semantic repository. Derive the following as needed:

```text
Published semantic release
    |-- lexical search index
    |-- vector search index
    |-- graph traversal projection
    |-- context packages
    |-- compiled-plan caches
    `-- documentation / UI views
```

Explain indexing, incremental updates, release alignment, stale-index handling, and recovery. The system should survive an embedding-model replacement or search-engine migration without redefining business concepts.

A retrieval result should identify candidate semantic objects. Before execution, the runtime must load and validate the authoritative compatible revisions.

### Context assembly

Define a context package containing the relevant business definitions, logical resources, supported operations, resolved values, relationship IDs, and evidence about limitations.

Retrieve candidates broadly, then assemble the required dependency closure. Do not omit a mandatory policy, relationship predicate, or metric definition merely because it ranked below a top-k cutoff or exceeded a prompt budget.

Do not assume a collection of Markdown files alone provides deterministic enforcement. Agent Skills or similar instruction packages may explain how to use the platform, but canonical executable contracts and enforcement remain outside the prompt package.

Describe permission-aware retrieval and authorization revalidation before restricted content is sent to a model.

## 14. Agentic interpretation versus deterministic control

Make responsibility boundaries explicit:

| Activity | LLM/agent role | Deterministic responsibility |
|---|---|---|
| Interpret the question | Suggest intent and relevant concepts | Enforce scope and permitted tools |
| Resolve ambiguity | Explain alternatives or ask a question | Apply approved resolution rules; preserve unresolved state |
| Discover relevant semantics | Suggest search terms and candidates | Retrieve authorized, version-compatible records |
| Construct a plan | Propose typed operations and references | Validate structure, types, capabilities, policies, and dependencies |
| Choose relationships | Propose approved relationship IDs | Check path semantics and select allowed physical strategy |
| Generate executable work | Optional assistance within a controlled implementation | Compile supported operations and bind parameters |
| Execute | Request execution of an approved plan | Reauthorize, enforce budgets, execute, audit, and collect evidence |
| Explain results | Produce a grounded narrative | Supply result/evidence records and validate required answer properties |

A compiler cannot resolve an unknown business meaning by itself. An agent cannot authorize a query merely because it sounds reasonable. State which semantic decisions require business approval, which can be resolved by rules, and which remain uncertain.

### Runtime flow

Design a bounded workflow resembling:

```text
1. Accept request and establish server-controlled identity/scope.
2. Resolve domain and candidate business concepts.
3. Retrieve compatible logical implementations and dependencies.
4. Resolve values, temporal interpretation, and ambiguities.
5. Produce a typed semantic plan.
6. Validate semantics, permissions, capabilities, and execution limits.
7. Compile an executable plan for the selected bindings.
8. Execute through authorized adapters.
9. Collect results, provenance, coverage, and warnings.
10. Produce an answer grounded in those outputs.
```

Define states for clarification, unsupported requests, validation failure, source failure, partial results, and successful completion. Repair loops must be bounded and cannot weaken policies to obtain a successful execution.

### LangGraph and orchestration frameworks

Evaluate LangGraph or another workflow framework as an implementation option, not a foundational requirement.

Determine whether V1 needs durable execution, human approval pauses, retries, parallel work, resumability, or complex branching. A simple explicit state machine may be sufficient initially.

Keep workflow state, semantic plan contracts, tools, validation, and adapter interfaces independent of framework-specific objects. Explain how an orchestration framework could later be introduced or replaced without changing the business ontology or logical contracts.

Do not confuse the semantic relationship graph with an agent orchestration graph.

## 15. A small, typed semantic plan rather than unrestricted generated SQL

Define a versioned intermediate representation for the supported query operations. Start with the operations required by actual use cases, and provide a disciplined extension mechanism for later additions.

Likely operation families include projection, filtering, approved enrichment, relationship existence, aggregation, document retrieval, and evidence-backed combination. Do not claim V1 supports all of them.

A plan should reference semantic IDs and approved definitions rather than hard-coded physical tables, credentials, arbitrary filesystem paths, or unrestricted SQL fragments.

### Illustrative plan

The following illustrates a possible shape for counting active license records associated with a qualifying Pet contract. All referenced definitions must exist and be approved before execution:

```json
{
  "plan_schema_version": "1",
  "semantic_release_id": "release-example",
  "operation": "aggregate",
  "resource_ref": "logical.license_record",
  "measure_ref": "metric.license_record_count",
  "where": {
    "and": [
      {
        "predicate_ref": "rule.license_record.active"
      },
      {
        "exists": {
          "relationship_ref": "relationship.license_to_contract_via_producer",
          "where": {
            "equals": {
              "property_ref": "property.contract.product_category",
              "resolved_value_ref": "value-resolution-example"
            }
          }
        }
      }
    ]
  }
}
```

This is not a declaration that the supplied source tables already support those semantics. The design must establish identity compatibility, the relevant contract population, the active-license definition, and the product-category mapping first.

### Plan invariants

Validate typed inputs and outputs, permitted operators, referenced revisions, populations, grain, relationship use, filter scope, source capabilities, and authorization. Reject unknown required operations and extensions.

Keep server-controlled authorization context separate from model-authored fields. A model cannot establish its tenant or permissions by supplying them in JSON.

Prefer execution through a server-issued validated-plan handle whose associated plan, semantic release, parameter set, and execution context cannot be modified by the agent. Revalidate authorization at execution; possession of an old handle is not permanent permission.

If raw SQL is supported for expert or exploratory use, isolate that capability from the certified path and describe its separate permissions, validation, limitations, and result labeling. An AST parser alone does not establish arbitrary SQL equivalence or business correctness.

## 16. Adapter and execution contracts

Define a source-adapter interface that can cover discovery and the explicitly supported runtime operations without pretending all sources behave identically.

Discuss:

- Capability discovery at the binding level.
- Authentication and authorization propagation.
- Safe identifier resolution and parameter binding.
- Source-native query compilation or invocation.
- Result types, pagination, streaming, cancellation, timeouts, and limits.
- Source snapshots, consistency guarantees, and provenance.
- Error classification, retry safety, and observability.

SQL dialect, collation, case sensitivity, null handling, timezone behavior, and numeric precision can affect semantics. Explain how these differences are normalized or declared incompatible.

Treat stored procedures and arbitrary functions cautiously. Do not assume they are read-only because they return records. Model side effects and permit only reviewed capabilities.

### Heterogeneous execution

Explain how a future plan might query structured data, retrieve authorized document evidence, resolve approved entity links, and combine results. Specify where combination occurs, the expected sizes, and the permitted movement of identifiers or data.

Do not promise a globally consistent snapshot across unrelated systems unless the implementation actually provides one. Preserve source-specific observation times and surface material inconsistencies.

For V1, rejecting unsupported cross-source combinations is preferable to generating unbounded data transfers or ad hoc joins. A general distributed optimizer is not a prerequisite for extensibility.

## 17. Security and trust boundaries

Design security as enforcement in services, adapters, and source systems, not as instructions to the model.

Address metadata visibility, row/column restrictions, document ACLs, tenant isolation where applicable, credentials, model-context exposure, data egress, and result logging.

Prefer least-privilege read-only access for the initial system. Where source systems provide row- or column-level enforcement, preserve it rather than replacing it with a broad service credential and prompt instructions.

Define how authorization changes invalidate or constrain cached metadata, evidence, results, and executable plans. Cache keys must include the relevant access scope and semantic/source versions; cache sharing must not leak another user's results.

Distinguish business-use rules from access-control policy, even when both are called “policies.” Business stewards may define business meaning without obtaining permission to change database security.

Protect against prompt injection from documents, descriptions, source comments, tool outputs, and user-authored catalog content. These may supply information but cannot grant authority.

Keep future write/action capabilities behind a separate boundary with explicit enablement, approvals, side-effect descriptions, idempotency, and audit. Do not enable actions merely to make the initial model appear more general.

## 18. Publication, versions, lineage, and evolution

Provide a practical change lifecycle such as:

```text
DRAFT -> REVIEW -> VALIDATED -> PUBLISHED -> DEPRECATED -> RETIRED
```

Explain which transitions require business approval, technical approval, security approval, or automated checks. “Validated” should say what was tested; it should not imply universal truth.

### Version dimensions

Keep these distinct:

- Stable semantic identity.
- Semantic definition revision.
- Binding/implementation revision.
- Published compatible semantic release.
- Physical source schema/content version or observation.
- Business effective period and query as-of time.
- Search/index generation.
- API and plan-schema versions.

A runtime request should use a coherent compatible semantic release rather than a mixture of individually latest definitions. Preserve enough information to explain which rules and sources produced an answer; exact historical replay may still require source data snapshots that the system does not retain.

### Change propagation

Explain dependency tracking and impact analysis for changes to columns, keys, join cardinality, metric definitions, active rules, value mappings, permissions, and document extraction versions.

A changed label may be nonbreaking. A changed definition of “active,” grain, relationship population, or source authority may be semantically breaking even if the JSON schema is unchanged.

Discovery should detect drift, not silently rewrite approved semantics. Define warning, quarantine, fail-closed, and recertification behavior based on the type and severity of drift.

Recommend a minimal, reliable projection-update mechanism. A transactional outbox or similar approach may be sufficient; an enterprise event bus is not automatically necessary.

Explain additive evolution, deprecation, explicit migrations, extension namespacing, payload validation, and unknown-extension handling. Preserve extension points without building a fully dynamic meta-language in V1.

## 19. Ownership and authoring workflow

Propose clear accountability:

| Area | Primary accountability | Required collaboration |
|---|---|---|
| Technical discovery and connectors | Platform/data engineering | Source-system owners and security |
| Business definitions and terminology | Business owners/stewards | Data subject-matter experts |
| Logical models and computations | Data/analytics engineering | Business owners and source experts |
| Business-to-logical bindings | Joint business and technical approval | Governance as needed |
| Access policies | Security/data owners | Engineering and business stakeholders |
| Publication and operational reliability | Platform owner | Relevant approvers |

Business users should not need to write SQL to approve meaning. Technical users should not be allowed to change business meaning implicitly while optimizing an implementation.

Describe a workflow from discovered technical resources through logical modeling, business concept creation, binding, testing, approval, publication, and monitored use.

Acknowledge that authoring can start at any layer. A business concept may be defined before a dataset exists; a source may be discovered before anyone knows its business purpose.

Avoid creating three disconnected catalogs maintained independently. Explain how the interface presents cross-layer lineage, missing bindings, conflicts, and ownership without exposing unnecessary complexity.

## 20. Apply the design to the supplied producer domain

### Provided source structures

These are the supplied column lists. Types, constraints, uniqueness, value dictionaries, identity domains, and business definitions have not been established:

```text
Producer_Licenses
  Producer_Key
  License_Id
  License_Number
  State
  Status
  Termination_Date

Producer_Appointments
  Producer_Key
  Appointment_Id
  State
  Status
  Termination_Date

Agent_Contracts
  Producer_Key
  Contract_Id
  Contract_Type
  Status
  Product_Category

Agency_Contracts
  Producer_Key
  Contract_Id
  Contract_Type
  Status
  Product_Category
```

### Original example questions

1. “Fetch all individual records with no term date for Pet product category. For those records I need all active licenses.”
2. “What is the count of all license records that are active and have Pet contracts?”
3. “What is the count of all appointment records that are active and have Pet contracts?”

### Do not invent answers to these semantic questions

- What is an “individual record,” and where is the Producer/Individual source? No such table is included above.
- Which object's termination date is intended in question 1?
- Does “active” mean a status value, no termination date, an effective interval, a combination, or a different rule for each entity?
- Are License_Id and Appointment_Id unique globally, per source, per producer, per state, or per historical version?
- Does “count license records” count stored records or distinct business licenses?
- Do Agent_Contracts and Agency_Contracts represent compatible populations? Can their Producer_Key values be compared with license and appointment Producer_Key values?
- Does an agency contract qualify an individual producer directly, or is a missing agency-membership relationship required?
- Is Contract_Id globally unique, or does contract identity require source/type qualification?
- Is “Pet” a literal stored value, a code, a business category, or a hierarchy containing multiple values?
- Does “have Pet contracts” mean any Pet contract, an active Pet contract, an as-of-date contract, or a different definition? Do not add an active-contract filter merely because the license or appointment must be active.
- For question 1, should individuals with no active licenses remain in the answer? Should the response be nested by individual or flattened to one row per license?

### Required worked design

For each question, show:

1. Business interpretation and unresolved ambiguity.
2. Required business concepts, logical contracts, and bindings.
3. Input, intermediate, and output grain.
4. Typed semantic plan or an explicit blocked/clarification state.
5. Deterministic validation and compilation behavior.
6. Example execution shape under clearly labeled assumptions.
7. Provenance, answer shape, and expected failure modes.

For questions 2 and 3, demonstrate existential qualification through the producer relationship **only after that relationship and its identity domain are approved**. Explain why directly joining licenses or appointments to multiple contracts can multiply rows.

Use a synthetic fixture to make the issue concrete: a producer with three qualifying license records and four qualifying Pet contracts should contribute three license records, not twelve, under an approved existence-based interpretation. Likewise, two qualifying appointment records should contribute two appointments, not eight. These are test assumptions, not claims about the real data.

Do not fix fan-out by automatically adding `DISTINCT`. That can conceal an incorrect identity definition, remove legitimate records, or change the requested metric.

For a canonical contract model, evaluate union, separate role-specific resources, or explicit precedence only after population and identity compatibility are established. Preserve source identity where required. `EXISTS` can avoid count inflation but does not repair a semantically invalid cross-source association.

For question 1, preserve the distinction between qualifying individuals by Pet contracts and enriching them with active licenses. Explain left-preservation versus intentionally excluding individuals with no active license. The missing Individual source must remain explicit.

If SQL is shown, use an Azure SQL-compatible dialect and clearly identify assumed status codes and logical views. Do not present hypothetical SQL as runnable against the supplied schema without the missing definitions.

## 21. Also demonstrate a future mixed-source question

Use this illustrative question:

> “Which producers with active Pet contracts had licensing problems mentioned in compliance documents in the last six months?”

Show a plan that separates structured eligibility from document retrieval, evidence extraction, entity linking, and answer synthesis.

Resolve what “last six months” applies to, what “licensing problems” means, and whether the user wants documented mentions or verified incidents. Define the time anchor and timezone explicitly.

Explain which operations can be exact, which depend on retrieval coverage or extraction, and how the answer communicates that distinction. Do not turn matching passages into a definitive count of all incidents without a validated process that supports that claim.

Use this example to demonstrate extensibility, not to require its full implementation in V1.

## 22. Recommended implementation posture and phased delivery

Treat a **modular monolith with explicit internal contracts** as the default starting hypothesis. Use existing infrastructure when reasonable. Justify any departure.

Possible modules include registry, authoring/publication, discovery, resolution, planning, validation, compilation, adapters, policy enforcement, and evidence/audit. These are responsibility boundaries, not a mandate for separately deployed services.

### Design now, implement minimally

Identify the minimum durable foundations:

- Stable IDs and typed object/relationship categories.
- Explicit grain, identity, and source scope.
- Governed bindings rather than implicit name matching.
- A versioned plan format and bounded supported operations.
- Deterministic authorization, validation, and source execution boundaries.
- A coherent published semantic release with dependencies.
- Provenance, explicit ambiguity, and unsupported outcomes.
- Replaceable indexes, orchestration, and adapters.
- Tests tied to the semantics the application claims to support.

### Suggested V1 default

Use one business domain and one read-only relational adapter, with only the constructs required to answer the approved producer-domain questions. Start with curated values, controlled relationship contracts, a small plan language, and straightforward orchestration.

V1 must include real enforcement, not merely fields named `policy` and `certified` that the runtime ignores. Keep authoring and approval pragmatic; the workflow may initially be lightweight, but published definitions must still be distinguishable from drafts.

If a real initial requirement depends on documents, include a narrow document-retrieval slice with access control and citation provenance. Otherwise, define the adapter/evidence boundaries and defer document implementation.

### V2 and later

Candidate increments include additional adapters, temporal relationships, richer metrics, document evidence, record-level crosswalks, improved authoring UX, incremental index publication, and more durable orchestration.

Classify each capability as **V1 implementation**, **design boundary now / implementation later**, or **defer until evidence justifies it**. Specify adoption triggers rather than vague “future scalability” arguments.

Likely deferrals include a dedicated graph database, general-purpose distributed optimizer, full enterprise instance graph, autonomous ontology editing, arbitrary custom code execution, generalized write actions, and organization-wide master-data management.

Show at least three evolution scenarios without foundational replacement:

1. Add a new source adapter and logical binding.
2. Add a document-backed evidence capability.
3. Replace the search engine or orchestration framework.

Also show what still requires migration, review, or recertification. Extensibility does not mean change is free.

## 23. Interfaces and persistence: make the design concrete

Provide representative request/response contracts for the essential services, with typed errors and version handling. Candidate operations include:

```text
search_concepts(query, scope)
get_semantic_context(concept_ids, release_id)
resolve_dimension_value(dimension_id, text, context)
resolve_relationships(source, target, purpose, context)
validate_semantic_plan(plan, execution_context)
compile_semantic_plan(validated_plan)
execute_compiled_plan(plan_handle, execution_context)
retrieve_evidence(request, execution_context)
publish_semantic_release(change_set)
```

These names are illustrative, not mandatory APIs. Do not make every internal operation an LLM-visible tool. Explain the smaller agent tool surface and which operations are controlled by the runtime itself.

Describe the canonical persistence model, including typed records for concepts, logical resources, technical assets, bindings, relationship contracts, revisions/releases, approvals, test evidence, and dependencies as needed.

Use common identity and extension mechanisms where useful, but retain domain constraints. Explain what is normalized, what is stored as validated structured payload, and what is derived.

Provide enough schema detail to implement the first vertical slice. Do not produce dozens of speculative tables merely to cover every possible future feature.

## 24. Testing, evaluation, and observable correctness

Separate at least four forms of evaluation:

**Semantic contract tests:** Grain, key uniqueness, relationship multiplicity, source compatibility, temporal intervals, metric behavior, and value mappings.

**Compiler/runtime tests:** Plan validation, approved operation lowering, parameterization, policy enforcement, pagination, limits, and error handling.

**Agent evaluation:** Correct intent, clarification, semantic-object selection, plan construction, and grounded answers across paraphrases and ambiguity.

**Operational checks:** Source drift, index freshness, permission changes, coverage, latency, cost, and failure recovery.

Required adversarial and regression cases include:

- Fan-out from multiple contracts per producer.
- Unmatched optional relationships and null-rejecting filters.
- Multi-hop paths that accidentally remove preserved records.
- Duplicate or reused identifiers across sources and states.
- Conflicting definitions of active.
- Missing Individual/Producer source data.
- Agent and agency keys that are not interchangeable.
- Unauthorized columns, rows, metadata, or document passages.
- Stale or incompatible semantic/index releases.
- Unsupported operations and excessive data movement.
- Prompt injection in retrieved source content.
- Partial source failure and misleadingly complete answers.
- Document retrieval mistaken for exhaustive counting.

Use synthetic fixtures with independently specified expected results. Golden SQL strings alone are insufficient because multiple SQL forms can be equivalent and a memorized query can still answer the wrong question.

Define measurable acceptance criteria for the first vertical slice. Track semantic correctness separately from SQL syntax success, runtime success, retrieval relevance, latency, and model confidence.

## 25. Operational and nonfunctional design

State assumptions for scale, concurrent requests, source sizes, latency, freshness, and cost. Do not invent precise service-level objectives without business input; propose measurable targets and identify their owners.

Address request budgets, maximum rows, statement timeouts, cancellation, pagination, backpressure, rate limits, and bounded retries. Explain how source-side filtering and execution reduce data movement.

Keep security-sensitive logs and query results appropriately restricted and minimized. Audit which semantic release, bindings, rules, sources, and evidence produced each answer without indiscriminately storing all sensitive payloads.

Define failure behavior for unavailable sources, failed contract checks, stale data, ambiguous definitions, and partial evidence. The runtime must not silently swap to a different source or broaden a query when a preferred binding fails.

Discuss cache invalidation and consistency before promising performance benefits. Distinguish a cache of semantic metadata from a cache of sensitive user results.

Explain deployment and recovery for the proposed V1 footprint. Keep operational complexity proportional to actual requirements.

## 26. Learn from existing systems without copying their scope

Where external research is available, compare relevant patterns from official documentation for selected systems in these categories:

- Governance catalogs and business glossaries, such as Microsoft Purview and DataHub.
- Ontology-oriented platforms, such as Palantir Foundry and applicable Microsoft Fabric capabilities.
- Analytical semantic layers, such as Looker, dbt/MetricFlow, Snowflake semantic views, and Databricks metric views.
- Agent orchestration approaches, including LangGraph and simpler explicit workflows.
- Agent instruction packaging, including Agent Skills, when relevant to runtime context delivery.

These are research candidates, not assertions that each product provides every feature in this prompt. Verify actual capabilities, limitations, maturity, and preview status at the time of the research.

Cite primary documentation for factual product claims. Explain which pattern is useful, what problem it addresses, and why it should or should not be adopted here. Do not treat governance lineage, business ontology, analytics compilation, and agent orchestration as interchangeable capabilities.

Do not recommend buying or reproducing a large platform merely because it demonstrates a relevant pattern. If current documentation cannot be checked, label the comparison as unverified rather than inventing support.

## 27. Required architecture decisions

Provide concise architecture decision records for the most consequential choices. Each should include context, options, recommendation, rationale, tradeoffs, reversibility, and a revisit trigger.

At minimum, cover:

1. Boundaries and ownership of the three semantic layers.
2. Canonical storage and typed graph representation.
3. First-class bindings and record identity strategy.
4. Relationship contracts and fan-out-safe qualification/aggregation.
5. Typed plan versus unrestricted generated source queries.
6. Adapter capabilities and unsupported-operation behavior.
7. Published releases, drift handling, and compatibility.
8. Agent orchestration and whether LangGraph is justified now.
9. Structured results versus document evidence and extraction.
10. Security boundaries and the path from validated plan to execution.

Differentiate decisions that are expensive to reverse, decisions cheap to revisit, and decisions deliberately left open. Do not present framework or database preference as a first principle.

## 28. Required output format

Produce a self-contained Markdown design document suitable for an engineering team to implement the first functioning version.

Organize it into these deliverables:

### A. Executive recommendation

Explain the proposed architecture, the most important changes from a simple table/column catalog, the minimal V1, and what is intentionally deferred.

### B. Semantic-layer constitution

Define responsibilities, invariants, ownership, lifecycle, and prohibited shortcuts for each layer and cross-cutting service. Explain the why and what of each important construct.

### C. Detailed architecture

Include component descriptions, trust boundaries, data flow, canonical persistence, authoring/publication, retrieval, and runtime behavior. Provide readable Mermaid diagrams and explanatory prose. Include a relationship/model diagram and a runtime sequence diagram, not just deployment boxes.

### D. Concrete contracts and interfaces

Provide representative schemas for technical resources, business concepts, logical resources, bindings, relationship contracts, value resolution, semantic plans, execution results, and evidence. Show their version and identity relationships. Label illustrative formats clearly.

### E. Worked use cases

Walk through all three supplied producer questions and the mixed-source future question. Show assumptions, missing information, grain, plan construction, deterministic controls, example execution shapes, and answer limitations.

### F. Delivery plan

Provide V1, V2, and later capability boundaries; a suggested module/repository layout; implementation dependencies; and an ordered vertical-slice plan. Separate “model the boundary now” from “build the feature now.”

### G. Verification and operations

Provide contract tests, agent evaluations, security tests, release checks, observability, failure modes, and measurable acceptance criteria.

### H. Decisions, risks, and unresolved questions

Provide the architecture decision records, important tradeoffs, prioritized open questions, and a concise list of assumptions that prevent production certification.

Keep recommendations consistent across diagrams, schemas, examples, and implementation phases. A capability shown as deferred must not become an unexplained prerequisite for V1 elsewhere in the document.

## 29. Final self-review before delivering the design

Check the design against these questions:

- Can a business concept survive a table rename, source migration, or multiple implementations?
- Can a concept exist without a binding, and does the agent handle that honestly?
- Are business meaning, executable rules, and observed source metadata distinct but connected?
- Are identity, grain, relationship purpose, and filter scope explicit?
- Can the runtime prevent an unauthorized or semantically invalid plan from executing even when the model insists it is correct?
- Are cardinality and uniqueness assertions backed by evidence rather than assumed from labels?
- Does the model distinguish metadata graphs, record graphs, and workflow graphs?
- Can search and orchestration implementations be replaced without redefining the ontology?
- Can a new source be added without pretending it supports SQL or unrestricted joins?
- Are document evidence, probabilistic extraction, and exact computation distinguished?
- Are the missing producer-domain definitions surfaced rather than silently invented?
- Does publication prevent mixing incompatible definition revisions?
- Are security, source drift, and partial results handled explicitly?
- Is V1 small enough to build, but real enough to enforce its claimed contracts?
- Does each deferred feature have a preserved boundary rather than an unnecessary implementation?

End with a clear statement of the recommended first vertical slice and the smallest set of business decisions required to certify it.

---

## Final instruction

Design for **stable meaning and replaceable implementation**.

The desired result is not a maximal platform diagram. It is a practical architecture in which business concepts, logical contracts, technical bindings, and governed execution remain coherent as sources, agents, frameworks, and requirements evolve.
