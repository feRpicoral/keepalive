# keepalive

Reusable GitHub Actions workflow that prevents scheduled workflows from being disabled after 60 days of repository inactivity.

GitHub disables a public repository's scheduled (`cron`) workflows once there have been no commits for 60 days. This workflow checks how long it has been since the last commit and, only when that exceeds a threshold, lands one empty commit on the default branch to reset the clock.

It pushes the commit through a pull request and merges it with the built-in `GITHUB_TOKEN`, so it works on repositories whose default branch requires pull requests. It needs no personal access token, no GitHub App, and no ruleset bypass.

## Usage

Add a caller workflow to each repository:

```yaml
name: keepalive

on:
  schedule:
    - cron: "0 3 * * 1"
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  keepalive:
    uses: feRpicoral/keepalive/.github/workflows/keepalive.yml@v1
    with:
      stale-days: 50
```

Run the caller at least once a week so a commit lands before the 60-day limit. A manual `workflow_dispatch` run when the repository is active is a safe no-op.

Each consuming repository must also allow Actions to open the pull request (Settings → Actions → General → "Allow GitHub Actions to create and approve pull requests"), or via the CLI:

```sh
gh api --method PUT repos/OWNER/REPO/actions/permissions/workflow \
  -f default_workflow_permissions=read \
  -F can_approve_pull_request_reviews=true
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `stale-days` | `50` | Only act when the latest commit is older than this many days. |
| `commit-message` | `chore: keepalive` | Message for the empty commit. |
| `pr-title` | `chore: keepalive` | Title of the pull request. |
| `pr-body` | _(see workflow)_ | Body of the pull request. |
| `branch-prefix` | `keepalive` | Prefix for the temporary branch (`<prefix>/<run-id>`). |
