# HCC Repo Readiness

A governance scorecard for HCC UI repositories. Scores repos on 7 dimensions to determine what review process they can safely operate under.

## The Problem

Not every repo needs the same review process because not every repo carries the same risk. But lighter review is only safe when the repo has enough governance infrastructure to catch problems automatically. **Less human oversight requires more infrastructure, not less.**

A sweep that adds AGENTS.md to every repo doesn't make repos AI-ready. It makes them _look_ AI-ready. This tool tells the difference.

## The 8 Dimensions

| # | Dimension | What it asks |
|---|-----------|-------------|
| 1 | **Ownership** | Is it clear who's responsible, and was that deliberate? |
| 2 | **Agent Guidance** | Can an AI agent orient itself and know the rules? |
| 3 | **Automated Enforcement** | Can someone violate a rule that matters without CI catching it? |
| 4 | **CI Trustworthiness** | Does a green checkmark actually mean something? |
| 5 | **Behavioral Verification** | Does CI test real user behavior, or just unit logic? |
| 6 | **Data Layer Hygiene** | Are mocks governed, or can agents fake their way to green? |
| 7 | **Structured Docs** | Can someone learn what this app does without reading every file? |
| 8 | **E2E Testing Maturity** | Can the repo prove features work against a real environment without flaking? |

Each dimension is scored as **PASS** / **PARTIAL** / **FAIL** based on an honest, in-depth analysis — not a file-existence check.

## How It Works

This repo includes a Claude Code skill that audits a target repo against the rubric.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed

### Usage

From the repo you want to score:

```bash
claude /score-repo
```

The skill reads the target repo, evaluates each dimension with evidence, and produces a report with:
- A scored breakdown per dimension
- A recommended review tier (T1/T2/T3)
- The top 3 fixes for maximum impact

It does not modify the target repo. Score only.

## The Tier System

The rubric feeds into a tier-based review process:

- **T1 (Critical Infrastructure)**: Full review, CODEOWNER approval, manual testing. The 2-reviewer rule makes total sense here.
- **T2 (Important, Not Critical)**: Single CODEOWNER approval, human review for significant changes, bots OK for bulk work.
- **T3 (Bulk/Low-Risk)**: One approval + green CI. Cursory reviews. Bots can work mostly unsupervised.

**Tier = f(risk profile, governance score)**

A repo's risk profile is what it does (publishes packages? handles auth? simple settings UI?). Its governance score is how many dimensions it meets. A repo that's T3 by risk but scores poorly must operate at a higher tier until the gaps are closed.

## Team Standards

The `standards/` directory contains team-specific conventions. The skill evaluates repos against these standards when scoring.

- `standards/experience-ui.md` — Console UI / Experience UI team

Other teams can add their own standards file. Same rubric, different conventions.

## Future

- **Remote scoring** — point at a GitHub repo URL instead of requiring a local checkout
- **Standards selection** — specify which standards file to evaluate against (e.g., `/score-repo --standards experience-ui https://github.com/org/repo`)

## Full Rubric

See [rubric.md](rubric.md) for the complete scoring criteria, evidence guidelines, and tier recommendation logic.

## Background

- [Engineering an AI-ready code base: Governance lessons from the Red Hat Hybrid Cloud Console](https://developers.redhat.com/articles/2026/04/15/governance-lessons-red-hat-hybrid-cloud-console)
- [How we turned Storybook into a behavioral verification engine](https://developers.redhat.com/articles/2026/04/29/how-we-turned-storybook-behavioral-verification-engine)
- [Vercel: AGENTS.md outperforms skills in our agent evals](https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals)
