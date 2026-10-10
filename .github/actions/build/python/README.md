# Python Build Action

This composite action checks out the calling repository, sets up Python,
optionally runs tests, builds and pushes a Docker image to Azure Container
Registry (ACR), and optionally scans the image with Trivy.

## Usage from another repository

Add a workflow in the consuming repository, for example
`.github/workflows/build.yml`:

```yaml
name: Build and push Python image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - name: Build and push image
        uses: pandeyaws/shared-templates/.github/actions/build/python@<ref>
        with:
          python-version: "3.12"
          enable-tests: "true"
          docker_username: ${{ secrets.DOCKER_USERNAME }}
          docker_password: ${{ secrets.DOCKER_PASSWORD }}
          acr_name: ${{ vars.ACR_NAME }}
          acr_login_server: ${{ vars.ACR_LOGIN_SERVER }}
          image_name: sample-python-app
          image_tag: ${{ github.sha }}
          scan_image: "true"
```

Replace `<ref>` with a release tag or commit SHA to pin the action version.
Configure `ACR_NAME` and `ACR_LOGIN_SERVER` as repository or organization
variables, and store credentials as GitHub secrets.

The action checks out the consuming repository and expects its `requirements.txt`,
Dockerfile, and application source at the repository root.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `python-version` | No | `3.12` | Python version to install. |
| `enable-tests` | No | `false` | Set to `"true"` to run tests and upload test artifacts. |
| `docker_username` | Yes | — | Docker registry username. |
| `docker_password` | Yes | — | Docker registry password or token. |
| `acr_name` | Yes | — | Azure Container Registry name used by `az acr login`. |
| `acr_login_server` | Yes | — | ACR login server, for example `myregistry.azurecr.io`. |
| `image_name` | Yes | — | Image repository/name. |
| `image_tag` | Yes | — | Main image tag. The Git commit SHA is also added as a tag. |
| `scan_image` | No | `true` | Set to `"false"` to skip the Trivy vulnerability scan. |

## Requirements and behavior

- The consuming workflow must grant `id-token: write` permission for Azure
  workload identity federation.
- Configure the Azure federated credential to trust the consuming repository
  and workflow.
- The workflow runner must support PowerShell (`pwsh`) and have Azure CLI
  available for `az acr login`.
- Test dependency installation currently runs even when `enable-tests` is
  `"false"`; the test and artifact upload steps are conditional.
- The action currently references `inputs.AZURE_CLIENT_ID`,
  `inputs.AZURE_TENANT_ID`, and `inputs.AZURE_SUBSCRIPTION_ID` in its Azure
  login step, but these inputs are not declared in `action.yml`. Declare and
  pass those values before using the action; as written, Azure login will not
  receive the required IDs.
- `docker_username` and `docker_password` are currently required inputs but
  are not used by the action's ACR login flow.
