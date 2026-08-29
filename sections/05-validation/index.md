---
title: Validation
has_children: false
nav_order: 6
---

# Validation

## Testing Approach

Tests were developed incrementally, alongside each piece of functionality, rather than all at once at the end of the project. In most cases, a new endpoint or use case was first verified manually (through Swagger UI or the frontend) to confirm its intended behavior, and then covered by an automated test reproducing that same check, a "verify, then automate" pattern rather than strict Test-Driven Development's red-green-refactor cycle.

The test suite follows a **mirrored structure**: each production module has a matching test module under `tests/`, replicating the package tree (`tests/domain/`, `tests/application/`, `tests/infrastructure/`, `tests/adapters/api/`). This keeps gaps visible, a production module without a corresponding test module has not been tested, and keeps each failing test pointing at exactly one file to investigate.

The **domain layer** (pure Python, no framework dependency) is the most densely tested part of the suite, since its business rules can be asserted directly and cheaply. **Adapters** (the SQLite repository, the Gemini AI Gateway) are tested against their **port interfaces**, so the tests describe expected behavior rather than internal implementation, a change in how an adapter is implemented internally does not break its tests without reason.

## Framework

The framework used is **pytest**. It is preferred over the standard library's `unittest` for two reasons: plain `assert` statements give rich failure introspection with no boilerplate assertion methods to remember; and its fixture system allows composable setup and easy injection of test doubles, used extensively for isolated, disposable SQLite databases (via the built-in `tmp_path` fixture) and for mocked external clients.

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

Manual acceptance testing complements the automated test suite by exercising the application as a user-facing system. The test plan below maps the main functional requirements to reproducible end-to-end checks.

### Acceptance test plan

| Requirement | Steps | Expected result |
|---|---|---|
| FR-01 | Submit a requirement via `POST /requirements` or the **Create** frontend page with non-empty text | A requirement is created with a unique id and status `Submitted`; retrieving it returns the same stored text |
| FR-01 (error path) | Retrieve a requirement using an id that does not exist | A `404` response is returned using the shared structured error format |
| FR-02 | Request an analysis for a submitted requirement via `POST /requirements/{id}/analyse` or the **Analyse** frontend page | Gemini returns a quality indication and issues; the requirement status becomes `Analyzed`; the authoritative requirement text is not autonomously modified by the AI |
| FR-04 | Inspect the AI analysis result | The analysis distinguishes between a requirement that is ready for validation and one that needs clarification |
| FR-05 — Approve | Submit a human validation decision with `decision: "approve"` | The requirement status becomes `Validated` |
| FR-05 — Edit | Submit a human validation decision with `decision: "edit"` and a revised requirement text | The revised text is persisted and the requirement status becomes `Clarified` |
| FR-05 — Reject | Submit a human validation decision with `decision: "reject"` | The requirement status becomes `Rejected` |
| FR-05 — Human-in-the-loop invariant | Inspect a requirement after AI analysis but before any human validation decision | The requirement remains `Analyzed`, confirming that AI output alone cannot produce `Validated`, `Clarified`, or `Rejected` |
| Deployment check | Run BridgeIT in its documented local or containerized environment and repeat the core submission → analysis → validation flow | The application starts correctly and the end-to-end workflow remains available through the exposed interfaces |

### Executed end-to-end acceptance session

A manual end-to-end acceptance session was performed on **29 August 2026** against the locally running BridgeIT application.

The FastAPI backend was running at `http://127.0.0.1:8000`, with the Gemini AI gateway enabled, and the frontend was served at `http://127.0.0.1:5500`.

The session focused on the core human-in-the-loop workflow and exercised all three FR-05 validation decisions through the real frontend.

| Test | Scenario | State after AI analysis | Human decision | Final persisted state | Result |
|---|---|---|---|---|---|
| TC-E2E-01 | Ambiguous requirement requiring clarification | `Analyzed` | `Edit` | `Clarified` | **PASS** |
| TC-E2E-02 | Clear requirement ready for approval | `Analyzed` | `Approve` | `Validated` | **PASS** |
| TC-E2E-03 | Problematic requirement rejected by the analyst | `Analyzed` | `Reject` | `Rejected` | **PASS** |

### Additional acceptance checks

After the three primary validation outcomes had been verified, the session was extended with additional checks covering requirement refinement, lifecycle enforcement, and the in-application Guide.

| Test | Scenario | Verified behavior | Result |
|---|---|---|---|
| TC-E2E-04 | Re-analyse a clarified requirement and then approve it | `Clarified` → `Analyzed` → `Validated` | **PASS** |
| TC-E2E-05 | Request another analysis after the requirement has reached `Validated` | The backend rejects the invalid transition with HTTP `409` | **PASS** |
| TC-E2E-06 | Repeat the edit and re-analysis refinement cycle twice | `Submitted` → `Analyzed` → `Clarified` → `Analyzed` → `Clarified` → `Analyzed` | **PASS** |
| TC-UI-01 | Open and inspect the in-application **Guide** page | The Guide loads successfully and exposes the expected workflow help and navigation | **PASS** |

### TC-E2E-01 — AI-assisted clarification

**Initial requirement**

> The system should notify users quickly when an important event occurs.

The requirement was created successfully and submitted for Gemini-assisted analysis.

The analysis identified three concrete quality problems:

1. the intended recipients represented by **"users"** were not specified;
2. **"quickly"** did not define a measurable response-time constraint;
3. **"important event"** did not specify which events should trigger a notification.

After AI analysis, the requirement status was `Analyzed`. The AI did not make an authoritative validation decision.

The Business Analyst selected `Edit` and replaced the text with:

> The system shall notify registered administrators within 5 seconds when a critical system failure is detected.

The resulting status was `Clarified`.

A subsequent lookup returned the edited text together with the `Clarified` state, confirming persistence of both the human decision and the revised requirement.

**Requirement id:** `3a1de6c5-2f2a-4ea0-8409-13e4da08cfe4`

**Result: PASS**

### TC-E2E-02 — Human approval of a clear requirement

**Initial requirement**

> The system shall send an email notification to registered administrators within 5 seconds after detecting a critical system failure.

The requirement was created and analysed successfully.

Gemini reported no blocking quality issues and indicated that the requirement was ready for Business Analyst review. Despite the positive AI assessment, the requirement remained `Analyzed` until an explicit human decision was recorded.

The Business Analyst selected `Approve`.

The resulting status was `Validated`.

A subsequent lookup returned the same requirement text and the `Validated` state, confirming persistence of the approval decision.

**Requirement id:** `8d8750e6-3071-4b31-8f61-bcce6e9b1615`

**Result: PASS**

### TC-E2E-03 — Human rejection of a problematic requirement

**Initial requirement**

> The system shall automatically delete all customer data every day.

The requirement was created and analysed successfully.

The AI identified several issues, including:

- insufficient definition of the customers and data affected;
- ambiguity in the execution schedule;
- missing safeguards concerning notification, archival, and permanent data loss.

After AI analysis, the requirement status was `Analyzed`.

The Business Analyst explicitly selected `Reject`.

The resulting status was `Rejected`.

A subsequent lookup returned the original requirement text and the `Rejected` state, confirming persistence of the rejection decision.

**Requirement id:** `d13eb837-183e-48e6-828f-7d9de024e6d4`

**Result: PASS**

### TC-E2E-04 — Re-analysis after clarification

TC-E2E-01 left the requirement in the `Clarified` state after the Business Analyst edited its text.

The clarified requirement was submitted for AI-assisted analysis again.

The re-analysis completed successfully and moved the requirement from:

`Clarified` → `Analyzed`

The requirement therefore returned to a state requiring an explicit Business Analyst decision rather than being autonomously finalized by the AI.

The Business Analyst then selected `Approve`, producing:

`Analyzed` → `Validated`

This verified the complete refinement path:

`Submitted` → `Analyzed` → `Clarified` → `Analyzed` → `Validated`

**Requirement id:** `3a1de6c5-2f2a-4ea0-8409-13e4da08cfe4`

**Result: PASS**

### TC-E2E-05 — Invalid transition after validation

After the requirement used in TC-E2E-04 had reached the final `Validated` state, another analysis request was attempted through:

`POST /requirements/{id}/analyse`

The backend rejected the request with HTTP status `409`.

The returned error reported that the transition from `Validated` to `Analyzed` was not permitted.

This confirms that the application enforces this lifecycle constraint and prevents a validated requirement from being moved back into the AI-analysis state through this operation.

**Requirement id:** `3a1de6c5-2f2a-4ea0-8409-13e4da08cfe4`

**Result: PASS**

### TC-E2E-06 — Repeated refinement cycle

A separate requirement was used to verify repeated clarification and re-analysis.

**Initial requirement**

> The system should notify users quickly about important events.

The requirement was created and analysed. The Business Analyst then edited it, producing the `Clarified` state.

The clarified requirement was analysed again, returning it to `Analyzed`.

A second human edit moved the requirement back to `Clarified`, followed by a second re-analysis.

The observed lifecycle was:

`Submitted` → `Analyzed` → `Clarified` → `Analyzed` → `Clarified` → `Analyzed`

The second re-analysis completed successfully, demonstrating that the `Edit` → `Clarified` → `Analyse` refinement cycle can be repeated before a final validation decision is recorded.

**Requirement id:** `df1dad9c-1496-497d-8b90-efbaf2f1914d`

**Result: PASS**

### TC-UI-01 — In-application Guide smoke test

The **Guide** page was opened through the BridgeIT frontend as a final user-interface smoke test.

The page loaded successfully and exposed the expected workflow guidance and application navigation.

This confirmed that the user-facing help page is reachable as part of the running application.

**Result: PASS**

### Acceptance result

All executed manual acceptance checks completed successfully.

The acceptance session verified the complete core workflow:

`Submitted` → `Analyzed` → explicit human decision

and all three FR-05 outcomes:

- `Edit` → `Clarified`;
- `Approve` → `Validated`;
- `Reject` → `Rejected`.

The extended checks additionally verified that:

- a `Clarified` requirement can be analysed again and return to `Analyzed`;
- the refinement cycle can be repeated before a final human decision;
- attempting to analyse a `Validated` requirement is rejected with HTTP `409`;
- the in-application **Guide** is available through the running frontend.

The tests provide direct evidence of BridgeIT's central human-in-the-loop invariant: **Gemini assists the requirements engineering process, but it does not autonomously determine the authoritative final state of a requirement. That decision remains under Business Analyst control.**
