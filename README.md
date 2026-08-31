# BridgeIT — Software Engineering Project Report

![Status](https://img.shields.io/badge/status-finalization-brightgreen)
![Report](https://img.shields.io/badge/report-GitHub%20Pages-blue)
![Format](https://img.shields.io/badge/format-Markdown-lightgrey)

> AI-Supported Requirements Engineering Platform — University of Bologna Software Engineering Project (A.Y. 2025/2026)

BridgeIT is a Requirements Engineering platform supporting the lifecycle of natural-language software requirements from submission to AI-assisted quality analysis and explicit Business Analyst validation.

The project follows a **human-in-the-loop** principle: AI assists the Requirements Engineering process, while authoritative validation decisions remain under human control.

## Project Links

- [Published report](https://unibo-dtm-se-2526-bridgeit.github.io/report/)
- [Software artifact](https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact)
- [Report repository](https://github.com/unibo-dtm-se-2526-bridgeit/report)
- [GitHub organization](https://github.com/unibo-dtm-se-2526-bridgeit)

## Current Project Status

The central BridgeIT workflow has been implemented and validated end-to-end in the software artifact:

`Submitted` → `Analyzed` → explicit Business Analyst decision

with the supported human outcomes:

- `Approve` → `Validated`
- `Edit` → `Clarified`
- `Reject` → `Rejected`

A requirement in `Clarified` state can be analysed again and return to `Analyzed`. The `Edit` → `Clarified` → `Analyse` refinement cycle can therefore be repeated before a final validation decision is recorded.

The project currently includes:

- Hexagonal Architecture and Domain-Driven Design;
- FastAPI and Pydantic backend;
- SQLite persistence through Python's standard `sqlite3`;
- Google Gemini integration behind an `AIGateway`;
- vanilla HTML/CSS/JavaScript frontend;
- automated testing and static verification with pytest, Ruff, and Mypy;
- Docker and Docker Compose support;
- GitHub Actions CI/CD and automated releases;
- a complete User Guide;
- recorded manual end-to-end acceptance evidence.

The core implementation, User Guide, and Validation documentation are complete. Final editorial alignment, self-evaluation, and AI-tool-use disclosure are being finalized before submission.

## Repository Organization

BridgeIT is split into two repositories:

- **`BridgeIT-artifact`** — application source code, frontend, backend, automated tests, Docker configuration, and CI/CD-related files;
- **`report`** — this Jekyll-based Software Engineering report and its Markdown chapters.

## Report Structure

The report follows the structure provided by the Software Engineering course and includes:

1. **Concept** — project motivation, context, and vision;
2. **Requirements** — functional and non-functional requirements;
3. **Design** — architecture, domain model, ports, adapters, and design rationale;
4. **Development** — implementation and technological choices;
5. **Validation** — automated and manual testing evidence;
6. **Release** — licensing, versioning, and release process;
7. **Deployment** — local and containerized execution;
8. **CI/CD** — GitHub Actions and automation;
9. **User Guide** — end-user workflow across the BridgeIT frontend;
10. **Developer Guide** — development environment and project commands;
11. **Self-evaluation** — final project assessment;
12. **Future Work** — limitations and possible extensions.

## Validation

The manual acceptance session exercises the implemented human-in-the-loop workflow and records:

- Requirement creation;
- AI-assisted analysis;
- human clarification and editing;
- explicit human approval;
- explicit human rejection;
- re-analysis after clarification;
- repeated refinement cycles;
- invalid-transition enforcement after validation;
- frontend Guide availability.

Detailed test steps, Requirement identifiers, lifecycle transitions, and results are documented in the **Validation** chapter.

## Architecture and Technologies

BridgeIT adopts **Hexagonal Architecture (Ports and Adapters)** and **Domain-Driven Design**.

The implemented technology stack includes:

- Python;
- FastAPI;
- Pydantic;
- SQLite through Python's standard `sqlite3`;
- Google Gemini through `google-genai`;
- vanilla HTML, CSS, and JavaScript;
- Poetry;
- pytest;
- Ruff;
- Mypy;
- Docker and Docker Compose;
- GitHub Actions.

The separation between core logic and technical adapters keeps persistence and AI-provider choices outside the domain model.

## Current Scope

The final implementation deliberately prioritizes the complete:

**Requirement → AI Analysis → Human Validation**

workflow together with the engineering infrastructure required to implement, test, deploy, and document it reliably.

The following capabilities remain outside the implemented core scope:

- authentication and user management;
- authorization policies;
- persistence and caching of AI-analysis results;
- richer traceability-link management;
- derived artifact generation.

These remain possible future extensions of the current design.

## Local Preview

The report is built with Jekyll. From the repository root:

```bash
bundle install
bundle exec jekyll serve
```

The local preview is normally available at:

```text
http://127.0.0.1:4000
```

## Publication

The report is published through GitHub Pages from the `main` branch:

https://unibo-dtm-se-2526-bridgeit.github.io/report/

## Academic Context

BridgeIT was developed for the **Software Engineering** course of the **Digital Transformation Management** degree programme at the University of Bologna.

The report documents both the implemented system and the engineering process used to design, develop, validate, release, and deploy it.
