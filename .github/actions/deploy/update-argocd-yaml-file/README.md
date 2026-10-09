# Update ArgoCD YAML Action

This composite action updates the `targetRevision` for one environment in an
ApplicationSet manifest stored in a separate GitHub repository, then commits
and pushes the change.

## Usage

```yaml
steps:
  - name: Update the Argo CD ApplicationSet
    uses: pandeyaws/shared-templates/.github/actions/deploy/update-argocd-yaml-file@master
    with:
      app_id: ${{ vars.APP_ID }}
      private_key: ${{ secrets.APP_PRIVATE_KEY }}
      target_revision: ${{ needs.version.outputs.tag }}
      repo_owner: pandeyaws
      repo_name: sample-python-app-argocd
      branch: master
      appset_path: argocd/sample-python-app-appset.yaml
      environment_name: dev
```

Replace `@master` with a release tag or commit SHA when you want to pin the
action to a stable version.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `app_id` | Yes | — | GitHub App ID used to create an installation token. |
| `private_key` | Yes | — | GitHub App private key. Pass this as a GitHub secret. |
| `target_revision` | Yes | — | Non-empty, single-line revision or image tag to write. |
| `repo_owner` | Yes | `pandeyaws` | Owner of the Argo CD config repository. |
| `repo_name` | Yes | `sample-python-app-argocd` | Argo CD config repository name. |
| `branch` | No | `master` | Branch to check out and push the change to. |
| `appset_path` | No | `argocd/sample-python-app-appset.yaml` | Manifest path relative to the config repository root. |
| `environment_name` | No | `dev` | Environment entry whose `targetRevision` is updated. |

## Requirements and behavior

- Run on a Linux runner with `bash` and Python available.
- The GitHub App must be installed on the target repository with contents-write
  permission. The action uses `actions/create-github-app-token` and
  `actions/checkout` to access the repository.
- The manifest must contain exactly one matching `environment` entry with its
  `targetRevision` on the following line. The action fails if it finds zero or
  multiple matches.
- If the manifest already has the requested revision, the action exits without
  creating a commit. Otherwise it commits the change and pushes to the
  configured branch.
