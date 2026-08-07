# Analytical Platform Visual Studio Code

[![Ministry of Justice Repository Compliance Badge](https://github-community.service.justice.gov.uk/repository-standards/api/analytical-platform-visual-studio-code/badge)](https://github-community.service.justice.gov.uk/repository-standards/analytical-platform-visual-studio-code)

[![Open in Dev Container](https://raw.githubusercontent.com/ministryofjustice/.devcontainer/refs/heads/main/contrib/badge.svg)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/ministryofjustice/analytical-platform-visual-studio-code)

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/ministryofjustice/analytical-platform-visual-studio-code)

This repository contains the Visual Studio Code container image for use on the Analytical Platform.

## Running Locally

### Build

```bash
make build
```

### Test

```bash
make test
```

### Run

```bash
make run
```

Open a browser <http://localhost:8080/?folder=/home/analyticalplatform/workspace>

## Managing Software Versions

This repository includes a Copilot prompt for maintenance in [`.github/prompts/`](.github/prompts/). To run it, open Copilot Chat in Visual Studio Code and type `/maintenance`. The prompt updates the Ubuntu base image digest and the pinned APT package versions in the [Dockerfile](./Dockerfile) and prepares a single pull request.

### Base Image

Generally Dependabot does this, but the following command will return the digest:

```bash
docker pull --platform linux/amd64 ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:$(curl --silent https://api.github.com/repos/ministryofjustice/analytical-platform-cloud-development-environment-base/releases/latest | jq -r .tag_name)

docker image inspect --format='{{index .RepoDigests 0}}' ghcr.io/ministryofjustice/analytical-platform-cloud-development-environment-base:$(curl --silent https://api.github.com/repos/ministryofjustice/analytical-platform-cloud-development-environment-base/releases/latest | jq -r .tag_name)
```

### Visual Studio Code

Releases for Visual Studio Code are published on [GitHub](https://github.com/microsoft/vscode/releases), but we specifically want the Debian package version, to obtain this you can run the following:

```bash
curl --silent https://packages.microsoft.com/repos/code/pool/main/c/code/ | grep $(curl --silent https://api.github.com/repos/microsoft/vscode/releases/latest | jq -r .tag_name) | grep amd64
```

This will return a string like this for version `1.86.2`:

```bash
<a href="code_1.86.2-1707854558_amd64.deb">code_1.86.2-1707854558_amd64.deb</a> ...
```

From that, we want `1.86.2-1707854558`.

## Maintenance

Maintenance of this component is scheduled in this [workflow](https://github.com/ministryofjustice/analytical-platform/blob/main/.github/workflows/schedule-issue-vscode.yml), which generates a maintenance ticket as per this [example](https://github.com/ministryofjustice/analytical-platform/issues/5904).
