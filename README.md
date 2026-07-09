# automation-workflows

Centralized GitHub Actions reusable workflows for Archibus partner PR automation.

## Available Workflows

### `partner-pr-automation.yml`

Automates the partner release PR process:
- Detects merge conflicts and stops if found
- Validates required PR body fields (release summary, changed apps, submitter info)
- Creates a CM ticket via the `CreateJiraCM` function app
- Sets due date (today + SLA days)
- Writes CM ↔ email mapping to Azure Storage Table
- Posts CM ticket URL as a PR comment

**Only runs when:** source branch starts with `release/` or `upgrade/`.

## Usage

Add the following workflow to each partner repo at `.github/workflows/pr-automation.yml`:

```yaml
name: PR Automation

on:
  pull_request:
    types: [opened, edited, synchronize]
    branches:
      - staging
      - release

jobs:
  call-central-workflow:
    uses: Archibus/automation-workflows/.github/workflows/partner-pr-automation.yml@main
    secrets:
      JIRA_WEBHOOK_URL: ${{ secrets.JIRA_WEBHOOK_URL }}
```

## Required Secrets

Each partner repo (or the org) must have the following secrets configured:

- `JIRA_WEBHOOK_URL` — Function App URL for `CreateJiraCM` (includes auth code)

## PR Template

Partners must include the following fields in the PR description:

```markdown
## Release Information

**Release Summary:**
<Brief description of what's changing>

**Changed Web Central Applications:**
<List of apps being updated>

**Submitter Name:**

**Submitter Email:**

**Submitter Phone:**
```

If any required field is missing, the workflow posts a comment with the correct template.

## Updating the Template / Workflow

Since this workflow is centralized, any change made here automatically applies to all partner repos on the next PR event. To update:

1. Edit `.github/workflows/partner-pr-automation.yml`
2. Commit and push to `main`
3. All partner repos will use the updated version on their next PR

## Branches / Versioning

Partner repos pin to `@main` by default. For controlled rollouts, tag stable versions (e.g. `v1.0.0`) and have partner repos pin to a specific tag:

```yaml
uses: Archibus/automation-workflows/.github/workflows/partner-pr-automation.yml@v1.0.0
```
