---
title: Developer guide
has_children: false
nav_order: 11
---

# Developer Guide

If anybody wants to contribute to BridgeIT, the best way is to open an [Issue on GitHub](https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact/issues) or a Pull Request directly. There is no dedicated contact email for the project at this stage; all discussion happens on GitHub, within the [`unibo-dtm-se-2526-bridgeit`](https://github.com/unibo-dtm-se-2526-bridgeit) organization.

## Internal Conventions

### Naming

Files and modules are written in `snake_case` (e.g. `sqlite_requirement_repository.py`), while classes use `PascalCase` (e.g. `Requirement`, `SQLiteRequirementRepository`, `GeminiAIGateway`). Unlike some conventions, port interfaces are **not** prefixed with `I` — `RequirementRepository` and `AIGateway`, the two abstract base classes defining the application layer's ports, carry the same plain, descriptive naming as any other class, consistent with typical Python style rather than a C#/Java-style interface prefix. Enumerations combine a `PascalCase` type name with `UPPER_CASE` members (e.g. `RequirementStatus.SUBMITTED`, `RequirementStatus.VALIDATED`, `QualityScore.NEEDS_CLARIFICATION`).

### Code Style

Style and typing are enforced by **Ruff** (linting and formatting) and a strict **Mypy** configuration, applied equally to both production code and the test suite (`mypy bridgeit tests`), untyped definitions are not relaxed for `tests.*`, unlike some other conventions.

Unlike some other course projects, BridgeIT does **not** currently have pre-commit or commit-msg hooks installed (no `commitlint`, no `pre-commit` / `husky` setup): Conventional Commits discipline is followed manually by the team and only checked downstream, by `semantic-release`, when a commit reaches `master`, not blocked locally at commit time. This is a known gap, listed as a possible future improvement rather than solved, since introducing it would require additional npm tooling not otherwise needed for local development (see [Future work](../12-future/)).

### Versioning and Development Workflow

The team follows the **Conventional Commits** specification. Please refer to the [Release](../06-release/) and [CI/CD](../08-cicd/) sections for how commit types drive automatic versioning and releases.

## Development Environment Setup

The project is managed with **Poetry**, which creates a project-local `.venv/`. Starting from a fresh clone:

```bash
git clone https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact.git
cd BridgeIT-artifact
poetry install     # Python dependencies → .venv/
```

No `npm install` step is needed for local development: the project's `package.json` only declares `semantic-release`-related tooling, used exclusively by the CI/CD release job (see [CI/CD](../08-cicd/)), not by anything a contributor runs locally.

**Editor setup**: contributors using VS Code are encouraged to install the Ruff extension and enable "format on save" for Python files, so formatting issues are caught at save time instead of surfacing later as a CI failure. This is a personal editor preference, not committed to the repository, `.vscode/settings.json` is intentionally gitignored, so each contributor's local preferences don't leak into the shared repository configuration.

## Running tests and checks

All commands are defined as `poe` (poethepoet) tasks, so they behave identically on a local machine and in the CI pipeline:

```bash
poe test              # full test suite (pytest)
poe coverage           # tests with coverage measurement
poe coverage-report    # coverage report, missing lines shown in the terminal
poe coverage-html      # HTML coverage report
poe static-checks      # ruff check + mypy together
poe format-check        # verify formatting without changing files
poe format               # auto-fix formatting
poe compile             # syntax-only compile check
```

Quality checks run through `poe`, so any editor or IDE can be connected to the same commands, no editor-specific configuration is needed to reproduce the CI results locally.
