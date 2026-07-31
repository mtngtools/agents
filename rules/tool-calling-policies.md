---
name: tool-calling-policies
description: Security and auditability constraints for tool calls and network access.
metadata:
  type: rule
  discoverable: true
  applies-to: [security, network, credentials, external-services, logging]
---

# Tool Calling & Network Usage Policies

Conservative rules for tools, network services, and external systems. Priority: security and auditability.

## Network calls
- Run local prepublish (typecheck → lint → build → test) before remote calls
- Only call external services when: human-authorized OR allowed by repo config (CI with credentials, E2E with env vars)

## Secrets
- Never print or commit API keys, credentials, tokens
- Use environment variables or CI secret stores
- Delete temporary files (e.g., `RELEASE_BODY_*.md`) before finishing; don't commit

## Writes
- Prefer PR with minimal diffs over direct pushes to protected branches
- Require explicit maintainer approval for destructive ops (publish to npm, create releases, modify branch rules)
- Record approval in PR or issue comment

## Logging
- Emit concise logs of actions and external calls
- Never leak secrets in logs
- Store in CI output or secure artifacts; don't write to repo
