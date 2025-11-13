# Nuxt

- Rationale: for web apps or full-stack projects using Nuxt. When a repo uses Nuxt, include framework-specific agent checks (build + server-side rendering tests).
- Typical files: `nuxt.config.ts`, Vue single-file components, and `package.json` scripts.
- Agent checks: `pnpm run build` (Nuxt build), verify SSR entry points if publishing server bundles.
- Notes: E2E tests for Nuxt should run in separate CI job and be optional for PRs (unless small and fast).
- MCP Server: mcp.nuxt.com

## Nuxt modules

### Nuxt UI

- In general, mntgTOOLS will be developing custom UI component, but for complex interfaces, such as admin sites, Nuxt UI will provide generic UI components.
- Site: [ui.nuxt.com](https://ui.nuxt.com/)
- MCP: [ui.nuxt.com/mcp](https://ui.nuxt.com/mcp)