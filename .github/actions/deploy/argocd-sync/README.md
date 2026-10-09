# ArgoCD Sync Action

This composite action updates an ApplicationSet revision in an Argo CD
configuration repository, then logs in to Argo CD and syncs the configured
applications.
## Usage

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Sync Argo CD applications
        uses: pandeyaws/shared-templates/.github/actions/deploy/argocd-sync@master
        with:
          app_id: ${{ vars.APP_ID }}
          private_key: ${{ secrets.APP_PRIVATE_KEY }}
          target_revision: ${{ needs.version.outputs.tag }}
          argocd_server: ${{ vars.ARGOCD_SERVER }}
          argocd_username: admin
          argocd_password: ${{ secrets.ARGOCD_PASSWORD }}
          parent_app: apps-sample-python-app
          child_app: sample-python-app-dev
          cert_manager_app: cert-manager
          external-secrets: external-secrets
```

Replace `@master` with a release tag or commit SHA when you want to pin the
action to a stable version.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `app_id` | Yes | — | GitHub App ID used to create a token for the Argo CD config repository. |
| `private_key` | Yes | — | GitHub App private key. Pass this as a GitHub secret. |
| `target_revision` | Yes | — | Revision or image tag to write to the ApplicationSet. |
| `repo_owner` | No | `pandeyaws` | Owner of the Argo CD config repository. |
| `repo_name` | No | `sample-python-app-argocd` | Argo CD config repository name. |
| `branch` | No | `master` | Branch to update in the config repository. |
| `appset_path` | No | `argocd/sample-python-app-appset.yaml` | ApplicationSet manifest path, relative to the config repository root. |
| `environment_name` | No | `dev` | Environment entry whose `targetRevision` is updated. |
| `argocd_server` | Yes | — | Argo CD server hostname and port, for example `argocd.example.com:443`. |
| `argocd_username` | Yes | `admin` | Argo CD login username. |
| `argocd_password` | Yes | — | Argo CD login password. Pass this as a GitHub secret. |
| `parent_app` | Yes | `apps-sample-python-app` | Parent application to sync and wait for health/sync. |
| `child_app` | Yes | `sample-python-app-dev` | Child application to wait for health/sync. |
| `cert_manager_app` | Yes | `cert-manager` | Application to sync after waiting for the child application. |
| `external-secrets` | Yes | `external-secrets` | Application to sync after the cert-manager application. |

Inputs marked required currently have defaults for some values. Supply them
explicitly in the workflow for clarity and to avoid relying on metadata defaults.

## Requirements and behavior

- Run on a Linux GitHub-hosted or self-hosted runner with `bash`, `curl`,
  `sudo`, and Python available. The action downloads and installs the latest
  Argo CD CLI release.
- The GitHub App must be installed on the config repository with permission to
  write contents. The action creates a token and pushes the manifest change to
  the configured branch.
- The Argo CD credentials must be able to log in and sync/wait on all configured
  applications.
- The action updates the configured environment's `targetRevision` before
  logging in to Argo CD. It expects exactly one matching environment entry in
  the manifest.
- Sync order: sync and wait for the parent, wait for the child, then sync the
  cert-manager and external-secrets applications.
