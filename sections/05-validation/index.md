---
title: Validation
has_children: false
nav_order: 6
---

# Validation

## Testing Approach

Tests were developed incrementally, alongside each piece of functionality, rather than all at once at the end of the project. In most cases, a new endpoint or use case was first verified manually (through Swagger UI or the frontend) to confirm its intended behavior, and then covered by an automated test reproducing that same check, a "verify, then automate" pattern rather than strict Test-Driven Development's red-green-refactor cycle.

The test suite follows a **mirrored structure**: each production module has a matching test module under `tests/`, replicating the package tree (`tests/domain/`, `tests/application/`, `tests/infrastructure/`, `tests/adapters/api/`). This keeps gaps visible, a production module without a corresponding test module has not been tested, and keeps each failing test pointing at exactly one file to investigate.

The **domain layer** (pure Python, no framework dependency) is the most densely tested part of the suite, since its business rules can be asserted directly and cheaply. **Adapters** (the SQLite repository, the Gemini AI Gateway) are tested against their **port interfaces**, so the tests describe expected behavior rather than internal implementation — a change in how an adapter is implemented internally does not break its tests without reason.

## Framework

The framework used is **pytest**. It is preferred over the standard library's `unittest` for two reasons: plain `assert` statements give rich failure introspection with no boilerplate assertion methods to remember; and its fixture system allows composable setup and easy injection of test doubles — used extensively for isolated, disposable SQLite databases (via the built-in `tmp_path` fixture) and for mocked external clients.

## Quality Gate

Coverage is measured with `poetry run poe coverage`, which runs `pytest` with `coverage`, scoped to report only on production code (`bridgeit/`, excluding the test suite itself). The same command runs in the CI pipeline on every push and pull request, together with static analysis (Ruff, Mypy), so no untested or badly-typed code reaches `master`. The generated HTML coverage report is uploaded as a CI build artifact, so results can be inspected for any commit.

## Automated Testing

### Test doubles

- **Mocks** (`unittest.mock.MagicMock`): used to stand in for the `google-genai` client inside `GeminiAIGateway`'s tests. These tests verify the adapter's own behavior — response parsing, retry logic on transient 429/503 errors, error translation — without making a real network call, since a mock is both free and deterministic, unlike the real Gemini API.
- **Fakes**: `InMemoryRequirementRepository` is a genuine, working implementation of the `RequirementRepository` port, backed by a plain Python dictionary instead of SQLite. It is used to verify the port's contract itself, independent of any specific storage technology, and to test application-layer use cases without needing a real database.

### Unit Testing

The suite counts **58 tests**, organized to mirror the production package tree.

**Domain Layer**

| Module | Tests | What it covers | Requirements |
|---|---|---|---|
| `test_requirement.py` | 13 | `Requirement` creation, lifecycle transitions (`submit`, `mark_analyzed`, `clarify`, `validate`, `reject`), invalid transitions, value-object equality (`RequirementText`, `RequirementStatus`) | FR-01, FR-03, FR-05 |
| `test_ai_analysis.py` | 4 | `AIAnalysis` and `QualityScore` value objects | FR-02, FR-04 |

**Application Layer**

| Module | Tests | What it covers | Requirements |
|---|---|---|---|
| `test_in_memory_requirement_repository.py` | 5 | Conformance of the fake repository to the `RequirementRepository` port's contract | NF-02 |
| `test_analyse_requirement.py` | 3 | `AnalyseRequirementUseCase`'s orchestration of the repository and the AI Gateway | FR-02 |
| `test_validate_requirement.py` | 7 | `ValidateRequirementUseCase`: approve/edit/reject decisions, the invariant that no analysis changes a requirement's state without a recorded human decision, invalid-state handling | FR-05 |

**Infrastructure Layer**

| Module | Tests | What it covers | Requirements |
|---|---|---|---|
| `test_sqlite_requirement_repository.py` | 5 | Save/retrieve round trip, missing-id lookup, status updates, persistence across separate repository instances pointing at the same file (confirming real disk persistence, not just in-memory state) | FR-01, NF-07 |
| `test_gemini_ai_gateway.py` | 12 | Successful response parsing, malformed/non-JSON responses, retry on transient 429/503 errors (with no retry on non-transient errors like 401/404), lazy API-key loading | FR-02, NF-07, NF-08 |

**Adapter (API) Layer**

| Module | Tests | What it covers | Requirements |
|---|---|---|---|
| `test_requirements_routes.py` | 3 | `POST /requirements`, `GET /requirements/{id}`, the structured 404 error format | FR-01 |
| `test_analysis_router.py` | 5 | `POST /requirements/{id}/analyse` and `POST /requirements/{id}/validate`, with a mocked AI Gateway injected directly | FR-02, FR-05 |
| `test_main_imports_without_key.py` | 1 | The application module imports and starts successfully even when `GEMINI_API_KEY` is not set — the key is only required when an analysis is actually requested, not at startup | NF-07 |

### Results

| Metric | Value |
|---|---|
| Total unit + integration tests | 58 |
| Passing | 58 |
| Failing | 0 |
| Success rate | 100% |
| Statement coverage (production code) | 95% |

All 58 tests pass. The HTML coverage report is uploaded as a build artifact by the CI workflow on every run, so the result can be inspected for any commit.

## Integration testing

Alongside the isolated unit tests above, several tests exercise real collaborating components through their actual interfaces, rather than through test doubles:

- **`SQLiteRequirementRepository` & a real SQLite file**: exercised directly, not through a fake, confirming the adapter's SQL and the real database engine behave as expected together, including across process/instance boundaries.
- **FastAPI routes & the real dependency-injected repository/AI Gateway wiring**: `TestClient`-based tests exercise the actual routing, middleware, and error-handling configuration of the running application object (`app`), not a simplified stand-in for it.
- **Application use cases & the domain**: `AnalyseRequirementUseCase` and `ValidateRequirementUseCase`'s tests exercise real `Requirement` domain objects (not mocked), verifying that the use case's calls into the domain correctly enforce its lifecycle rules — only the AI Gateway or repository (where not the subject of the test) are replaced by test doubles.

## System testing

BridgeIT does not have a fully automated, end-to-end test suite exercising the complete Requirement → AI Analysis → Human Validation flow as a single automated run; this was a deliberate scope trade-off given the project's timeline, with the full flow instead verified manually (see below). Automating it is listed as future work (see [Future work](../12-future/)).

What **is** verified automatically at system level, through the CI/CD workflow (`.github/workflows/check.yml`):

- the whole test suite runs across **3 operating systems** (Ubuntu, Windows, macOS) and **4 Python versions** (3.10, 3.11, 3.12, 3.13) — 12 runs in total, providing automated evidence that the codebase is not accidentally tied to one specific environment;
- the package is built with Poetry (`poetry build`) as part of every release, verifying the build itself succeeds, though, unlike some other course projects, it is not subsequently published to or reinstalled from PyPI, since BridgeIT is not intended for public package distribution (see [Release](../06-release/)).

The **Docker Compose** setup was used, separately, to manually verify the backend runs correctly in a clean environment with no locally pre-configured Poetry/Python setup, the same purpose a clean-runner PyPI round-trip would serve for a publicly distributed package.

## Manual Acceptance Tests

The acceptance tests below were executed by hand against the running application, following the plan below so that another team member can repeat them exactly. Each test corresponds to the acceptance criteria of the [Requirements](../02-requirements/) section.

| Requirement | Steps | Expected result |
|---|---|---|
| FR-01 | Submit a requirement via `POST /requirements` (or the "Create" frontend page) with non-empty text | A `201` response (or successful redirect) is returned with a unique id and status `Submitted`; retrieving it via `GET /requirements/{id}` returns the same text unchanged |
| FR-01 (error path) | Retrieve a requirement using an id that does not exist | A `404` response is returned, in the shared structured error format (`{"error": {"code": "requirement_not_found", ...}}`) |
| FR-02 | Request an analysis for a submitted requirement via `POST /requirements/{id}/analyse` (or the "Analyse" frontend page) | A real Gemini-generated `quality_indication` and a list of `issues` are returned; the requirement's status becomes `Analyzed`; its stored text is unchanged |
| FR-04 | Inspect the analysis result above | The `quality_indication` is either `ready_for_validation` or `needs_clarification`, distinct from the requirement's own status field |
| FR-05 | Submit a validation decision via `POST /requirements/{id}/validate` (or the "Validate" frontend page) with `decision: "approve"` | The requirement's status becomes `Validated`; repeating the same flow with `"reject"` on a different requirement results in status `Rejected` |
| FR-05 (invariant) | Attempt to retrieve a requirement's status before any validation decision has been recorded, after an analysis | The status remains `Analyzed`, not `Validated` — confirming no AI output changes the authoritative state without an explicit human decision |
| — (Docker) | Run `docker compose up --build` on a machine with no local Poetry/Python setup, then repeat the FR-01 → FR-02 → FR-05 flow above via Swagger UI at the container's exposed port | The full flow behaves identically to a local Poetry-based run |

All rows above were executed successfully, both via Swagger UI and via the real frontend, with no discrepancy in behavior between the two clients.
