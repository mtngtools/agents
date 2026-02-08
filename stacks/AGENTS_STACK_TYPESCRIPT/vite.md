# Vite

- Rationale: fast dev server and build tooling for modern TS projects, good DX and d.ts generation with plugins.
- Typical files: `vite.config.ts`, `tsconfig.json`, `package.json` scripts (build/preview/dev).
- Agent checks: `pnpm run build` (should succeed), ensure `vite-plugin-dts` is configured if the project exports types.
- Notes: ensure build outputs land in `dist/` and that d.ts files are generated when needed for packages.
- Use version of Vite with Rolldown support, either:
  - `vite@8.0.0-beta.13` (preferred)
  - `rolldown-vite`
    - Overriding in package.json to use the [`rolldown-vite`](https://vite.dev/guide/rolldown) drop in replacement.
- Configure for both 'es' and 'cjs'
  - But no 'cjs' if:
    - Nuxt, Vue, h3 libraries or apps.
    - Types package (with package name ending `-types`)