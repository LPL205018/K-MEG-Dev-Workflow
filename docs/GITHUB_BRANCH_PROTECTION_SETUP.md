# GitHub Branch Protection Setup

This checklist configures `main` to enforce the K-MEG workflow gates.

## 1. Open Branch Protection

1. Go to `Settings` -> `Branches`.
2. Click `Add branch ruleset` (or `Add rule`).
3. Target branch: `main`.

## 2. Pull Request Controls

Enable:

- `Require a pull request before merging`
- `Require approvals` = `1`
- `Dismiss stale pull request approvals when new commits are pushed`
- `Require review from Code Owners`

## 3. Status Checks

Enable:

- `Require status checks to pass before merging`
- `Require branches to be up to date before merging`

Add required checks:

- `Gatekeeper - PM SA SD QA / G1-PM`
- `Gatekeeper - PM SA SD QA / G2-SA`
- `Gatekeeper - PM SA SD QA / G3-SD`
- `Gatekeeper - PM SA SD QA / G4-QA`
- `CI Pipeline - Multi-Stack (Strict Mode) / CI Result Summary`

Note:

- These check names are defined in `Rules of Dev/Workflow-Automation-Wiring-v1.md`.
- A check can only be selected after the corresponding workflow has run at least once.

## 4. Optional Hardening

Enable:

- `Block force pushes`
- `Block deletions`
- `Restrict who can push to matching branches` (small maintainer set)

## 5. Verification

1. Open a test PR to `main`.
2. Confirm reviewer is auto-assigned by `.github/CODEOWNERS`.
3. Confirm required checks appear and must pass before merge.
4. Confirm direct push to `main` is blocked.

