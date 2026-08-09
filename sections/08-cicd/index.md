---
title: CI/CD
has_children: false
nav_order: 9
---

# CI/CD

This section describes the automated pipeline that takes a commit from a developer's machine all the way to a published GitHub Release, plus the automation that keeps dependencies up to date.

## Automated Actions

Here are summarised the automated actions in the CI/CD workflow. Actions 1–4 are implemented as two chained GitHub Actions workflows (`check.yml`, which calls `deploy.yml` as its final job), structured as `check → test → deploy`, each depending on the previous one succeeding, plus one independent, declarative bot configuration (`renovate.json`) that runs on its own schedule outside of this workflow, corresponding to Action 5.

| Action | Description | Reason |
|---|---|---|
| 1 | **Static analysis / code quality**: a syntax compile check, strict-mode Mypy type checking, Ruff linting, and Ruff format checking all run before anything else | Catching type errors, lint issues, formatting drift, and syntax problems before tests even run keeps feedback fast and cheap |
| 2 | **Automated testing**: the full test suite runs with coverage across a matrix of 3 operating systems (Ubuntu, Windows, macOS) × 4 Python versions (3.10–3.13), for 12 combinations in total | Gives real confidence the backend behaves the same for every team member and evaluator, not just on one developer's machine |
| 3 | **Automatic version bump**: `semantic-release` inspects the Conventional Commit messages since the last release, computes the next semantic version, updates `CHANGELOG.md` and `pyproject.toml`, and creates the corresponding git tag, with no manual version editing by a human | Removes an entire class of human error (forgetting to bump, bumping the wrong part) and ties the version number directly to the semantics of what actually changed |
| 4 | **Release to GitHub**: once a new version has been computed, the package is built (`poetry build`) and a GitHub Release is published with the build attached and auto-generated release notes | Gives anyone a versioned, downloadable artifact and a human-readable changelog for each release, without publishing to a public package index BridgeIT does not need (see [Release](../06-release/)) |
| 5 | **Dependency updates**: Renovate is configured to open pull requests for outdated npm/pip dependencies and GitHub Actions versions on a recurring schedule | Keeps the project from silently drifting onto outdated or vulnerable dependencies without a human needing to periodically remember to check |

## GitHub Actions Exploitation

### Triggers

The workflow (`.github/workflows/check.yml`) runs on:

- `push`, to any branch except `dependabot/**` and `renovate/**` (avoiding a redundant run on a bot's own branch, since the same change is also tested via its pull request), and ignoring pushes that only touch non-code files (`.gitignore`, `CHANGELOG.md`, `LICENSE`, `README.md`, `renovate.json`, etc.) via `paths-ignore`.
- `pull_request`, so proposed changes are validated before merge.
- `workflow_dispatch`, allowing a maintainer to trigger a run manually from the GitHub UI.

### Workflow jobs

Each job depends on the previous one succeeding.

- **`check` (Preliminary Checks)**: installs Poetry, restores the development environment, then runs `poe compile` (syntax check), `poe static-checks` (Ruff + Mypy), `poe format-check`, and `poe coverage` / `coverage-report` / `coverage-html` (tests with coverage), uploading the HTML coverage report as a build artifact.
- **`test`**: needs `check`; runs the test suite (`poe test`) across the matrix of `ubuntu-latest` / `windows-latest` / `macos-latest` and Python 3.10–3.13 (12 jobs in parallel), with `fail-fast: false` so one failing combination does not hide results from the others.
- **`deploy`**: needs `test`; declares `permissions: contents: write, packages: write`, and calls `.github/workflows/deploy.yml` as a reusable workflow (`uses: ./.github/workflows/deploy.yml`, `secrets: inherit`). This inner workflow runs `semantic-release` to compute the next version from Conventional Commits, update `CHANGELOG.md`, build the package, and publish the GitHub Release — or does nothing if the pushed commits do not warrant a release (e.g. a `docs:`-only commit).

**Two real defects were diagnosed and fixed in this deploy job during development**, not merely anticipated in the abstract:

1. `deploy.yml` originally referenced `secrets.RELEASE_TOKEN`, a custom secret name never actually configured in the repository — so the job failed with `No GitHub token specified` on every merge to `master`, silently, for several weeks, until the log was read closely and the reference corrected to `secrets.GITHUB_TOKEN` (GitHub's own automatically-provided token, already carrying the declared `contents: write` permission).
2. Once that was fixed, `semantic-release` began running for real and immediately failed again: it unconditionally tried to configure and publish to PyPI even with no `PYPI_TOKEN` secret set. `release.config.mjs` was changed to only attempt PyPI configuration/publishing if that token is actually present; otherwise it still builds the package, but skips the `poetry publish` step entirely.

Both fixes were confirmed by observing the next real merge complete with a fully green pipeline, not just by reasoning about the fix in isolation.

### Authentication: `GITHUB_TOKEN`, no long-lived secrets

Unlike a workflow that publishes to a public package index, BridgeIT's release step needs only to write back to its own repository (the changelog commit, the tag, and the GitHub Release itself). This is done entirely with **`secrets.GITHUB_TOKEN`**, the short-lived token GitHub automatically issues for every workflow run, scoped by the `permissions:` block declared at the job level (`contents: write`, `packages: write`). No manually created, long-lived token is stored in the repository's secrets for this purpose — only `PYPI_TOKEN` exists as an optional secret, and the release step is explicitly written to work correctly whether or not it is set (see above).

### Other permissions and environment variables

- `permissions: contents: write` lets `semantic-release` push the changelog commit and the new tag back to the repository.
- `permissions: packages: write` is requested for potential GitHub Packages usage.
- `RELEASE_DRY_RUN`, computed in `deploy.yml`, forces a dry run (no real release side effects) whenever the workflow is not running on `master`/`main`, so pull-request and feature-branch runs can safely exercise the same release logic without actually tagging or publishing anything.

## Dependency automation

Automated dependency updates are handled by **Renovate** (`.github/renovate.json`), configured to:

- check weekly, with a concurrent PR limit of 25 and no hourly limit;
- open pull requests separately for major, minor, and patch updates (`separateMajorMinor`, `separateMinorPatch`), rather than bundling them;
- assign opened PRs to the project maintainer (`assignees`), previously misconfigured to the professor's own GitHub username, inherited unnoticed from the course template, and corrected during development;
- tag GitHub Actions version updates specifically with the `ci` Conventional Commit type (`semanticCommitType`), keeping them distinguishable from dependency updates to the Python/Node packages themselves;
- expose a dependency dashboard issue (`dependencyDashboard`) summarizing all pending and rejected updates in one place.
