# jules-automation

Reusable GitHub Actions workflow that automates a FIFO [Jules](https://jules.google.com) coding workflow: it auto-merges pull requests once all checks pass, closes linked issues, and keeps exactly one open issue labeled `jules` at a time (oldest first).

## Usage

Add the following workflow file to your repository at `.github/workflows/jules-automation.yml`:

```yaml
name: Jules automation

on:
  workflow_run:
    workflows: ["CI"] # Replace with the workflow name(s) that run your checks
    types: [completed]
  pull_request:
    types: [opened, synchronize, reopened]
  issues:
    types: [opened, closed]
  workflow_dispatch:

concurrency:
  group: jules-automation
  cancel-in-progress: false

jobs:
  jules-automation:
    uses: sander-van-damme/jules-automation/.github/workflows/jules-automation.yml@main
    permissions:
      contents: write
      issues: write
      pull-requests: write
      checks: read
      statuses: read
```

That's it. You can adjust the triggers to fit your setup — the workflow handles each event type independently.
`workflow_run` requires an explicit `workflows` list, so set it to your repository's CI workflow name(s).
The reusable workflow itself uses a different concurrency group (`jules-automation-reusable`) to avoid deadlocks with your top-level workflow.

## How it works

### 1) Auto-merge after successful checks

Triggered by `workflow_run` (completed runs of the workflows you list) and `pull_request` events.

For runs created by a pull request event with a successful conclusion, the workflow:

- finds the PR(s) for the run commit,
- verifies the PR is still open, not draft, and still on the same SHA,
- verifies commit statuses/check-runs are fully green,
- merges using the first enabled repository merge strategy,
- closes issues referenced by PR closing keywords.

### 2) Assign next Jules issue (FIFO)

Triggered by `issues: opened|closed` and `workflow_dispatch` (manual run).

The workflow:

- ensures the `jules` label exists,
- does nothing if any open issue already has `jules`,
- otherwise labels the oldest open issue with `jules`.

This keeps work in FIFO order: oldest issue gets the label, then after closure the next oldest gets labeled.
