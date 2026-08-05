---
title: Design
has_children: false
nav_order: 4
---

# Design

This chapter describes the design of BridgeIT, the strategies adopted to satisfy the requirements identified during analysis, and the rationale behind the main technical choices.

## Architecture

BridgeIT adopts a **Hexagonal Architecture** (Ports and Adapters), combined with **Domain-Driven Design** principles for the core domain model.

### Why Hexagonal Architecture

A plain layered architecture was considered but discarded: in a layered style, the persistence and AI-provider details tend to leak upward into business logic (e.g. domain code importing a SQL library, or knowing about Gemini's response format). Hexagonal Architecture instead makes the domain and application layers depend on abstract **ports**, and pushes every concrete technology (SQLite, Google Gemini) to the edges as **adapters**. This was considered worth the extra indirection for two reasons specific to this project: the AI provider was expected to be revisited during development (the team did in fact switch Gemini models mid-project without touching any use case or domain code), and the domain rules around a requirement's lifecycle needed to be unit-testable without a database or network connection.

### High-level overview

Four layers, from the inside out:

1. **Domain** (`bridgeit/domain/`) — pure business logic and rules, no framework or infrastructure dependency.
2. **Application** (`bridgeit/application/`) — use cases that orchestrate the domain, plus **ports**: abstract interfaces the application depends on but does not implement.
3. **Adapters** (`bridgeit/adapters/`) — driving adapters that translate an external protocol into calls to use cases; currently a single **FastAPI** adapter.
4. **Infrastructure** (`bridgeit/infrastructure/`) — driven adapters implementing the ports declared in the application layer: `SQLiteRequirementRepository` for persistence, `GeminiAIGateway` for the external AI service.

Dependency direction (always inward):

```
Adapters (FastAPI)  ─────depends on─────►  Application (Use Cases)  ─────depends on─────►  Domain
                                                    ▲
                                                    │ implements
                                                    │
Infrastructure (SQLite, Gemini)  ──────────────────┘
```

Nothing in `domain/` or `application/` imports from `adapters/` or `infrastructure/` — only the reverse. This is what allows the persistence technology and the AI provider to be swapped without touching business logic, and what makes the domain and use cases testable in isolation from any external system.

### Responsibilities of each component

- **Domain** — models `Requirement` as the aggregate root of the "requirement lifecycle" bounded context, together with its value objects and invariants (see Modelling below). Owns all business rules, including which status transitions are valid.
- **Application / Use Cases** — one use case per user-facing action (`AnalyseRequirementUseCase`, `ValidateRequirementUseCase`, plus requirement submission/retrieval). A use case fetches a `Requirement` through its repository port, asks the domain to perform an operation, asks the AI gateway port for an analysis when needed, and persists the result. Use cases contain **no** HTTP or SQL code.
- **Application / Ports** — abstract interfaces (`RequirementRepository`, `AIGateway`) defining *what* the application needs from the outside world, without saying *how*. `AIGatewayError` is the single error type use cases need to know about, regardless of which AI provider is behind it.
- **Adapters (driving)** — the FastAPI routes in `bridgeit/adapters/api/` (`main.py`, `analysis_router.py`) translate incoming HTTP requests into use-case calls and translate results (or exceptions) back into HTTP responses. `errors.py` defines a single JSON error shape (`{"error": {"code", "message"}}`) shared by every endpoint, via `ApiError`.
- **Infrastructure (driven)** — `SQLiteRequirementRepository` implements `RequirementRepository` on the standard-library `sqlite3` module; `GeminiAIGateway` implements `AIGateway` on Google's `google-genai` client library, including retry logic for transient failures (see Development chapter).

## Infrastructure

BridgeIT is **not a distributed system**: it runs as a single Python process (the FastAPI application) with an embedded SQLite database file on the same machine, serving a browser-based frontend running on the same host during development.

The only component outside the process boundary is the **Google Gemini API**, called over HTTPS as a third-party dependency for AI-assisted requirement analysis. There is no load balancer, message broker, or service discovery: a single instance is sufficient for the scope of this academic project.

## Modelling

### Domain-driven design (DDD) modelling

The project has a single bounded context, **Requirement Lifecycle**, covering submission of a requirement, its AI-assisted analysis, and its human validation.

Domain concepts:

- **`Requirement`** (entity / aggregate root) — identified by an id; holds a `RequirementText` and a `RequirementStatus`; the only object allowed to change its own status, and only through valid transitions (`Submitted → Analyzed → Validated / Clarified / Rejected`). An invalid transition raises `InvalidStateTransitionError`.
- **`RequirementText`** (value object) — wraps the raw requirement text.
- **`RequirementStatus`** (value object / enum) — the finite set of lifecycle states above.
- **`AIAnalysis`** (value object) — the outcome of an AI analysis: a `QualityScore` plus a list of issues.
- **`QualityScore`** (value object / enum) — `ready_for_validation` or `needs_clarification`. Deliberately a **binary category, not a numeric score**: a fabricated percentage would suggest a precision the AI analysis doesn't actually have.

Repository: `RequirementRepository` is the single repository of the bounded context, with two implementations — `SQLiteRequirementRepository` for production and an in-memory fake for tests (see Validation chapter).

No explicit domain events are used: the workflow is a simple, synchronous request/response cycle (submit → analyse → validate) rather than an event-driven one, so an event bus would add complexity without a corresponding benefit at this scale.

### Object-oriented modelling

| Type | Kind | Role |
|---|---|---|
| `Requirement` | Entity | Aggregate root; owns `RequirementText` + `RequirementStatus`; enforces transition rules |
| `RequirementText` | Value object | Wraps requirement text |
| `RequirementStatus` | Enum | Submitted / Analyzed / Validated / Clarified / Rejected |
| `AIAnalysis` | Value object | Holds `QualityScore` + issues list |
| `QualityScore` | Enum | ready_for_validation / needs_clarification |
| `RequirementRepository` | Port (abstract) | save / get_by_id |
| `AIGateway` | Port (abstract) | analyse(text) returns `AIAnalysis`; raises `AIGatewayError` |
| `AnalyseRequirementUseCase` | Use case | Orchestrates repository + AI gateway for FR-02/FR-04 |
| `ValidateRequirementUseCase` | Use case | Orchestrates repository + domain transition for FR-05 |
| `SQLiteRequirementRepository` | Adapter (driven) | Implements `RequirementRepository` on plain `sqlite3` |
| `GeminiAIGateway` | Adapter (driven) | Implements `AIGateway` on Google's `google-genai` client, with retry |
| `ApiError` | Adapter (driving) | Shared exception, produces a structured JSON error response |

### In case of a distributed system

Not applicable — see Infrastructure above.

## Interaction

All interaction is synchronous request/response over HTTP (FastAPI). The two most significant flows:

**Analyse a requirement (FR-02, FR-04)** — `POST /requirements/{id}/analyse`
1. The route handler calls `AnalyseRequirementUseCase.execute(id)`.
2. The use case fetches the `Requirement` via `RequirementRepository`.
3. It asks `AIGateway.analyse(text)` for an `AIAnalysis`.
4. It persists the updated requirement via the repository.
5. The route translates the result (or any `RequirementNotFoundError` / `InvalidStateTransitionError` / `AIGatewayError`) into the appropriate HTTP status and JSON body.

**Validate a requirement (FR-05)** — `POST /requirements/{id}/validate`
1. The route handler calls `ValidateRequirementUseCase.execute(id, decision, modified_text)`.
2. The use case fetches the `Requirement` and asks the domain to transition its status according to the decision (`approve` / `edit` / `reject`).
3. It persists the result via the repository.
4. The route translates the result or exception into an HTTP response.

## Behaviour

`Requirement` is the only stateful domain object, and its state can only change through its own methods — never by an adapter setting a field directly. This is what lets `InvalidStateTransitionError` be raised consistently regardless of which adapter triggers the transition.

`GeminiAIGateway` is functionally stateless between calls, but wraps each call in **retry logic**: up to 3 attempts with a 1.5-second wait, only for transient errors (HTTP 429 rate-limit and 503 service-unavailable); a non-retryable error such as 401 (invalid key) fails immediately rather than wasting two more attempts on something that cannot succeed on retry (see Development chapter for the reasoning).

State is updated and persisted synchronously, in the same request, immediately after each use case operation — there is no separate background process or job queue.

## Data-related aspects

Persistent data is minimal by design: a single SQLite table stores each requirement's `id`, `text` and current `status`.

```sql
CREATE TABLE IF NOT EXISTS requirements (
    id     TEXT PRIMARY KEY,
    text   TEXT NOT NULL,
    status TEXT NOT NULL
);
```

The result of an AI analysis (`quality_indication`, `issues`) is **not** persisted — it is returned directly in the API response and recomputed on demand the next time `/analyse` is called. This keeps the schema minimal and avoids the added complexity of invalidating a stored analysis if the requirement text later changes.

Plain `sqlite3` (standard library) was chosen over an ORM such as SQLAlchemy to keep the persistence adapter as thin as possible: the goal of Hexagonal Architecture is for infrastructure to be swappable, and a thin adapter with no ORM-specific base classes leaking into the domain achieves that more directly than an ORM would, at the scale of this project.

Only the persistence adapter (`SQLiteRequirementRepository`) ever queries the database, and it is reached exclusively through the `RequirementRepository` port — never called directly by a use case or a route (with the exception of a temporary, explicitly flagged shortcut in `main.py`'s pre-existing `/requirements` routes, predating the AI Gateway work, earmarked for the same refactor).

Concurrent access is handled by SQLite itself (file-level locking); the project's scope — a single local instance used by one Business Analyst at a time — does not require any additional concurrency handling on top of that.
