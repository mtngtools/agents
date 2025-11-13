# Turbo

- Rationale: high-performance build system and task runner for monorepos. Provides intelligent caching and parallel task execution.
- Typical files: `turbo.json` for pipeline definitions and caching configuration.
- Agent checks: Verify `turbo.json` is configured, run `pnpm run build` (should use Turbo pipeline), ensure tasks are properly defined in the pipeline.
- Notes: Turbo provides intelligent caching based on file changes and dependencies. Use Turbo for orchestrating builds, tests, and other tasks across packages in a monorepo.

