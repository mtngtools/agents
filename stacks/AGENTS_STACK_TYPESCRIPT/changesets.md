# Changesets

- Rationale: version management tool for monorepos. Handles independent (per-package) versioning or fixed single-version strategies.
- Typical files: `.changeset/` directory with change files, `changeset` config in `package.json`.
- Agent checks: Ensure changesets are added for version bumps, verify changeset files are included in PRs when making versioned changes.
- Notes: Use independent (per-package) versioning with `changesets` for most monorepos, or fixed single-version for tightly-coupled packages. Changesets help manage version bumps and changelog generation.

