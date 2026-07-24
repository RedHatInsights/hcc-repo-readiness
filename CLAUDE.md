# HCC Repo Readiness

This repo contains the Repo Readiness Rubric — a 7-dimension governance scorecard for HCC UI repositories.

## Skill: /score-repo

When invoked, this skill audits an HCC UI repo against the rubric and produces a scored report.

### Usage

Run this from the target repo's directory:

```
/score-repo
```

Or specify a path:

```
/score-repo /path/to/repo
```

### What it does

1. Reads the rubric from this repo's `rubric.md`
2. Analyzes the target repo's structure, config, docs, tests, and CI
3. Scores each of the 7 dimensions as PASS / PARTIAL / FAIL
4. Produces a report with evidence, tier recommendation, and top fixes

### What it does NOT do

- It does not modify the target repo
- It does not create PRs, issues, or files
- It only reads and reports
