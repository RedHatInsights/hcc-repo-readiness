# Experience UI Team Standards

Standards for the Console UI / Experience UI team's HCC frontend repos. The score-repo skill evaluates repos against these conventions when scoring dimensions 2 (Agent Guidance) and 7 (Structured Docs).

These are the team's engineering standards — what we expect in our repos regardless of what the starter app template ships with.

## Data Fetching

- **Use TanStack Query (React Query) for server state.** No `useEffect` + `useState` for API calls. No Redux for server state.
- Query hooks live in `data/queries/` directories, organized by API domain.
- **Query keys use the hierarchical factory pattern** — centralized key factories per domain (e.g., `rolesKeys.all`, `rolesKeys.list(params)`, `rolesKeys.detail(id)`). No inline query keys. Factories use `as const` for type inference and hierarchical spreading for invalidation. See `QueryKeysFactory.mdx` in rbac-ui for the reference implementation.

## UI Components

- **Use TableView from `@redhat-cloud-services/frontend-components` for tabular data.** Not PatternFly's Table directly, not `@patternfly/react-data-view`.
- TableView always requires `useTableState` for pagination, filtering, sorting, and selection state.
- For PatternFly component usage, import paths, and patterns, defer to the PatternFly MCP tooling — don't hardcode PF-specific implementation details in repo guidance as they change across versions.
- Components use named exports only. Features (route/page containers) may additionally have a default export.

## Testing

- **Cypress is deprecated.** Use Storybook for component testing and Playwright for E2E. Existing Cypress tests are legacy — do not count them as passing behavioral verification or E2E maturity. Do not write new Cypress tests.
- **Jest for non-React code.** Utility functions, pure logic, data transformations — anything that doesn't touch React gets a Jest unit test.
- **Storybook for component correctness.** Each component has a `.stories.tsx` file that tests its behavior through interaction tests (play functions) with MSW handlers. This is the first-class test artifact for React code.
- **User journey stories for integration.** Cross-component flows are tested through user journey stories that exercise the app as a user would — no context injection, no mocking internals, only MSW handlers at the network boundary.
- Play functions must use shared interaction helpers — no inlining modal waits, drawer scoping, tab switching, row selection.

## Mocking

- **MSW at the network boundary.** No mocking React components or hooks in tests.
- **Handler factories over inline handlers.** Stories must not define MSW request handlers (`http.get`, `http.post`) inline. Import handler factories from `data/mocks/` instead.
- Handler factories are typed against the same API types the real code uses. When API types change, mock type mismatches break the build.
- Seed data uses constants, not inline strings. Play functions reference seed constants, never hardcoded entity names.
- Stateful stories use resettable collections for test isolation.

## Data Layer

- API clients are implementation details. Only `data/api/` files may import from `@redhat-cloud-services/*-client` packages. Everyone else imports types through data layer re-exports.
- Data layer hooks get dependencies from dependency injection (e.g., `useAppServices()`), not direct imports of platform hooks.

## Architecture Boundaries

- Feature islands own their routes, components, data hooks, and mocks. Each feature island has a README explaining what it owns, what APIs it calls, and any non-obvious constraints.
- Cross-feature imports go through shared directories, never direct feature-to-feature.
- All boundaries documented in AGENTS.md must have a corresponding lint rule enforcing them. If it's in the guidance but not in eslint, it's a suggestion, not a boundary.

## CI Requirements

- CI pipeline structure is shared via Konflux. What runs inside the pipeline is determined by the repo's npm scripts.
- The repo's scripts must cover: lint (including type checking), unit tests, and Storybook interaction tests. If Konflux calls `npm run lint` and the script doesn't include type checking, that's a repo problem, not a CI problem.
- All checks must block merge on failure. No `continue-on-error` on quality gates.
- E2E tests (Playwright) are desirable but not gating on every PR due to cost. The Storybook layer compensates.
- Conventional commits (org standard).

## E2E Testing

- **Spec file docblocks are mandatory.** Every `.spec.ts` file starts with a structured docblock that describes: what the file tests, a decision tree (add here if / don't add here if), the capabilities and personas tested, and data prerequisites (auth, seed data, utilities). This is agent guidance at the test level — without it, agents (and humans) don't know where a new test belongs or what data setup it needs. See `e2e/_TEMPLATE.spec.ts` in rbac-ui for the canonical format.
- **Seed map pattern** — never hardcode UUIDs or entity names in test files. Seed data dynamically, export a name-to-UUID mapping, reference it in tests.
- **Prefix-based isolation** — every test run uses a unique prefix for its seeded data, enabling parallel CI runs without conflicts or shared state pollution.
- **Seed/test/cleanup lifecycle** — automated via npm scripts. No manual setup, no leftover data after runs.
- **Multi-persona testing matrix** — test the same flows under different permission levels (at minimum: admin and non-admin). Permission-denied paths are as important as happy paths.
- **No magic strings or numbers** — test code references seed constants and seed map entries. If a test has an inline UUID or entity name, it's wrong.
- **Progressive confidence** — build and verify in Storybook first (fast, deterministic), then graduate critical journeys to Playwright (full-stack against staging).

