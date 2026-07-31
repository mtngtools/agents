---
name: rebase-wip-with-issue
description: Rebase all WIP commits using GitHub issue.
metadata:
  type: command
  applies-to: [git, rebase, issues, wip]
---

# rebase-wip-with-issue

Read back in prior git commits for the recent "wip" commits. 

Rebase these wip commits with supplied issue, but not closing it, as we probably will have additional changes.

Do not just add together all the commit message. I don't care about interim changes. Compare beginning and end and draft a new message from scratch. 

You MUST follow the [draft-commit-message](../draft-commit-message/SKILL.md) skill to draft the commit message, remembering to include the issue number at the end of the first line and at the bottom of the commit message.