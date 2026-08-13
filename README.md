# automation-workflows

Centralized GitHub Actions reusable workflows for Archibus partner PR automation.

## Available Workflows

### `partner-pr-automation.yml`

Automates partner release PR flow and CM ticket creation by webhook:
- Loads centralized JSON config from `automation-workflows`
- Supports centralized repo allow-list, branch allow-list, and blocked PR creators
- Skips automation when PR creator is in blocked user list
- Checks merge conflicts before attempting merge
- Merges PR and posts a rich payload to `JIRA_WEBHOOK_URL`

## Usage

Add this caller workflow to each target repository (for example `.github/workflows/pr-automation.yml`):

```yaml
name: PR Automation

on:
  pull_request:
    types: [opened, edited, synchronize]
    branches:
      - "*-staging"

jobs:
  call-central-workflow:
    uses: Archibus/automation-workflows/.github/workflows/partner-pr-automation.yml@master
    secrets:
      JIRA_WEBHOOK_URL: ${{ secrets.JIRA_WEBHOOK_URL }}
      CENTRAL_CONFIG_TOKEN: ${{ secrets.CENTRAL_CONFIG_TOKEN }}
```

`config_path` from older caller workflows is deprecated and ignored in centralized mode.

## Centralized Config File

Manage all repo onboarding in one file in `automation-workflows`:

- `config/partner-pr-automation-central-config.json`

Example:

```json
{
  "enabled_repositories": [
    "Archibus/isquared",
    "Archibus/ai-webcentral",
    "Archibus/cbre"
  ],
  "allowed_base_branch_patterns": [
    "*-staging"
  ],
  "default_blocked_pr_creators": [
    "dependabot[bot]",
    "some-user-to-ignore"
  ],
  "repository_overrides": {
    "Archibus/ai-webcentral": {
      "blocked_pr_creators": [
        "release-bot"
      ],
      "allowed_base_branch_patterns": [
        "*-staging"
      ]
    }
  }
}
```

Field notes:
- `enabled_repositories` (array): only listed repos run automation.
- `allowed_base_branch_patterns` (array): global base-branch patterns allowed to run automation (currently `*-staging`).
- `default_blocked_pr_creators` (array): applied to all enabled repos.
- `repository_overrides.<repo>.blocked_pr_creators` (array): per-repo override list.
- `repository_overrides.<repo>.allowed_base_branch_patterns` (array): optional per-repo branch pattern override.

To allow release later, add `"*-release"` to `allowed_base_branch_patterns`:

```json
"allowed_base_branch_patterns": ["*-staging", "*-release"]
```

The reusable workflow enforces these patterns, so automation is still skipped if a caller workflow trigger is broader.

## Required Secret

Caller repositories must include:

- `JIRA_WEBHOOK_URL` — Function App URL for `CreateJiraCM` (includes auth code)

Optional:

- `CENTRAL_CONFIG_TOKEN` — token with read access to `Archibus/automation-workflows`.
  - If omitted, workflow falls back to `${{ github.token }}`.

## Webhook Payload

The reusable workflow posts JSON including:
- `prLink`
- `prCreatedAt`
- `baseBranch`
- `mergedBranch` (PR source/head branch)
- `repository`, `prNumber`, `prAuthor`

## Updating the Template / Workflow

Since this workflow is centralized, any change made here applies to caller repos on their next PR event:

1. Edit `.github/workflows/partner-pr-automation.yml`
2. Commit and push to `master`
3. Caller repos pinned to `@master` pick it up automatically

## Branches / Versioning
For controlled rollouts, tag stable versions (e.g. `v1.0.0`) and pin caller repos to a tag:

```yaml
uses: Archibus/automation-workflows/.github/workflows/partner-pr-automation.yml@v1.0.0
```
