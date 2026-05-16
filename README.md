# jules-automation

A reusable GitHub Actions workflow that automates a FIFO [Jules](https://jules.google.com) coding workflow: it keeps exactly one open issue labeled `jules` at a time (oldest first), and it auto-merges pull requests once CI passes and closes linked issues.

## Usage

Add a single workflow file to your repository.

Create `.github/workflows/jules-automation.yml`:

```yaml
name: Jules automation

on:
  issues:
    types: [opened, closed]
  workflow_run:
    workflows: ["CI"] # Replace with your repository's CI workflow name
    types: [completed]

jobs:
  jules-automation:
    uses: sander-van-damme/jules-automation/.github/workflows/jules-automation.yml@main
    with:
      event: ${{ github.event_name }}
    permissions:
      contents: write
      issues: write
      pull-requests: write
```

Set `workflows` to the exact name of your repository's CI workflow.

## How it works

### 1) Assign next Jules issue (FIFO)

Triggered when an issue is opened or closed.

- Ensures the `jules` label exists.
- Does nothing if any open issue already has `jules`.
- Otherwise labels the oldest open issue with `jules`.

This keeps work in FIFO order: the oldest issue gets the label, and after it is closed the next oldest is labeled.

### 2) Auto-merge after CI success

Triggered when the configured CI workflow completes on a pull request.

- Skips runs that are not from a pull request or did not succeed.
- Finds the PR(s) associated with the completed run.
- Verifies each PR is still open, not a draft, and on the same commit SHA.
- Merges using the first enabled repository merge strategy.
- Closes issues referenced by PR closing keywords.
