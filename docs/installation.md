# Installation & Discovery

How to install and use rules and skills from `mtngtools/agents`.

## Skills

Skills in `skills/` follow [skills.sh](https://www.skills.sh/) format with YAML frontmatter (`name`, `description`, `metadata`).

### Install in Claude Code

Reference in `AGENTS_REPO.md` with a link to the skill:
```markdown
See [`/commit-with-issue`](https://github.com/mtngtools/agents/blob/main/skills/repo/commit-with-issue/SKILL.md) skill.
```

Claude reads the skill's instructions and applies them.

### Install in other agents

If your agent supports skills.sh or similar discovery:
- Look in `skills/` directory structure
- Parse YAML frontmatter for `name`, `description`, `metadata.applies-to`
- Invoke skills by name or tag

## Rules

Rules in `rules/` are reference guidance documents, not procedural skills. They're indexed in `rules/INDEX.md`.

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

## For Antigravity agents

If using Antigravity's agent system, rules can be wrapped as directives and skills as commands. The `metadata.type` field maps:
- `rule` → directive or guidance doc
- `command` → Antigravity command (parse SKILL.md for steps)
- `skill` → Antigravity skill/composite command

Document the mapping in your Antigravity configuration.
