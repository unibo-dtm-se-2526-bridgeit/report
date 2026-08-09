---
title: Release
has_children: false
nav_order: 7
---

# Release

The BridgeIT-artifact codebase produces one release artefact:

- Python package (source distribution and wheel), containing the `bridgeit` backend package.

The artefact is generated through the Poetry build system using the following command:

```bash
poetry build
```

The generated files are stored inside the `dist/` directory.

The artefact is released on:

- GitHub Releases, which allows users to download a specific version of the project together with the release notes.

BridgeIT is not published to PyPI or TestPyPI: as an academic project, it is not intended for public package distribution. The release pipeline is configured to skip PyPI publishing entirely when no `PYPI_TOKEN` secret is configured — this was a deliberate engineering decision, not an oversight, made after diagnosing an earlier CI failure caused by the release step unconditionally attempting to publish without one.

In addition, the project's `CHANGELOG.md` file is updated to document the changes introduced in each release.

The release process is automated through a CI/CD pipeline; such workflow will be further explained in the [CI/CD](../08-cicd/) section.

## Choice of the license

Both the source code and the released artefact are distributed under the **Apache License 2.0**. This license was inherited from the official Poetry project template provided for the course, and was deliberately retained rather than replaced: parts of the project's tooling and CI/CD configuration are still derived from that template, so keeping the same license is consistent with properly crediting that inherited work alongside BridgeIT's own original code. The copyright notice has been updated to reflect the project's actual authors.

## Choice of the versioning schema

BridgeIT adopts Semantic Versioning (SemVer), using the format:

```text
MAJOR.MINOR.PATCH
```

Semantic Versioning clearly communicates the nature of the changes introduced in each release:

- `MAJOR` version: incremented when incompatible or breaking changes are introduced.
- `MINOR` version: incremented when new functionality is added in a backward-compatible manner.
- `PATCH` version: incremented when bugs are fixed or minor improvements are made without introducing new features.

Since only one artefact is generated from the codebase, there is a single, unambiguous version number.

### Creating a New Release

As stated earlier, the CI/CD workflow is in charge of: updating the application version and the changelog, releasing the new version and creating the tag of the new version. An update of the application is released everytime a commit is pushed or a branch is merged to the branch `master` and the changes committed are so that an upgrade of the version is necessary. The last condition is also automated by the application of the Conventional Commit specification, as described in [Development](../04-development/) section; in this way, the level of the version change depends on the type of the commit. The version is upgraded accordingly to the highest level of change brought by the commits pushed:

- a `fix` commit leads to a `PATCH` level update;
- a `feat` commit leads to a `MINOR` level update;
- a `BREAKING CHANGE` commit leads to a `MAJOR` level update.

Any other type of commit will not update the version at any level.

This automation, powered by `semantic-release`, was not functional for the first part of the project: a misconfigured GitHub token caused the release job to fail on every merge to `master`, silently, for several weeks before being diagnosed and fixed. The pipeline has been fully operational, including automatic release creation, since that fix.
