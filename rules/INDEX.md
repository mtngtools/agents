# Rules Index

Definitive rules for `mtngtools` organization. Each rule has frontmatter indicating discovery mode.

## Auto-discoverable rules

Agents can autonomously discover and apply these:

| Name | Description | Applies to |
|------|-------------|-----------|
| [communication](./communication.md) | Communication style and tone | communication, output, tone |
| [tool-calling-policies](./tool-calling-policies.md) | Security constraints for tools and network | security, network, credentials |

## Human-initiated rules

Agents must be explicitly directed to these; humans set context:

| Name | Description | Applies to |
|------|-------------|-----------|
| [agent-behavior](./agent-behavior.md) | Conservative work principles | commits, gating, authorization |
| [git-and-github](./git-and-github.md) | Git and GitHub conventions | commits, git, github, branching |

## Loading

Reference rules in output as `/rule-name` (e.g., `/git-and-github`). If not installed in your system, pull from [`mtngtools/agents`](https://github.com/mtngtools/agents) `rules/` folder.

Auto-discoverable rules should be loaded at session start or when relevant to the task. Human-initiated rules are loaded when humans provide context that requires them.
