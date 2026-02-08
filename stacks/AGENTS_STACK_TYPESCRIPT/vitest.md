# Vitest

- Rationale: lightweight, fast unit test runner that integrates well with Vite.
- Typical files: `vitest.config.ts`, `vitest.setup.ts`.
- Agent checks: `pnpm run test` or `pnpm run test:ci` for headless runs. Prefer unit tests mock external services; E2E tests should be gated by environment variables.
- Core tests
    - Config in `vite.config.ts`
    - Folders:
        - `tests/unit`
        - `tests/integration`
- E2E tests
    - Config in `vitest.e2e.config.ts`
    - Folders:
        - `tests/e2e`
        - `tests/integration`
    - `test/e2e`: 
- Additional folders (such as 'helpers') in 'tests/` as needed