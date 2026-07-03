---
name: draft-commit-message
description: Draft a commit message following the organization's git and commit message guidelines.
type: command
---

# Draft Commit Message

Draft a commit message for the current changes according to the conventional commit guidelines and organization policies defined in `AGENTS_ORGANIZATION.md`.

## Commit Message Policy & Guidelines

### 1. Git and GitHub Operations
- Use command-line `git` for all git operations when possible.
- For GitHub specific operations (e.g. creating PRs, issues, projects, etc.), use the GitHub CLI: `gh`. 

### 2. Format & Style
- Follow conventional commit guidelines: `https://www.conventionalcommits.org/en/v1.0.0/#summary`
- Use lowercase types (e.g., `feat`, `fix`, `chore`, `docs`, `refactor`, `style`, `test`).
- Use the imperative mood in the description (e.g., "add", "fix", "change" instead of "added", "fixes", "changed").

### 3. Issue References
- Unless an exception is human-approved, always include a reference to the affected issue on the first line (so it appears in GitHub commit lists), for example:  
  `feat(api)!: return data to client (#123)`
- If the issue number is not known, request it from human maintainers before committing, or obtain permission to proceed without an issue number.

### 4. Commit Message Body & Footer
- Add further details about the issue at the bottom of the body.
- If the issue is completed by this commit, use close keywords supported by GitHub: "close, closes, closed, fix, fixes, fixed, resolve, resolves, resolved".
  - Example:
    ```
    feat(api)!: return data to the client (#123)

    This change returns data to the client in a special way.

    [verb] #123
    ``` 
- If the work is ongoing, use "addresses" or "refs" to indicate ongoing work.
- For the [verb], do not mark as closes, fixes, resolves unless specifically asked to, others say "addresses".
- Use `BREAKING CHANGE:` to indicate breaking changes (or `!` after type/scope).
- When multiple issues are impacted:
  - List each reference comma-separated on the first line (e.g., `feat(api): update endpoints (#123, #124)`).
  - List each reference on its own line with details at the bottom of the commit message.

### DO NOT FORGET TO
- List issues numbers of first line without verbs next to number
- List issues numbers on bottom with verbs next to them (when appropriate for context)

