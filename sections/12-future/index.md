---
title: Future work
has_children: false
nav_order: 13
---

# Known issues and future work

## What is missing

- **Traceability Link and Derived Artifact management (FR-06, FR-07)**: designed in the domain model (`Traceability Link`, `Derived Artifact`) but not implemented at the application or API layer. These were explicitly scoped as optional stretch goals from early in the project, prioritized below the core Requirement → AI Analysis → Human Validation cycle, and time did not allow reaching them within the one-month timeline.
- **Automated end-to-end / system tests**: the full workflow is verified manually (see [Validation](../05-validation/)), not by an automated test suite exercising the complete flow as a single run. Unit and integration coverage of each layer individually is thorough (95%), but no automated test currently catches a regression that only manifests across layer boundaries end-to-end.
- **AI analysis persistence**: an `AIAnalysis` result is returned to the caller but is not itself stored — only the requirement's resulting status and text are. Re-requesting an analysis for the same requirement calls Gemini again rather than returning a cached prior result.
- **Pre-commit / commit-msg hooks**: Conventional Commits discipline is followed manually, not enforced locally at commit time (see [Developer guide](../10-devguide/)).
- **User management / authentication**: never implemented. This was investigated during the project — an early internal note suggested it might be a course requirement, but no verifiable source for that confirmation was ever found, and the decision was made to not implement it rather than risk a rushed, insecure implementation crowding out the project's actual core focus. If the course requires it, this remains open work.

## What does not work as it should

- **The frontend has some code duplication** across its six pages (e.g. the navigation bar, the status "stamp" indicator) rather than being factored into reusable components, a direct, accepted consequence of using no frontend framework (see `architecture.md`'s ADR on this choice).
- **No client-side routing**: navigation between frontend pages triggers a full page reload rather than an in-place transition, since the frontend is plain HTML/CSS/JavaScript with no single-page-application framework.
- **No request-level hardening on the API**: there is currently no limit on the length of submitted requirement text, no rate limiting, and no authentication on any endpoint. For a locally-run academic prototype this has not caused a real problem, but it would need addressing before any public-facing deployment.
- **Gemini's free-tier quota is a real, occasionally visible constraint**: the AI Gateway retries transient 429/503 errors, but a sustained burst of requests beyond the daily free quota would still surface as a hard failure to the user, with no fallback provider or graceful degradation beyond that retry window.
- **The local `docs/` copy and the published report can drift apart** if not deliberately kept in sync — this happened at least once during the project (the local `docs/report.md`'s "Current Development Status" was found badly out of date and had to be corrected) and remains a structural risk of maintaining documentation in two places, even with the published site treated as authoritative.

## Potential future developments

- Implement FR-06 and FR-07 (traceability links and derived artifacts), completing the domain model's coverage end-to-end.
- Add an automated system/acceptance test suite, replacing the currently manual end-to-end verification.
- Add commit-msg linting (e.g. `commitlint` with a local hook) so Conventional Commits violations are caught before a commit is made, not only downstream in CI.
- Introduce a real, hosted deployment (e.g. on a platform such as Render or Railway) with HTTPS, building on the Docker image that already exists, together with the request-length limits, rate limiting, and basic authentication such a public deployment would require.
- Cache AI analysis results, both to reduce Gemini quota consumption and to make repeated inspection of an existing analysis instant rather than triggering a new API call.
- Revisit the frontend's technology choice if the number of pages or their interactivity grows substantially, the current vanilla HTML/CSS/JavaScript approach was evaluated as appropriate for the project's actual scope, not as a permanent constraint (see `architecture.md`).
- If a future course requirement confirms it is needed, implement user management and authentication as its own, properly-scoped work package, rather than retrofitted under time pressure.
