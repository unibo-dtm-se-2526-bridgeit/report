---
title: Development
has_children: false
nav_order: 5
---

# Development

This section describes the engineering practices and implementation choices adopted for BridgeIT. It distinguishes the software currently implemented from capabilities that remain outside the present scope, so that the report does not claim features that are not yet available.

## DVCS

BridgeIT uses Git as its distributed version control system and GitHub as the collaboration platform. The project is hosted in the `unibo-dtm-se-2526-bridgeit` GitHub organization and is separated into two repositories:

- `BridgeIT-artifact`, containing the executable software, automated tests, workflow configuration, and implementation-oriented documentation;
- `report`, containing the Jekyll-based project report required by the course.

This separation keeps the software artifact independent from the final report while preserving links between the two repositories.

### Branching convention

The stable branches are used only for integrated work: the artifact repository currently uses `master`, while the report repository uses `main`. Development is performed on short-lived branches whose names communicate the purpose of the change. Prefixes used by the team include:

- `feature/` for new application capabilities, such as the AI Gateway and validation workflow;
- `fix/` for corrections to existing behavior;
- `docs/` for report and documentation changes;
- `chore/` for repository maintenance and non-functional changes;
- `style/` for formatting-only changes.

A branch should contain one coherent change and should be merged only after its associated pull request has been reviewed and the automated checks have completed successfully.

### Commit convention

Commit messages follow the Conventional Commits style:

```text
<type>: <short imperative description>
```

Examples used in the project include:

```text
feat: add AI Gateway and validation endpoints
fix: switch the default Gemini model
docs: complete design section
style: apply ruff format
chore: import roadmap into the artifact repository
```

The main prefixes are `feat`, `fix`, `docs`, `test`, `refactor`, `style`, `chore`, and `ci`. This convention makes the Git history easier to inspect and supports the release automation inherited from the course template through semantic-release.

### Pull requests, reviews, issues, and project tracking

Non-trivial changes are integrated through GitHub pull requests. A pull request provides a place to:

- explain the change and the requirements it addresses;
- inspect the commits and modified files;
- run the CI workflow;
- receive review comments from the other team member;
- apply follow-up commits before merging.

Review comments are resolved before integration whenever they identify an inconsistency, a missing test, or documentation that no longer reflects the implementation.

GitHub Issues are used for concrete development tasks. Issues can be assigned to a team member, associated with a milestone, and added to the organization-level `BridgeIT Roadmap` project. The project board tracks work through states such as `Backlog`, `Next`, `WIP`, `Review`, and `Done`. Milestones group related issues into larger increments such as domain modeling, persistence, AI integration, frontend development, testing, and release preparation.

## Implementation details

### Network protocols

The application uses HTTP as its application-level communication protocol.

The browser frontend communicates with the FastAPI backend through HTTP requests. HTTP was selected because BridgeIT exposes resource-oriented operations such as creating, retrieving, analysing, and validating a Requirement, and because it is directly supported by browsers, FastAPI, testing tools, and OpenAPI documentation.

HTTP itself runs over TCP, which provides reliable and ordered delivery. BridgeIT does not currently require low-latency datagrams, bidirectional streaming, message queues, or publish/subscribe communication; therefore UDP, WebSockets, MQTT, AMQP, and similar protocols would add complexity without addressing a present requirement.

The backend also communicates with the external Gemini service through HTTPS as handled by the `google-genai` client. HTTPS is required for external communication because requirement text and provider credentials must not be transmitted in plaintext. Local development may use plain HTTP on the loopback interface, while any non-local deployment should use HTTPS.

### In-transit data representation

Data exchanged between the frontend and backend is represented as UTF-8 JSON.

JSON was selected because it is natively supported by JavaScript, integrates directly with FastAPI and Pydantic, is human-readable during development, and is suitable for the relatively small request and response objects used by BridgeIT.

The API uses a common error representation:

```json
{
  "error": {
    "code": "requirement_not_found",
    "message": "Requirement not found."
  }
}
```

The `ApiError` contract is already applied to `GET /requirements/{id}` and is the required format for the analysis and validation endpoints as they evolve. A shared error structure keeps frontend handling consistent and prevents each endpoint from inventing a different response shape.

The Gemini adapter translates provider-specific responses into BridgeIT domain concepts. Provider response formats are therefore confined to the driven adapter and are not exposed directly to the domain or frontend.

### Database access

BridgeIT uses SQLite as its DBMS and queries it through Python's standard-library `sqlite3` driver. No ORM is currently used.

SQL was selected because the currently persisted data is structured and small, and the required operations are straightforward creation, lookup, and update operations. SQLite satisfies the course requirement for a DBMS while keeping local setup and automated testing lightweight.

Persistence is isolated behind the `RequirementRepository` port. The `SQLiteRequirementRepository` adapter is responsible for translating between domain objects and database rows. This separation prevents SQL, connection handling, and SQLite-specific details from leaking into the domain or application use cases.

The current schema contains the following table:

```sql
CREATE TABLE requirements (
    id TEXT PRIMARY KEY,
    text TEXT NOT NULL,
    status TEXT NOT NULL
);
```

AI analyses are not currently persisted. An analysis is recalculated when the relevant use case is invoked. This is a deliberate present-state decision and should not be confused with a claim that analysis history is already stored.

Database statements should use parameter binding rather than string interpolation. This reduces the risk of SQL injection and keeps values separate from SQL syntax.

### Authentication

Authentication is not implemented in the current version of BridgeIT. The prototype currently assumes a trusted user operating the application in a controlled academic or local environment.

Consequently, BridgeIT does not currently use OAuth, JWT, session cookies, or another identity protocol. The term *Business Analyst* identifies the intended user role in the requirements workflow, but it does not yet correspond to an authenticated account.

If user management is implemented in a later milestone, the authentication mechanism and credential-storage strategy must be documented and tested before the report claims that users are authenticated.

### Authorization

Authorization is likewise not implemented in the current version. There are no enforced RBAC or ABAC policies, and the current API does not distinguish permissions among Business Stakeholders, Business Analysts, or Software Engineers.

This limitation is explicit: the current domain focuses on the Requirement lifecycle, AI-assisted analysis, human validation, and traceability. Any future authorization model should be introduced only together with concrete functional requirements and acceptance tests.

### AI-provider interaction and failure handling

AI-assisted analysis is accessed through the abstract `AIGateway` port. `GeminiAIGateway` is the current driven adapter and uses the `google-genai` library with the `gemini-3.5-flash-lite` model.

The adapter retries transient provider failures caused by HTTP 429 and 503 responses. It performs at most three attempts and waits 1.5 seconds between attempts. Other failures are not retried indiscriminately, because retrying permanent errors would waste time and free-tier quota.

AI output never changes a Requirement automatically. The application layer converts the adapter result into an `AIAnalysis`, and an explicit validation use case is required before an AI suggestion can affect authoritative domain state.

## Technological details

### Programming languages

BridgeIT uses:

- **Python** for the domain model, application use cases, ports, adapters, FastAPI backend, database access, and automated tests;
- **HTML** for the frontend page structure;
- **CSS** for presentation;
- **JavaScript** for browser-side behavior and communication with the backend.

The frontend intentionally uses vanilla HTML, CSS, and JavaScript. A framework was not necessary for the current six-page interface, and avoiding one keeps the client lightweight and makes the HTTP interactions visible for academic evaluation.

### Backend frameworks and libraries

- **FastAPI** exposes the HTTP API and provides automatic OpenAPI generation.
- **Pydantic** validates and serializes transport-layer request and response models.
- **`sqlite3`** provides direct SQL access to SQLite without an ORM.
- **`google-genai`** provides the client used by the Gemini adapter.
- **Python abstract interfaces** define the `RequirementRepository` and `AIGateway` ports, keeping use cases independent from concrete adapters.

FastAPI and Pydantic belong to the driving-adapter boundary. They must not be imported into domain entities or value objects.

### Testing and static verification

- **pytest** is the automated testing framework.
- **FastAPI `TestClient`** exercises HTTP routes in system-facing tests.
- **`unittest.mock.MagicMock`** replaces the Gemini client in tests that must not call the real provider.
- An **in-memory fake repository** replaces SQLite where a fast and deterministic test double is more appropriate.
- **Ruff** performs linting and formatting.
- **Mypy** performs static type checking.
- **Poetry** manages Python dependencies and the project environment.
- **Poe the Poet** exposes repeatable project commands on top of Poetry.

At the time this section was drafted, the project reported 58 automated tests. The definitive test distribution, success rate, and coverage results belong in the [Validation](../05-validation/) section and must be updated from actual CI output before final submission.

### Frontend

The frontend currently contains six pages covering:

1. application entry point and health information;
2. Requirement creation;
3. Requirement listing and visualization;
4. AI-assisted analysis;
5. human validation;
6. help and user guidance.

Browser-side JavaScript calls the FastAPI API and renders domain-relevant results. A BridgeIT logo is used as a qualitative state indicator rather than presenting a misleading numerical AI score.

### Build, CI/CD, and release tooling

GitHub Actions executes the project's automated checks. The workflow includes preliminary static checks and a Python test matrix. Downstream deployment or release jobs run only when their dependencies succeed.

The course template also includes Node.js-based semantic-release tooling. Node.js and npm are therefore build and release dependencies, not implementation technologies used by the BridgeIT application itself. Conventional Commits allow semantic-release to determine version changes and produce release metadata.

### Documentation technology

The dedicated report repository uses Markdown, Jekyll, and GitHub Pages. The report is organized into numbered sections provided by the course template and is published as a static website through its GitHub Actions workflow.

### External services

Google Gemini is the only external runtime service currently integrated into BridgeIT. It is accessed exclusively through `GeminiAIGateway`, so no domain object or application use case depends directly on provider-specific SDK types.

The project is intended to use a free GenAI tier. Provider quotas and rate limits are therefore treated as engineering constraints. The retry policy addresses transient overload conditions; further quota-saving measures, such as caching identical analysis requests, should only be documented as implemented after the corresponding code and tests exist.

## Current implementation boundary

The choices described above reflect the implementation currently known at the time of writing. In particular:

- SQLite persistence for Requirements is implemented through a repository adapter;
- the AI Gateway, analysis use case, validation use case, and corresponding FastAPI endpoints have been developed;
- the six-page vanilla frontend has been developed;
- authentication and authorization are not implemented;
- AI Analysis persistence and caching are not currently claimed.

This section must be revised if later milestones introduce user accounts, authorization rules, a different database schema, analysis persistence, caching, or additional deployment technologies.

