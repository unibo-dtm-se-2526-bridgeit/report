---
title: Design
has_children: false
nav_order: 4
---

# Design

This chapter describes the design of BridgeIT, the strategies adopted to satisfy the requirements identified during analysis, and the rationale behind the main technical choices.

## Architecture

BridgeIT adopts a **Hexagonal Architecture (Ports and Adapters)** combined with **Domain-Driven Design (DDD)** principles for the core domain model.

### Why Hexagonal Architecture

A plain layered architecture was considered but discarded because persistence and AI-provider details could more easily leak into the business logic. In BridgeIT, the domain and application layers depend on abstract **ports**, while concrete technologies such as SQLite and Google Gemini are placed in infrastructure adapters.

This separation was considered particularly useful because the AI provider may change during development without requiring modifications to the application use cases or domain logic. It also allows the main business rules to be tested independently from databases, HTTP requests, and external AI services.

### High-level overview

The system is organized into four main layers:

1. **Domain** (`bridgeit/domain/`) — contains the business entities, value objects, lifecycle rules, and invariants. It has no dependency on FastAPI, SQLite, or the AI provider.
2. **Application** (`bridgeit/application/`) — contains the main use cases and the abstract ports required by the application.
3. **Adapters** (`bridgeit/adapters/`) — contains the driving HTTP adapter implemented with FastAPI and the related request/response handling.
4. **Infrastructure** (`bridgeit/infrastructure/`) — contains the concrete driven adapters for persistence and AI integration, namely SQLite and Google Gemini.

The dependency direction is inward: the outer layers depend on the abstractions and business logic of the inner layers, while the domain does not depend on external technologies.

The application layer therefore acts as the coordination point between the domain model and the external services. The concrete infrastructure implementations satisfy the interfaces defined by the application ports.

### Responsibilities of each component

* **Domain** — models `Requirement` as the aggregate root of the requirement lifecycle, together with its value objects and invariants. It owns the business rules governing valid state transitions.
* **Application / Use Cases** — the main business operations are represented by `SubmitRequirementUseCase`, `AnalyseRequirementUseCase`, and `ValidateRequirementUseCase`. These use cases coordinate repositories, the AI gateway, and domain objects, without containing HTTP or SQL-specific code. Requirement retrieval is currently implemented directly in the API layer as a thin repository operation.
* **Application / Ports** — `RequirementRepository` and `AIGateway` define the abstractions required by the application layer. The application depends on these interfaces rather than on concrete infrastructure technologies. `AIGatewayError` provides an implementation-independent error type for AI-related failures, while `RequirementNotFoundError` (in `bridgeit/application/errors.py`) is shared by the use cases that look up a requirement.
* **Adapters (driving)** — the FastAPI routes in `bridgeit/adapters/api/` translate HTTP requests into application operations and convert domain/application errors into HTTP responses. The request/response DTOs, which depend on Pydantic, live in this adapter (`bridgeit/adapters/api/dto.py`) rather than in the application layer. `ApiError` provides a common structured error representation.
* **Infrastructure (driven)** — `SQLiteRequirementRepository` implements the persistence port using Python's standard-library `sqlite3` module, while `GeminiAIGateway` implements the AI port using Google's `google-genai` client.

## Infrastructure

BridgeIT is **not a distributed system**. It runs as a single Python process containing the FastAPI backend and uses a local SQLite database file for persistent storage.

The browser-based frontend communicates with the backend through HTTP. During local development, the frontend can be opened as local files or served from a local development server.

The only external service involved in the application workflow is the **Google Gemini API**, which is accessed over HTTPS for AI-assisted requirement analysis.

The project does not require a load balancer, message broker, service discovery mechanism, or multiple backend instances. A single local application instance is sufficient for the scope of the project.

## Modelling

### Domain-driven design (DDD) modelling

The project contains a single bounded context, **Requirement Lifecycle**, which covers the submission, AI-assisted analysis, refinement, and human validation of requirements.

The main domain concepts are:

* **`Requirement`** — entity and aggregate root, identified by a unique id. It contains the current `RequirementText` and `RequirementStatus` and is responsible for enforcing valid lifecycle transitions.
* **`RequirementText`** — immutable value object that contains the natural-language text of the requirement.
* **`RequirementStatus`** — enumeration representing the requirement lifecycle: `Submitted`, `Analyzed`, `Clarified`, `Validated`, and `Rejected`.
* **`AIAnalysis`** — immutable value object containing the result of an AI-assisted analysis, namely a quality indication and a list of identified issues. The current implementation does not persist AI analysis results separately.
* **`QualityScore`** — enumeration representing the two quality indications produced by the AI analysis: `ready_for_validation` and `needs_clarification`.

The requirement lifecycle is controlled by the `Requirement` aggregate itself. The implemented transitions are:

`Submitted → Analyzed → Validated`

or:

`Submitted → Analyzed → Clarified → Analyzed`

and:

`Submitted → Analyzed → Rejected`

After being validated or rejected, a requirement reaches a final state. A clarified requirement must be analysed again before another human validation decision can be recorded.

The `RequirementRepository` is the only persistence abstraction used by the application. It has two implementations: `SQLiteRequirementRepository` for the actual application and an in-memory fake used by tests.

### Object-oriented modelling

| Type                          | Kind                    | Role                                                                                        |
| ----------------------------- | ----------------------- | ------------------------------------------------------------------------------------------- |
| `Requirement`                 | Entity / Aggregate Root | Owns the current requirement text and lifecycle status and enforces valid state transitions |
| `RequirementText`             | Value Object            | Encapsulates the requirement text                                                           |
| `RequirementStatus`           | Enum                    | Represents Submitted / Analyzed / Clarified / Validated / Rejected                          |
| `AIAnalysis`                  | Value Object            | Contains the AI quality indication and identified issues                                    |
| `QualityScore`                | Enum                    | Represents `ready_for_validation` or `needs_clarification`                                  |
| `RequirementRepository`       | Port                    | Abstract persistence interface with `save` and `get_by_id`                                  |
| `AIGateway`                   | Port                    | Abstract interface for requesting an AI analysis                                            |
| `SubmitRequirementUseCase`    | Use Case                | Creates and persists a new requirement                                                      |
| `AnalyseRequirementUseCase`   | Use Case                | Retrieves a requirement, requests an AI analysis, updates its status, and persists it       |
| `ValidateRequirementUseCase`  | Use Case                | Applies the human validation decision and persists the resulting state                      |
| `SQLiteRequirementRepository` | Adapter (driven)        | Implements persistence using SQLite                                                         |
| `GeminiAIGateway`             | Adapter (driven)        | Implements the AI gateway using Google Gemini                                               |
| `ApiError`                    | Adapter (driving)       | Represents structured API errors returned to clients                                        |

### In case of a distributed system

This aspect is not applicable because BridgeIT is implemented as a single-process application with a local database and one external AI service.

## Interaction

All application interactions are synchronous request/response operations over HTTP.

### Analyse a requirement

The main analysis flow is initiated through:

`POST /requirements/{id}/analyse`

The interaction proceeds as follows:

1. The FastAPI route receives the request.
2. The route invokes `AnalyseRequirementUseCase`.
3. The use case retrieves the requirement through `RequirementRepository`.
4. The use case asks the `Requirement` aggregate whether an analysis is allowed in its current status (`ensure_can_be_analyzed`); if not, the request is rejected with `409` **before** any call to the AI provider, so no free-tier quota is consumed.
5. The use case sends the current requirement text to `AIGateway`.
6. The Gemini adapter performs the AI analysis and returns an `AIAnalysis`.
7. The use case updates the requirement status from `Submitted` or `Clarified` to `Analyzed`.
8. The updated requirement is persisted through the repository.
9. The API returns the analysis result, including the quality indication and any identified issues.

The AI result is therefore returned to the client, while the authoritative requirement state is updated separately through the domain lifecycle.

### Validate a requirement

Human validation is initiated through:

`POST /requirements/{id}/validate`

The interaction proceeds as follows:

1. The FastAPI route receives the validation decision.
2. The route invokes `ValidateRequirementUseCase`.
3. The use case retrieves the requirement through `RequirementRepository`.
4. The domain object applies the selected decision:

   * `approve` → `Validated`;
   * `edit` → `Clarified`, replacing the current text;
   * `reject` → `Rejected`.
5. The updated requirement is persisted through the repository.
6. The API returns the resulting requirement status.

Only this explicit human validation action can produce the final statuses `Validated` or `Rejected`.

## Behaviour

`Requirement` is the only stateful domain object and controls its own lifecycle. External layers cannot arbitrarily assign a new status; instead, they must invoke domain operations such as `mark_analyzed`, `clarify`, `validate`, or `reject` (and `ensure_can_be_analyzed` to check, without changing state, whether an analysis is allowed).

When an invalid transition is attempted, the domain raises `InvalidStateTransitionError`. This rule is therefore independent of whether the operation originated from the web interface or from another adapter.

The `GeminiAIGateway` is functionally stateless between requests. It includes retry logic for selected transient failures, specifically rate limiting (`429`) and service unavailability (`503`), while non-retryable errors such as authentication failures are returned immediately.

State updates are synchronous: the requirement is updated and persisted during the same request that performs the corresponding use case operation. There is no background worker, asynchronous job queue, or event-driven mechanism.

## Data-related aspects

Persistent data is intentionally minimal. A single SQLite table stores the current state of each requirement:

```sql
CREATE TABLE IF NOT EXISTS requirements (
    id     TEXT PRIMARY KEY,
    text   TEXT NOT NULL,
    status TEXT NOT NULL
);
```

The stored data therefore consists of:

* the requirement identifier;
* the current requirement text;
* the current lifecycle status.

The application does **not** persist a complete revision history. When a Requirements Engineer edits a requirement, the existing text is replaced by the revised text while the identifier is preserved.

The result of an AI analysis is also **not persisted**. The `AIAnalysis` object is returned directly by the analysis endpoint and is recomputed when the requirement is analysed again. This keeps the current schema small and avoids introducing a separate persistence model for analysis results.

SQLite was implemented using the standard-library `sqlite3` module instead of an ORM such as SQLAlchemy. This keeps the persistence adapter lightweight and prevents ORM-specific concepts from leaking into the domain or application layers.

The persistence adapter is the only component that directly queries the SQLite database. The application layer depends only on the `RequirementRepository` abstraction.

The current implementation contains a small architectural shortcut in the requirement retrieval endpoint: `GET /requirements/{id}` accesses the repository directly instead of using a dedicated retrieval use case. This does not affect the domain rules, but it represents a minor technical-debt item that could be refactored in a future iteration.

Concurrency is delegated to SQLite's own locking mechanism. Given the scope of the project, which assumes a single local instance and one primary Requirements Engineer at a time, no additional concurrency infrastructure is required.
