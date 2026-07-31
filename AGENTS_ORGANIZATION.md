# AGENTS_ORGANIZATION

Organization-level agent guidance for [`mtngtools`](https://github.com/mtngtools). Definitive copy in [`mtngtools/agents`](https://github.com/mtngtools/agents) on GitHub.

If not in context, pull from GitHub or ask the user to provide it.

## Guidance hierarchy

Agents follow guidance in this order, with lower levels overriding higher:

1. **AGENTS_ORGANIZATION** (this file) — org-level conventions
2. **AGENTS_STACK_{stackname}** — tech stack guidance in [`stacks/`](./stacks/)
3. **AGENTS_REPO** — repo-level guidance in target repo's root
4. **AGENTS** — package-level guidance in target package's root

Templates: [`./templates/TEMPLATE_AGENTS_REPO.md`](./templates/TEMPLATE_AGENTS_REPO.md) and [`./templates/TEMPLATE_AGENTS.md`](./templates/TEMPLATE_AGENTS.md).

## Discovery model

Rules and skills have frontmatter declaring `discoverable: true | false`:

- **Auto-discoverable** (load at session start) — behavioral constraints and safety rules
- **Human-initiated** (load when requested) — operational skills and policies

See [`rules/INDEX.md`](./rules/INDEX.md) and [`skills/INDEX.md`](./skills/INDEX.md) for discovery status and `applies-to` tags.

## Quick start

**For agents:** Run local prepublish (typecheck → lint → build → test), then open a PR. See [`AGENTS_STACK_TYPESCRIPT`](./stacks/AGENTS_STACK_TYPESCRIPT/README.md) for language-specific commands.

**For specs:** New features require updated specs approved by humans.

**For rules and skills:** Load based on discovery status. Reference by name (e.g., `/git-and-github`, `/commit-with-issue`). If not installed, pull from `mtngtools/agents`.

---

This file is a living draft.