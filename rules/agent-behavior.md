---
name: agent-behavior
description: Conservative principles governing agent work and commits.
metadata:
  type: rule
  discoverable: false
  applies-to: [commits, gating, authorization, branching]
---

# Agent Behavior

- Be conservative: small commits and PRs over direct pushes
- Never commit to main/protected branches: use feature or develop-{description} branches
- Require: feature branch + GitHub issue before starting work
- Gating: prepublish pass + explicit human approval required before commit/push
- Respect maintainer direction: if told "don't commit", keep changes staged only
