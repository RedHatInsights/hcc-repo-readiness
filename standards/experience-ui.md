# Experience UI Team Standards

> **Source of truth**: [experience-ui-governance/standards/](https://github.com/RedHatInsights/experience-ui-governance/tree/main/standards)
>
> This file is a pointer, not a copy. The full standards (architecture, data layer, testing, mocking, code style, CI, E2E) live in the governance repo and are consumed by team repos via `npm install github:RedHatInsights/experience-ui-governance`.

The score-repo skill evaluates repos against these conventions when scoring dimensions 2 (Agent Guidance) and 7 (Structured Docs).

## Standards index

| Topic | Governance file |
|-------|----------------|
| Feature islands, ServiceContext DI, navigation | [architecture.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/architecture.md) |
| TanStack Query, API clients, query keys, DI contract | [data-layer.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/data-layer.md) |
| Storybook-first testing, Jest, Playwright | [testing.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/testing.md) |
| MSW handler factories, seed data, type safety | [mocking.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/mocking.md) |
| Exports, imports, naming, commits | [code-style.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/code-style.md) |
| CI pipeline, quality gates | [ci.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/ci.md) |
| Playwright specs, seed maps, isolation, personas | [e2e.md](https://github.com/RedHatInsights/experience-ui-governance/blob/main/standards/e2e.md) |

## Enforcement

The governance repo also ships implementable enforcement:

- **ESLint plugin**: `experience-ui-governance/eslint-plugin` — 3 shared rules (`enforce-story-patterns`, `require-use-table-state`, `no-direct-user-type`)
- **CodeRabbit template**: `experience-ui-governance/templates/coderabbit.yml` — PR hygiene, export rules, separation of concerns
- **Reusable CI workflows**: Storybook build+test, Chromatic upload
