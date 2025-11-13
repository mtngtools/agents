# pnpm

- Rationale: fast, disk-efficient package manager with strict dependency resolution. Preferred package manager for TypeScript projects in the organization.
- Typical files: `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml` (for monorepos).
- Agent checks: `pnpm install` should succeed, verify `pnpm-lock.yaml` is committed, use `pnpm` commands in scripts (not `npm` or `yarn`).
- Notes: pnpm uses a content-addressable store and symlinks, which provides better disk efficiency and faster installs. Always use `pnpm` commands in CI/CD and documentation.

