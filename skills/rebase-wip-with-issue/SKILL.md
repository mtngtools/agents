---
name: rebase-wip-with-issue
description: Rebase all WIP commits using GitHub issue.
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [git, rebase, issues, wip]
---

# rebase-wip-with-issue

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

Read back in prior git commits for the recent "wip" commits. 

Rebase these wip commits with supplied issue, but not closing it, as we probably will have additional changes.

Do not just add together all the commit message. I don't care about interim changes. Compare beginning and end and draft a new message from scratch. 

You MUST invoke the `draft-commit-message` skill to draft the commit message, remembering to include the issue number at the end of the first line and at the bottom of the commit message.