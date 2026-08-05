# BridgeIT — Software Engineering Project Report

> **Status:** Work in progress

BridgeIT is an AI-assisted Requirements Engineering platform developed for the **Software Engineering** course of the **Digital Transformation Management** degree programme at the University of Bologna.

The project supports the Requirement lifecycle from submission to AI-assisted quality analysis and explicit Business Analyst validation. AI suggestions assist the process, but the final decision remains human-controlled.

## Project links

- [Published report](https://unibo-dtm-se-2526-bridgeit.github.io/report/)
- [Software artifact](https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact)
- [Report repository](https://github.com/unibo-dtm-se-2526-bridgeit/report)
- [GitHub organization](https://github.com/unibo-dtm-se-2526-bridgeit)

## Team

- **Nicole Tresca** — `@nikytresca`
- **Martina Fava** — `@marthinaf03`

## Project overview

BridgeIT provides a structured workflow for:

1. submitting a Requirement;
2. storing and retrieving Requirements;
3. requesting an AI-assisted quality analysis;
4. reviewing issues detected by the AI provider;
5. validating, clarifying, editing, or rejecting a Requirement through an explicit Business Analyst decision.

The AI assessment is intentionally qualitative and binary:

- `ready_for_validation`;
- `needs_clarification`.

The system does not expose a fabricated numerical confidence score.

## Architecture and technologies

BridgeIT adopts **Hexagonal Architecture** and **Domain-Driven Design** principles within the **Requirement Lifecycle** bounded context.

The implementation currently uses:

- Python, FastAPI, and Pydantic;
- SQLite through Python's standard-library `sqlite3` driver;
- Google Gemini through the `google-genai` client;
- vanilla HTML, CSS, and JavaScript;
- Poetry, pytest, Ruff, and Mypy;
- GitHub for version control, collaboration, and publication.

Architectural details are documented in the [Design chapter](sections/03-design/index.md), while implementation choices and development conventions are described in the [Development chapter](sections/04-development/index.md).

## Repository organization

The project is split into two repositories:

- **`BridgeIT-artifact`** contains the application source code, frontend, backend, automated tests, and implementation-related configuration.
- **`report`** contains this Jekyll-based Software Engineering report and its Markdown chapters.

The report chapters are stored in [`sections/`](sections/).

## Report contents

The report follows the structure supplied by the course template and includes chapters covering:

- concept and Requirements;
- architectural and domain design;
- development practices;
- validation and testing;
- release, deployment, and CI/CD;
- user and developer guidance;
- self-evaluation and future work.

Some chapters are still being completed because the project is a work in progress.

## Local preview

A local preview requires Ruby, Bundler, and the dependencies declared in the repository.

From the repository root:

```bash
bundle install
bundle exec jekyll serve
```

The terminal normally exposes the preview at:

```text
http://127.0.0.1:4000
```

Stop the local server with `Control + C`.

## Contribution workflow

Before editing the report:

```bash
git switch main
git pull origin main
```

After changing a report file:

```bash
git status
git diff
git add <modified-files>
git commit -m "docs: describe the completed change"
git pull --rebase origin main
git push origin main
```

Commit messages follow the Conventional Commits style.

## Publication

The report is published through GitHub Pages from the `main` branch. After a successful push, the published website may take a short time to update.

The publication status can be checked from the repository's **Actions** or **Deployments** area.

## Academic context

BridgeIT is an academic project developed for the University of Bologna Software Engineering course. This repository documents the implemented system, its architectural rationale, the development process, and the validation evidence collected by the team.
