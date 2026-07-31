---
name: git-and-github
description: Git CLI, GitHub workflow, and commit message conventions.
metadata:
  type: rule
  discoverable: false
  applies-to: [commits, git, github, branching, releases]
---

# Git and GitHub Conventions

## Tools
- Use `git` for all git operations, `gh` for GitHub (issues, PRs, projects, releases)

## Commit messages
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#summary): `type(scope): subject (#issue-number)`
- Always include issue reference on first line. Example: `feat(api)!: return data (#123)`
- Body: GitHub keywords (`closes`, `addresses`) + issue details
- Use `BREAKING CHANGE` for breaking changes
- Multiple issues: comma-separated on first line, detail at bottom

## Branching
- Never commit to `main` or protected branches
- Use feature branches (short-lived) OR long-running `develop-{description}` branches
- Don't push until gating passes
- Exception: maintainers may request draft PR before gating

## Releases & tags
- Coordinate with maintainers; prefer PR with release notes over direct tags
