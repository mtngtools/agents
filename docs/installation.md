# Installation

How to install and use skills and rules from `mtngtools/agents`.

## Install with skills.sh

All skills are discoverable via [skills.sh](https://www.skills.sh/).

```bash
pnpx skills add mtngtools/agents
```

This installs all skills from the repository. Skills are then available by name in your agent.

### Available skills

- **repo/** — git, branching, commits, PRs (10 skills)
- **general/** — content refinement (1 skill)
- **front-end/** — Vue building (2 skills)

See [skills/INDEX.md](../skills/INDEX.md) for descriptions and tags.

### Use a skill

```bash
/commit-with-issue
/create-pr-for-branch
/commit-without-issue
```

Skills have YAML frontmatter with `name`, `description`, `metadata.type`, and `metadata.applies-to` tags for discovery.

## Rules (not via skills.sh)

Rules in `rules/` are reference guidance documents, not executable skills. Skills.sh doesn't discover them—agents load them by reading `AGENTS_REPO.md` or context. Rules are indexed in `rules/INDEX.md`.

### Auto-discoverable rules

Always available to agents:
- [**communication**](./rules/communication.md) — style, tone, directness
- [**tool-calling-policies**](./rules/tool-calling-policies.md) — security, network safety, secrets

Load these at session start if you want agents to automatically apply org communication standards.

### Load-when-needed rules

Load only when the session requires them:
- [**git-and-github**](./rules/git-and-github.md) — before git commits, PRs, releases
- [**agent-behavior**](./rules/agent-behavior.md) — when initializing agent constraints

Document in `AGENTS_REPO.md` which rules apply to your repo:
```markdown
**Load when needed:**
- [**Git and GitHub**](https://github.com/mtngtools/agents/blob/main/rules/git-and-github.md) 
  — before commits, PRs, releases
```

## Metadata

Each rule and skill has `metadata` with:
- **`type`** — `rule`, `command`, or `skill`
- **`applies-to`** — tags for discovery and filtering (e.g., `[commits, git, github]`)
- **`discoverable`** (rules only) — `true` (auto-load) or `false` (human-initiated)

Use these tags to:
- Filter what agents should load ("I'm doing git work, load all `applies-to: [git]` rules")
- Display available skills/rules ("show me all rules tagged `security`")
- Organize documentation and discovery indexes

## Example: Setting up a repo

1. Create `AGENTS_REPO.md` in your repo root:
```markdown
## Organization rules & skills

Follow [AGENTS_ORGANIZATION.md](https://github.com/mtngtools/agents/blob/main/AGENTS_ORGANIZATION.md).

**Auto-loaded:**
- [Communication](https://github.com/mtngtools/agents/blob/main/rules/communication.md)
- [Tool Calling Policies](https://github.com/mtngtools/agents/blob/main/rules/tool-calling-policies.md)

**Load when doing git work:**
- [Git and GitHub](https://github.com/mtngtools/agents/blob/main/rules/git-and-github.md)
- Use skills like [`/commit-with-issue`](https://github.com/mtngtools/agents/blob/main/skills/repo/commit-with-issue/SKILL.md)
```

2. Agents read AGENTS_REPO.md and know which rules/skills to load based on the session context.

3. If a skill or rule is not installed locally, pull from `mtngtools/agents`.

## For other systems

**Claude Code:** Reference skills in `AGENTS_REPO.md`. Claude will read and apply the skill instructions.

**Antigravity:** Map `metadata.type` fields:
- `command` → Antigravity command
- `skill` → Antigravity skill/composite

**Custom:** Parse YAML frontmatter for name, description, and metadata.
