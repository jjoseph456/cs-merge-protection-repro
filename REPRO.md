# Code scanning merge protection: ref-agnostic analysis selection

Reproduces code-scanning#23230 / #23399: merge protection evaluates a
**different-ref code scanning analysis than the PR check**, for the same commit OID.

## Setup
- Commit `42e5f4423f6cb1a463b47085b6ce818ebdb61d96` exists on two refs:
  `refs/heads/feature` (PR #1 head) and `refs/heads/stale`.
- Ruleset `require-codeql-high` on `main`: Require code scanning results, tool CodeQL,
  **Security alerts: High or higher**, **Alerts: None**, enforcement Active.
- Two CodeQL analyses uploaded for the same commit OID:
  - `refs/pull/1/head` (PR ref): **clean, 0 results**
  - `refs/heads/stale` (most recent): **1 finding**, rule `actions/missing-workflow-permissions`

## Result (PR #1, head = the commit above)

| Stale-ref finding (most recent) | PR-ref analysis | PR merge state |
|---|---|---|
| High / error (security-severity 8.0) | clean | **BLOCKED** |
| Medium / warning (security-severity 5.0) | clean | clean (not blocked) |

The PR's own analysis is clean and all check runs are green, yet the High-on-stale case
blocks the merge. Merge protection selected the stale-ref analysis (ref-agnostic /
most-recent) rather than the PR-ref analysis that the PR check used.

## Refinement vs the customer report
The threshold IS honored on whatever analysis is (wrongly) selected: a wrong-ref Medium
does not block under High-or-higher. So a "Medium blocks despite High-or-higher" symptom
means the **wrongly-selected analysis actually contained a >= threshold finding**. The
lead for the customer case: inspect which analysis merge protection loaded for the blocked
PR, not the all-Medium analysis the customer sees in the UI.
