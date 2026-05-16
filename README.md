# jules-automation

Two reusable GitHub Actions workflows that automate a FIFO [Jules](https://jules.google.com) coding workflow: one keeps exactly one open issue labeled `jules` at a time (oldest first), and one auto-merges pull requests once CI passes and closes linked issues.

## Usage

Add **two** workflow files to your repository.

### 1) Issue labeling

Create `.github/workflows/jules-issue-labeling.yml`:

```yaml
name: Jules issue labeling

on:
  issues:
    types: [opened, closed]

jobs:
  jules-issue-labeling:
    uses: sander-van-damme/jules-automation/.github/workflows/jules-issue-labeling.yml@main
    permissions:
      issues: write
```

### 2) Auto-merge

Create `.github/workflows/jules-auto-merge.yml`:

```yaml
name: Jules auto-merge

on:
  workflow_run:
    workflows: ["CI"] # Replace with your repository's CI workflow name
    types: [completed]

jobs:
  jules-auto-merge:
    uses: sander-van-damme/jules-automation/.github/workflows/jules-auto-merge.yml@main
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
