# jules-automation

Drop-in GitHub Actions automation for a FIFO Jules workflow:

- auto-merge pull requests after checks pass,
- close linked issues,
- keep exactly one oldest open issue labeled `jules`.

## Installation

Copy this file into any repository:

- `.github/workflows/jules-automation.yml`

## How it works

### 1) Auto-merge after successful checks

Triggered by `workflow_run` (all completed workflows).

For runs created by a pull request event with a successful conclusion, the workflow:

- finds the PR(s) for the run commit,
- verifies the PR is still open, not draft, and still on the same SHA,
- verifies commit statuses/check-runs are fully green,
- merges using the first enabled repository merge strategy,
- closes issues referenced by PR closing keywords.

### 2) Assign next Jules issue (FIFO)

Triggered by:

- `issues` with type `closed`
- `workflow_dispatch` (manual run)

The workflow:

- ensures the `jules` label exists,
- does nothing if any open issue already has `jules`,
- otherwise labels the oldest open issue with `jules`.

This keeps work in FIFO order: oldest issue gets the label, then after closure the next oldest gets labeled.
