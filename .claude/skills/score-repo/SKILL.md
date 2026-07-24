---
name: score-repo
description: Audit an HCC UI repo against the 7-dimension governance rubric and produce a scored report
argument-hint: "[path to repo]"
---

# Score Repo Skill

You are auditing an HCC UI repository against the Repo Readiness Rubric.

## Instructions

1. Read the rubric definition from `rubric.md` in this repo (hcc-repo-readiness).

2. Check for a team standards file in `standards/`. If one exists (e.g., `standards/experience-ui.md`), read it. The standards define what the team considers correct — use them to evaluate whether a repo's guidance, patterns, and tooling align with team expectations, not just whether files exist. If the repo uses patterns that contradict the team standards (e.g., wrong component library, inline mocks when factories are required, useEffect for data fetching when TanStack Query is the standard), that's a scoring penalty on the relevant dimensions.

3. Determine the target repo to audit:
   - If `$ARGUMENTS` contains a path, use that path
   - If no arguments, use the current working directory
   - Verify the target is a valid repo (has package.json or similar)

3. For each of the 7 dimensions, investigate the target repo thoroughly:

   **1. Ownership**: Check for CODEOWNERS file. Read it. Are the owners real people? Is coverage intentional or just `* @someone`?

   **2. Agent Guidance**: Check for AGENTS.md or CLAUDE.md. Read it. Does it have the four keystone sections (overview, rules, docs index, file locations)? Are referenced docs real? Are rules actionable or generic? If the file was clearly auto-generated (e.g., by a sweep tool that analyzed the repo), evaluate whether the generated content is actually useful as guidance or just describes the current state without setting any rules.

   **3. Automated Enforcement**: Check eslint config. Are there project-specific rules beyond formatting? Check for custom rules directories. Grep for `eslint-disable` to gauge suppression density. Check if lint runs in the ACTIVE CI system (see CI rules below). **Cross-reference with dimension 2**: read the AGENTS.md/CLAUDE.md boundaries and rules, then check if each documented boundary has a corresponding lint rule enforcing it. If the guidance says "don't import X from Y" but there's no lint rule for it, flag the gap. Documented boundaries without enforcement are suggestions, not governance.

   **4. CI Trustworthiness**: Identify which CI system is ACTUALLY ACTIVE for this repo. HCC repos use Konflux (.konflux/ directory) as the primary CI. GitHub Actions (.github/workflows/) may also be active. Travis (.travis.yml), Jenkins (Jenkinsfile), and other legacy configs may exist in the repo but be INACTIVE — do not score against dead CI configs. Check package.json scripts to understand what the CI pipelines invoke. Evaluate whether the active CI runs lint, type checking, and tests through the repo's own npm scripts (e.g., `npm run lint`, `npm test`). If type checking is not part of the lint/test scripts, that's a gap in the repo's scripts, not a missing CI job.

   **5. Behavioral Verification**: Check for .storybook/ config. Find *.stories.tsx files. Do stories have play functions (interaction tests)? Do they exercise real user flows or just render? Does the active CI run storybook tests?

   **6. Data Layer Hygiene**: Check for mock directories, handler factories, seed data. Are mocks typed? Look at stories — do they use inline http.get/http.post handlers or import from factories? Is there any lint rule restricting inline mocks?

   **7. Structured Docs**: Check for docs directories (src/docs/, docs/). Read them — are they accurate and useful, or boilerplate? Does the AGENTS.md index point to them?

   **8. E2E Testing Maturity**: Check for e2e/ directory, playwright config, or cypress config. Check package.json for seed/cleanup/e2e scripts. Read test files — do they import from a seed map or use hardcoded UUIDs and entity names? Check for seed fixture files (JSON/TS). Check if multiple personas/permission levels are tested. Grep test files for hardcoded strings that look like entity names or UUIDs. Check if spec files have structured docblocks at the top (decision tree, capabilities, personas, data prerequisites) — read the first 30-40 lines of a few spec files to verify.

4. Score each dimension as PASS / PARTIAL / FAIL using the criteria in rubric.md. Be honest. Cite specific files and evidence.

5. Output the report in the format specified in rubric.md's Scoring Summary section.

## Rules

- **Be honest.** A generous score helps nobody. If something is borderline, lean toward the lower score and explain why.
- **Cite evidence.** Every score must reference specific files, line counts, or concrete observations.
- **Read, don't assume.** Don't score PASS on "CODEOWNERS exists" without reading it. Don't score Storybook without checking if stories have play functions.
- **Score quality, not existence.** An auto-generated AGENTS.md that describes a messy repo is a FAIL, not a PASS. A CI config file that exists but is not actively used is not CI.
- **Identify active vs dead infrastructure.** Repos accumulate stale configs (old Travis files, unused Jenkins pipelines). Only score against what actually runs. If unsure whether something is active, note the uncertainty.
- **Recommendations should match the repo's abstraction level.** Don't suggest raw compiler flags when the repo should have npm scripts. Don't suggest adding a CI job when the repo's scripts are the problem. Fix the root cause, not the symptom.
- **Do not modify the target repo.** This skill is read-only. No PRs, no file changes, no issues.
- **Be constructive in recommendations.** Specific, actionable, and encouraging. Point out what's already working well before describing what needs to change.

## Tone

Be direct and honest, but supportive — like a senior engineer doing a constructive code review, not a judge passing sentence. Acknowledge good work where it exists. When something scores FAIL, explain the gap clearly and what closing it would unlock, without editorializing ("governance theater", "proving jackshit", etc.). The goal is to make the reader want to fix things, not feel attacked. Facts and evidence are compelling enough on their own.
