---
title: Requirements
has_children: false
nav_order: 3
---

# Requirements

## User stories

The stories below translate the project's functional requirements into concrete usage scenarios, from the perspective of BridgeIT's primary personas.

**Personas**

- **Business Stakeholder** — a domain expert who originates a requirement in natural language, without technical background.
- **Requirements Engineer / Business Analyst** — refines, analyses, and validates requirements before they become authoritative.
- **Software Engineer** — consumes validated, structured requirements and their traceability to derived artifacts.

**US-01 — Submitting a requirement** *(→ FR-01)*
> As a Business Stakeholder, I want to submit a requirement in natural language, so that my intent is captured as a structured artifact I can later review.

**US-02 — Requesting AI-assisted analysis** *(→ FR-02)*
> As a Requirements Engineer, I want the platform to analyze a requirement for ambiguity or incompleteness, so that I can identify issues before the requirement is validated.

**US-03 — Clarifying a requirement after analysis** *(→ FR-03)*
> As a Business Stakeholder or Requirements Engineer, I want to revise a requirement's wording after an analysis has surfaced an issue, so that the requirement can be improved before validation.

**US-04 — Reviewing requirement quality** *(→ FR-04)*
> As a Requirements Engineer, I want to see a quality indication for a requirement, so that I can decide whether it needs further clarification.

**US-05 — Validating an AI suggestion** *(→ FR-05)*
> As a Business Stakeholder or Requirements Engineer, I want to explicitly approve, edit, or reject any AI-generated suggestion, so that no interpretation of my requirement becomes authoritative without my consent.

**US-06 — Inspecting traceability** *(→ FR-06)*
> As a Software Engineer, I want to inspect which artifacts are linked to a given requirement, so that I understand the origin of what I am implementing.

**US-07 — Creating a derived artifact** *(→ FR-07)*
> As a Software Engineer, I want to create a derived artifact from a validated requirement, so that I can begin implementation work with a clear, traceable link back to its source.

![BridgeIT Use Case Diagram](../../pictures/bridgeit-use-case-diagram.svg)

## Requirements analysis

### Functional requirements

**FR-01 Requirement Creation** — The system shall allow a stakeholder to submit a requirement expressed in natural language and have it represented as a structured, persistent artifact.
*Acceptance criteria:* given a non-empty description, submitting it stores a Requirement with a unique identifier and an initial status; retrieving it later returns the stored text unchanged.

**FR-02 AI-Assisted Requirement Analysis** — The system shall use the AI Gateway to analyze a submitted requirement's text and identify potential quality issues.
*Acceptance criteria:* given a Requirement eligible for analysis, requesting one produces an AI Analysis associated with its identifier, without altering the Requirement's stored text or status.

**FR-03 Requirement Clarification** — The system shall allow a stakeholder to revise a requirement's wording in response to issues identified during analysis.
*Acceptance criteria:* revising a Requirement's text keeps its identifier unchanged; the relationship between the original submission and the revision remains identifiable.

**FR-04 Requirement Quality Evaluation** — The system shall produce a Quality Indication for a requirement, to help stakeholders judge its readiness for validation.
*Acceptance criteria:* a Quality Indication distinguishes at least "ready for validation" from "needs clarification"; producing one does not, by itself, change the Requirement's status.

**FR-05 Human Validation of AI Suggestions** — The system shall require an explicit human decision (approve, edit, or reject) before any AI-generated suggestion affects the authoritative state of a requirement.
*Acceptance criteria:* an AI Analysis awaiting review does not affect the Requirement's authoritative state until a human decision is recorded; the recorded decision is retrievable together with the Requirement.

**FR-06 Traceability Link Management** — The system shall allow the creation and inspection of traceability links between a requirement and the artifacts derived from it. *(Optional stretch goal — not currently implemented; see [Future work](../12-future/).)*
*Acceptance criteria:* a Traceability Link between a Requirement and a Derived Artifact is retrievable by querying either side of the relationship.

**FR-07 Derived Artifact Creation** — The system shall allow a validated requirement to be used as the basis for creating a derived, structured artifact, preserving its link to the source requirement. *(Optional stretch goal — not currently implemented.)*
*Acceptance criteria:* an Artifact created from a Requirement whose status is "Validated" retains an explicit reference to it; creation is refused for a Requirement not yet Validated.

### Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-01 | **Maintainability** — changes to one concern (e.g. persistence, AI provider) shall not require changes to unrelated parts of the system. |
| NFR-02 | **Testability** — domain and application logic shall be verifiable through automated tests independently of external infrastructure. |
| NFR-03 | **Modularity** — the system shall be composed of clearly bounded modules with explicit responsibilities, consistent with DDD and Hexagonal Architecture. |
| NFR-04 | **Replaceable AI provider** — AI capability shall be accessed through an abstraction (the AI Gateway) allowing the provider to be replaced without affecting domain or application logic. |
| NFR-05 | **Security** — AI provider credentials shall never be embedded in source code or version control. |
| NFR-06 | **Extensibility** — new requirement-derived artifact types or AI-assisted capabilities shall be introducible without restructuring existing modules. |
| NFR-07 | **Reliability** — failures in an external dependency (e.g. AI provider unavailability) shall not corrupt or lose previously stored Requirement data. |
| NFR-08 | **Error handling** — failures in external dependencies shall surface a clear, actionable indication rather than an unhandled failure. |
| NFR-09 | **Configuration management** — environment-specific settings (e.g. AI provider credentials) shall be externalized from the codebase. |
| NFR-10 | **Observability** — significant domain events shall be logged to support debugging, without logging sensitive content in plaintext. |

### Implementation requirements

The following technology choices are constraints imposed by the course, not free design decisions, and are recorded here (rather than left to emerge from design) for that reason:

- **SQLite** as the DBMS, confirmed as an explicit course requirement once the team grew to two members.
- **A web or desktop frontend**, confirmed as an explicit course requirement for the same reason.

All other technology choices within these constraints (e.g. `sqlite3` over an ORM, vanilla HTML/CSS/JS over a frontend framework) are design decisions, not implementation constraints, and are discussed with their rationale in the [Design](../03-design/) section.

### Glossary

See [Domain Terminology](../01-concept/) for the project's ubiquitous language (Requirement, AI Analysis, Derived Artifact, Traceability Link, Quality Indication).
