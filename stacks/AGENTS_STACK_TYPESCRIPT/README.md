# AGENTS_TYPESCRIPT.md

TypeScript tooling & stacks (agent guidance)
--------------------------------------------

This document contains TypeScript-specific tooling choices and minimal verification steps agents should run when working on TypeScript projects in the mtngtools organization.

## What to check (quick)

- Run local verification: `pnpm run typecheck && pnpm run lint && pnpm run build && pnpm run test`.
- For E2E: ensure environment gating variable is set (e.g., `AWS_S3_E2E_ENABLED=true`) and documented in the PR.
- When using `gh api` or `gh release`, include a PR or human-authored description and remove temporary files after use.
- Avoid direct pushes to protected branches; open a PR and request the appropriate reviewers (use `CODEOWNERS` where helpful).

## Stack technologies

| Name | Type | Notes | Requested Minimum | More Info |
|------|------|-------|-------------------|-----------|
| [TypeScript (tsc)](typescript.md) | language | Static typing and API guarantees | 5.7+ | [typescriptlang.org](https://www.typescriptlang.org) |
| [Vue](vue.md) | framework | Progressive JavaScript framework | 3.5+ | [vuejs.org](https://vuejs.org) |
| [Nuxt](nuxt.md) | framework | Full-stack Vue framework | 3.14+ | [nuxt.com](https://nuxt.com) |
| [Nitro](nitro.md) | framework | Server-side framework | 2.10+ | [nitro.unjs.io](https://nitro.unjs.io) |
| [Tailwind CSS](tailwind.md) | framework | Utility-first CSS framework | 4.1+ | [tailwindcss.com](https://tailwindcss.com) |
| [pnpm](pnpm.md) | tooling | Package manager | 9+ | [pnpm.io](https://pnpm.io) |
| [Vite](vite.md) | tooling | Fast dev server and build tooling | 6+ | [vitejs.dev](https://vitejs.dev) |
| [oxlint (plugin)](oxlint.md) | tooling | Style enforcement and bug detection | 0.6+ | [oxc-project.github.io](https://oxc-project.github.io) |
| [Vitest](vitest.md) | tooling | Lightweight unit test runner | 2+ | [vitest.dev](https://vitest.dev) |
| [Unjs](unjs.md) | tooling | Universal JavaScript tools | N/A | [unjs.io](https://unjs.io) |
| [Turbo](turbo.md) | tooling | Build system and task runner for monorepos | 2+ | [turbo.build](https://turbo.build) |
| [Changesets](changesets.md) | tooling | Version management for monorepos | 2.27+ | [github.com/changesets](https://github.com/changesets/changesets) |

## Monorepo guidance

- Use `pnpm` workspaces for multi-package repos. Keep `pnpm-workspace.yaml` at the root and list packages explicitly (for example `packages/*`).
- Prefer multiple small packages with clear responsibilities over a single huge package.
- Use CI that runs per-package tests in parallel (via Turbo) and a top-level integration job for cross-package E2E.

## Notes

This file is focused on TypeScript tool choices and verification steps. For organization-wide policies about network usage, secrets, and agent behavior that apply regardless of language, see [`AGENTS_ORGANIZATION.md`](../../AGENTS_ORGANIZATION.md).
