---
description: "Perform monthly maintenance for this repository: update to the latest cloud base image version (tag and digest) and Visual Studio Code package version, update tests, and complete the manual maintenance Definition of Done"
tools:
  [
    "search/codebase",
    "search",
    "edit/editFiles",
    "execute/runInTerminal",
    "execute/getTerminalOutput",
  ]
---

# Maintenance

Perform monthly maintenance for this repository. This image depends on the Analytical Platform Cloud Base Development Image and pins a Visual Studio Code Debian package version. Update both where needed, update the container structure test expectation, and prepare one pull request.

Note: Ubuntu is inherited from the cloud base image, so base image maintenance here means updating this repository to the latest available cloud base image release.

Use the current values from `Dockerfile` and `test/container-structure-test.yml`. Do not assume versions.

## Prerequisite

Reminder: this maintenance depends on the Analytical Platform Cloud Base Development Image maintenance issue being completed first.

- Dependency repository: `ministryofjustice/analytical-platform-cloud-development-environment-base`
- Treat this as a manual pre-check by the user running the ticket.

## Objective

In one pull request:

1. Update the pinned base image in `Dockerfile` to the latest available cloud base image release (tag and digest).
2. Update `VISUAL_STUDIO_CODE_VERSION` in `Dockerfile` if a newer Debian package version is available.
3. Update the `code --version` expected output in `test/container-structure-test.yml` when Visual Studio Code is updated.
4. Keep everything else unchanged.

## Required Outcome

1. Create one maintenance branch.
2. Commit all required file updates with a Conventional Commit (`build:`).
3. Push and open one pull request.
4. Include a checklist aligned to the manual maintenance ticket Definition of Done.

## Execution Steps

1. Create a maintenance branch.

```bash
git checkout -b "chore/maintenance-vscode-$(date +%Y%m%d-%H%M%S)"
```

2. Update the base image version in `Dockerfile`.

- Read the current `FROM` image reference from `Dockerfile`.
- Get the latest release tag from `ministryofjustice/analytical-platform-cloud-development-environment-base`.
- Pull that latest tag for `linux/amd64` and inspect the digest.
- Update the `FROM` line to use the latest tag and matching digest.
- Keep the base image repository unchanged.

Reference commands from the README:

```bash
docker pull --platform linux/amd64 ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:$(curl --silent https://api.github.com/repos/ministryofjustice/analytical-platform-cloud-development-environment-base/releases/latest | jq -r .tag_name)

docker image inspect --format='{{index .RepoDigests 0}}' ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:$(curl --silent https://api.github.com/repos/ministryofjustice/analytical-platform-cloud-development-environment-base/releases/latest | jq -r .tag_name)
```

3. Check whether `VISUAL_STUDIO_CODE_VERSION` should be updated.

- Get the latest Visual Studio Code release tag from `microsoft/vscode`.
- Query the Microsoft Debian package feed and locate the amd64 package entry matching that release.
- Extract the full Debian version string (example format: `1.86.2-1707854558`).
- Compare with `VISUAL_STUDIO_CODE_VERSION` in `Dockerfile`.
- If newer, update `VISUAL_STUDIO_CODE_VERSION`.

Reference command from the README:

```bash
curl --silent https://packages.microsoft.com/repos/code/pool/main/c/code/ | grep $(curl --silent https://api.github.com/repos/microsoft/vscode/releases/latest | jq -r .tag_name) | grep amd64
```

4. Update test expectations when Visual Studio Code changes.

- In `test/container-structure-test.yml`, update the `commandTests` entry for `code --version`.
- The expected output should contain only the semantic version part (for example `1.110.1`), not the Debian build suffix.

5. Commit, push, and open a pull request.

- Use a Conventional Commit message with type `build`.
- Use a clear PR title, for example: `build: update cloud base image version and Visual Studio Code version`.
- Create the PR description using the structure below.

Use this PR description template (replace placeholders with actual values):

```markdown
## Summary
Updates pinned Visual Studio Code dependencies in `Dockerfile` and aligns the version assertion in `test/container-structure-test.yml`.

### Base image
- `ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:<tag>` digest is already up to date.
- No base image change in this PR.

### Visual Studio Code
| Item | Before | After |
| --- | --- | --- |
| `VISUAL_STUDIO_CODE_VERSION` | `<old-deb-version>` | `<new-deb-version>` |
| `code --version` expected output | `<old-semver>` | `<new-semver>` |
```

When base image does change in the PR, replace the `### Base image` section with:

```markdown
### Base image
- `ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:<old-tag>` -> `ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:<new-tag>`
- Digest: `sha256:<old-digest>` -> `sha256:<new-digest>`
```

## Guardrails

- Update to the latest available release of `ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base`.
- Update both base image tag and digest together in `FROM`.
- Keep platform aligned to `linux/amd64` for digest checks.
- Keep the base image repository unchanged.
- Do not change unrelated Dockerfile content.
- Do not update `test/container-structure-test.yml` unless Visual Studio Code version changed.
- Deliver all updates in the same branch and pull request.
- Use [Conventional Commits](https://www.conventionalcommits.org/) for both the commit message and the PR title (use the `build` type).

