# Installation

How to install and use skills and rules from `mtngtools/agents`.

## Install with skills.sh

All skills are discoverable via [skills.sh](https://www.skills.sh/).

```bash
pnpx skills add mtngtools/agents
```

This installs all skills from the repository. Skills are then available by name in your agent.

### Available skills

- **repo/** — git, branching, commits, PRs (13 skills)
- **plan/** — the wayfinder pipeline (4 skills)
- **general/** — content refinement and reduction (5 skills)
- **front-end/** — Vue building (1 skill)

See [skills/INDEX.md](../skills/INDEX.md) for descriptions, tags, and each skill's invocation category.

### Use a skill

```bash
/commit-with-issue
/create-pr-for-branch
/commit-without-issue
```

Skills have YAML frontmatter with `name`, `description`, `metadata.type`, `metadata.invocation`, and `metadata.applies-to` tags for discovery.

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

## The two metadata axes

Every skill declares two independent things:

- **`metadata.type`** — *what kind of thing this is*: `command` for a discrete
  operation, `skill` for one that composes other skills. Gemini and Antigravity
  map this onto their own surfaces, and `rules/` uses the same vocabulary.
- **`metadata.invocation`** — *who may start it*. See below.

They answer different questions for different consumers, so both are required
on every skill.

## Invocation categories

Every skill declares `metadata.invocation`, which says who may start it.

| Category | Who may start it | Frontmatter |
|---|---|---|
| `human-only` | Only a human, by name. A skill that chains into it must stop and confirm. | `disable-model-invocation: true` |
| `skill-callable` | A human by name, or another skill chaining in silently. Not offered to the model. | `disable-model-invocation: true` |
| `model-discoverable` | The model may reach for it unprompted. | *(flag omitted)* |

`disable-model-invocation` keeps a skill out of the model's listing, and Claude
Code honours it natively — so neither `human-only` nor `skill-callable` is ever
offered to the model. The harness cannot separate those two: to it they look
identical.

What separates them is the category itself, plus the gate paragraph every
`human-only` skill carries at the top of its body:

> **human-only.** Start this only when a human asks for it by name. If you
> arrived here from another skill, stop and get explicit confirmation before
> running any step.

An agent that reaches a `human-only` skill from anywhere other than a human
naming it is told, in the skill itself, to stop and confirm. This is a
convention agents follow, not something a harness enforces.

### Tags

`metadata.applies-to` carries tags for discovery and filtering (e.g.
`[commits, git, github]`). Use them to filter what agents load ("I'm doing git
work, load everything tagged `git`") and to organise discovery indexes.

Rules in `rules/` keep their own `metadata.type` and `metadata.discoverable`
fields; those are a separate vocabulary and are unaffected by the above.

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

**Claude Code:** honours `disable-model-invocation` natively, which withholds
`human-only` and `skill-callable` skills from the model. The split between those
two rests on the gate paragraph described above.

**Gemini CLI / Antigravity:** map `metadata.type`:

- `command` → Antigravity command
- `skill` → Antigravity skill/composite

This is what makes the skills work there, so `metadata.type` is not optional.

Gemini CLI also discovers `~/.agents/skills/` and `.agents/skills/` natively, so
the install path already works and `model-discoverable` skills need nothing
further. Gemini has no `disable-model-invocation` equivalent; `human-only` and
`skill-callable` skills are reached there as custom commands — a
`gemini/commands/<name>.toml` per skill, whose `prompt` points at the skill and
whose `description` names it in `/help`. Those live in
[`gemini/commands/`](../gemini/commands/).

`human-only` rests on the same gate paragraph there, carried in both the skill
body and the command prompt, with Gemini's own `activate_skill` consent prompt
underneath it — Gemini already asks before any model-initiated activation, so
its floor is stricter than Claude's.

**Custom:** parse the YAML frontmatter for `name`, `description`,
`metadata.type`, `metadata.invocation`, and `metadata.applies-to`.
