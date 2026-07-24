# Repo Readiness Rubric

Universal scorecard for evaluating HCC UI repository governance maturity. Designed to determine what review process a repo can safely operate under.

## Scoring Principles

1. **The presence of a file is not a passing score.** Each dimension is scored on whether it actively constrains the quality of work, not whether an artifact exists.
2. **Scoring requires in-depth, honest analysis.** Read the files. Evaluate their quality. An AGENTS.md that accurately describes a repo with no boundaries is not agent guidance — it's a map of a mess.
3. **Less human oversight requires more infrastructure, not less.** A repo that wants lighter review (T3) needs a higher governance score than one operating under full human review (T1).

## Dimensions

Each dimension is scored as: PASS / PARTIAL / FAIL

### 1. Ownership

**Question**: Is it clear who is responsible for this repo, and was that a deliberate decision?

**PASS**: CODEOWNERS file exists with named individuals (not just team aliases). Ownership covers the meaningful areas of the codebase. Assignment was intentional.
**PARTIAL**: CODEOWNERS exists with named individuals but is stale (people who left, coverage gaps) or lacks path-specific granularity.
**FAIL**: No CODEOWNERS, or CODEOWNERS is a single catch-all team alias (`* @team`). A team alias that nobody monitors is the same as no ownership.

**Where to look**: `.github/CODEOWNERS`, `CODEOWNERS`

### 2. Agent Guidance

**Question**: Can an AI agent orient itself in this repo and know what rules to follow without guessing?

**PASS**: AGENTS.md (or CLAUDE.md with equivalent content) exists with four keystone sections:
  - Project overview (what the app is, its architecture, key boundaries)
  - Non-negotiable rules (things that must never be violated)
  - Docs index pointing to retrievable files (not everything inline)
  - Key file locations (the tree, so the agent orients without running `find`)
  Content is accurate, rules are actionable, referenced docs exist, and guidance reflects the team's actual standards — not just the current state of the code. An auto-generated file that describes how the repo currently works without encoding how it SHOULD work is not guidance. It teaches agents to reproduce the status quo, including its mistakes.
**PARTIAL**: File exists with the right structure and has been human-reviewed, but has gaps — some sections are thin, some referenced docs are missing, or rules exist but don't cover all critical boundaries.
**FAIL**: No agent guidance. Or: file exists but was auto-generated without human review against team standards. Or: file encodes patterns the team doesn't endorse (e.g., recommending a component library the team has moved away from). Guidance that steers agents toward wrong patterns is worse than no guidance — it's actively harmful.

**Where to look**: `AGENTS.md`, `CLAUDE.md`, `.claude/`

### 3. Automated Enforcement

**Question**: Can a developer or agent violate a rule that matters without CI catching it?

**PASS**: ESLint (or equivalent) is configured with project-specific rules that enforce meaningful boundaries — not just formatting. Import restrictions, pattern enforcement, or domain-specific constraints exist and match the repo's risk profile. Rules are active in CI (not just in editor config). Principle: if a constraint is worth documenting in AGENTS.md as a warning, it's worth enforcing in lint. Documented rules without enforcement are suggestions; enforced rules are governance.
**PARTIAL**: Project-specific rules exist but don't cover all boundaries documented in AGENTS.md. Or: rules exist but some are warn-only instead of error.
**FAIL**: No project-specific enforcement rules. Standard shared lint config only (formatting, generic TS rules) with no repo-specific boundaries. If the AGENTS.md documents constraints that aren't enforced in lint, this dimension is FAIL — documented rules without enforcement are suggestions, not governance.

**Where to look**: `.eslintrc.*`, `eslint.config.*`, `eslint-rules/`, CI config (`.github/workflows/`, `Jenkinsfile`, `.konflux/`), look for `eslint-disable` density

### 4. CI Trustworthiness

**Question**: Can you see a green checkmark and trust it without pulling the branch locally?

**PASS**: CI runs build (which covers type checking via `fec build`), lint, and tests. None are skipped or set to `continue-on-error`. Failures block merge. Whether the *right* things are being linted and tested is scored in other dimensions (3, 5, 6) — this dimension scores whether CI infrastructure is trustworthy as a gate.
**PARTIAL**: CI runs but some checks are allowed to fail without blocking, or key steps (build, lint, or tests) are missing from the pipeline.
**FAIL**: No CI, or CI is a formality that doesn't block merge on failure.

**Where to look**: `.github/workflows/`, `Jenkinsfile`, `.konflux/`, CI status checks configuration in repo settings

### 5. Behavioral Verification

**Question**: Does CI test real user-facing behavior, or just unit logic?

**PASS**: Storybook (or equivalent) exists with interaction tests that exercise user flows at the mock boundary. Tests run in CI and block merge. Coverage is meaningful — not just rendering checks but actual user interactions (click, fill, navigate, verify state changes).
**PARTIAL**: Storybook interaction tests exist, run in CI, but coverage is thin (less than half of components have play functions) or limited to simple rendering checks.
**FAIL**: No interaction tests in CI. This includes: stories exist but have no play functions, or play functions exist but don't run in any CI pipeline. Tests that never run in CI are not verification — they're documentation that might be wrong.

**Where to look**: `.storybook/`, `**/*.stories.tsx`, `package.json` (test scripts), CI config for storybook test runs

### 6. Data Layer Hygiene

**Question**: Are mocks governed, or can agents generate throwaway mocks that make tests pass without proving anything?

**PASS**: Mock data is typed against real API contracts. Handler factories exist and are the required way to set up mocks (inline handlers are restricted or banned). Seed data uses constants, not inline strings. When API types change, mock type mismatches break the build.
**PARTIAL**: Some mock structure exists (MSW is set up, some handlers are factored out) but inline mocks are common and not restricted. Mock types may drift from real API types without detection.
**FAIL**: No mock governance. Each test/story creates its own mocks ad-hoc. Or: no mocks at all (tests hit real APIs or skip network testing entirely).

**Where to look**: `**/mocks/`, `**/handlers.*`, `**/*.stories.tsx` (check for inline `http.get`/`http.post`), MSW configuration, eslint rules restricting mock imports

### 7. Structured Docs

**Question**: Can a human or agent learn what this app does, how it's built, and where to make changes safely — without reading every file?

**PASS**: Documentation exists covering the app's scope, architecture, and key patterns. The AGENTS.md docs index points to these files. Docs are accurate, maintained, and reflect the team's actual standards.
**PARTIAL**: Docs exist, are human-reviewed, and cover the basics, but have gaps — missing sections, no docs index in AGENTS.md, or incomplete coverage of the app's architecture.
**FAIL**: No meaningful docs beyond a boilerplate README. Or: docs exist but encode wrong patterns (wrong libraries, outdated architecture). Or: docs were auto-generated and never reviewed. Docs that teach agents wrong patterns are worse than no docs.

**Where to look**: `src/docs/`, `docs/`, `README.md`, `ARCHITECTURE.md`, whatever AGENTS.md docs index references

### 8. E2E Testing Maturity

**Question**: Can the repo prove its features work against a real environment, and can those tests run reliably in CI without flaking or polluting shared state?

**PASS**: E2E tests exist with:
  - Structured docblocks at the top of each spec file (decision tree, personas, data prerequisites)
  - Seed fixtures for dynamic data creation (no hardcoded UUIDs or entity names in test files)
  - Prefix-based isolation so parallel CI runs don't conflict
  - Automated seed/test/cleanup lifecycle via npm scripts
  - Multi-persona testing matrix (at minimum: admin + non-admin permission levels)
  - Test files reference seed map entries or constants, never inline magic strings
**PARTIAL**: E2E tests exist but have structural problems: hardcoded IDs/strings, no cleanup scripts, no isolation mechanism, single persona only, or tests exist but aren't runnable in CI.
**FAIL**: No E2E tests, or only smoke tests that don't exercise real user journeys.

**Where to look**: `e2e/`, `playwright.config.*`, `cypress/`, `package.json` (seed/cleanup/e2e scripts), `e2e/fixtures/`, test files for hardcoded strings vs seed map imports

## Scoring Summary

After evaluating all 8 dimensions, produce a scorecard:

```
REPO READINESS REPORT: [repo-name]
Date: [date]

| # | Dimension                | Score   | Evidence |
|---|--------------------------|---------|----------|
| 1 | Ownership                | [P/F/~] | [brief]  |
| 2 | Agent Guidance           | [P/F/~] | [brief]  |
| 3 | Automated Enforcement    | [P/F/~] | [brief]  |
| 4 | CI Trustworthiness       | [P/F/~] | [brief]  |
| 5 | Behavioral Verification  | [P/F/~] | [brief]  |
| 6 | Data Layer Hygiene       | [P/F/~] | [brief]  |
| 7 | Structured Docs          | [P/F/~] | [brief]  |
| 8 | E2E Testing Maturity     | [P/F/~] | [brief]  |

Score: [X/8] (PASS counts as 1, PARTIAL as 0.5, FAIL as 0)

RECOMMENDED REVIEW TIER: [T1/T2/T3]
Based on: [risk profile] repo with [X/8] governance score

TOP 3 FIXES (highest impact first):
1. [dimension] — [what to do and why it matters]
2. [dimension] — [what to do and why it matters]
3. [dimension] — [what to do and why it matters]
```

## Tier Recommendation Logic

- Score < 3: Must operate as T1 regardless of risk profile
- Score 3-5: May operate as T2 if risk profile allows
- Score 6+: May operate at its natural risk-based tier (T2 or T3)
- Score 8: Full governance. Repo can safely operate at T3 if risk profile warrants it

These thresholds are starting points. Adjust based on experience.
