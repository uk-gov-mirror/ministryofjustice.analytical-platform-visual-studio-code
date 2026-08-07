---
description: "Perform monthly maintenance for this repository: update the cloud base image digest and Visual Studio Code package version, update tests, and complete the manual maintenance Definition of Done"
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

Use the current values from `Dockerfile` and `test/container-structure-test.yml`. Do not assume versions.

## Prerequisite

Reminder: this maintenance depends on the Analytical Platform Cloud Base Development Image maintenance issue being completed first.

- Dependency repository: `ministryofjustice/analytical-platform-cloud-development-environment-base`
- Treat this as a manual pre-check by the user running the ticket.

## Objective

In one pull request:

1. Update the pinned base image digest in `Dockerfile` for the existing image and tag.
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

2. Update the base image digest in `Dockerfile`.

- Read the current `FROM` image and tag from `Dockerfile`.
- Use the latest release tag from `ministryofjustice/analytical-platform-cloud-development-environment-base`.
- Pull for `linux/amd64` and inspect the digest.
- Update only the digest in the `FROM` line. Keep repository and tag unchanged.

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

5. Validate changes.

```bash
make test
```

6. Commit, push, and open a pull request.

- Use a Conventional Commit message with type `build`.
- Use a clear PR title, for example: `build: update cloud base image digest and Visual Studio Code version`.
- Include a concise PR summary of changed versions.

## Pull Request Checklist

Include this checklist in the PR description and complete each item:

### Definition of Done

Since this image relies on the Analytical Platform Cloud Base Development Image, ensure that maintenance issue is completed first.

- [ ] Merge any open Dependabot pull requests in the Visual Studio Code repository.
- [ ] Update the Visual Studio Code version if required.
- [ ] Create a new release in this repository.
- [ ] Update guidance if required: https://user-guidance.analytical-platform.service.justice.gov.uk/tools/visual-studio-code/#visual-studio-code
- [ ] Create a new release in development: https://controlpanel.services.dev.analytical-platform.service.justice.gov.uk/releases/
- [ ] Test deployment in development.
- [ ] Create a new release in production: https://controlpanel.services.analytical-platform.service.justice.gov.uk/releases/
- [ ] Test deployment in production.

## Guardrails

- Keep platform aligned to `linux/amd64` for digest checks.
- Do not change the base image repository or tag in `FROM`; update only digest.
- Do not change unrelated Dockerfile content.
- Do not update `test/container-structure-test.yml` unless Visual Studio Code version changed.
- Keep all maintenance updates in a single branch and a single pull request.
