# Vue

- Rationale: for web apps using Vue. When a repo uses Vue, include framework-specific agent checks (build + component tests).
- Typical files: Vue single-file components, and `package.json` scripts.
- Agent checks: `pnpm run build` (Vue build), verify component rendering if publishing UI bundles.
- Notes: E2E tests for Vue should run in separate CI job and be optional for PRs (unless small and fast).

