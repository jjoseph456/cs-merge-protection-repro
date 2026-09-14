# cs-merge-protection-repro

A minimal reproduction for investigating how code scanning merge protection
selects analyses when pull-request and branch results exist for different Git
references.

## Purpose

This repository isolates ref-selection behavior without including customer
data, support-case references, or internal issue identifiers.

## What This Reproduction Observes

Code-scanning merge protection evaluates analysis results associated with
different Git references. This repository provides a small, public-safe
environment for observing the relationship between:

- the CodeQL analysis for `main`
- the CodeQL analysis generated for a pull request
- the required code-scanning status shown on that pull request

It intentionally documents the observed references and merge status rather
than claiming a product defect or relying on private repository history.

## Setup

1. Fork this repository or create a branch from `main`.
2. Change a file under `.github/workflows/` so CodeQL analyzes the pull
   request.
3. Open a pull request targeting `main`.
4. Wait for the **CodeQL** workflow and the **Analyze GitHub Actions** check
   to complete.

The repository uses an advanced CodeQL workflow that analyzes GitHub Actions
files on both pull requests and pushes to `main`.

## Verification

Use the GitHub CLI to compare the pull-request and default-branch analyses:

```bash
OWNER=YOUR-OWNER
REPO=cs-merge-protection-repro
PR=YOUR-PR-NUMBER

gh pr view "$PR" --repo "$OWNER/$REPO" \
  --json mergeStateStatus,statusCheckRollup

gh api "repos/$OWNER/$REPO/code-scanning/analyses?ref=refs/pull/$PR/merge"
gh api "repos/$OWNER/$REPO/code-scanning/analyses?ref=refs/heads/main"
```

**Expected result:** after CodeQL completes, the pull request shows completed
CodeQL checks and the API returns a CodeQL analysis for its merge reference.
The default-branch query returns the baseline analysis for `main`.

If the observed result differs, capture the pull request number, both analysis
responses, and the completed check names before opening an issue. Do not add
customer data, private logs, or internal links to the report.

## Security

For a security issue, use GitHub's private vulnerability reporting feature.
