# Unjs

- Rationale: for projects using Unjs ecosystem tools. When a repo uses Unjs packages, include framework-specific agent checks (build + compatibility tests).
- Typical files: Various Unjs package configurations, and `package.json` scripts.
- Agent checks: `pnpm run build` (Unjs build), verify package compatibility and exports.
- Notes: E2E tests for Unjs projects should run in separate CI job and be optional for PRs (unless small and fast).

