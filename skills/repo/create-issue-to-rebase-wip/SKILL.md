---
name: create-issue-to-rebase-wip
description: Create GitHub issue for prior WIP commit, rebase with issue number referenced.
type: command
---

# create-issue-to-rebase-wip

Read back in prior git commits for the recent "wip" commit. 

Create github issue using 'gh' referencing the summary only of what the commit changed.

Then rebase these wip commits with that issue, but not closing it, as we probably will have additional changes.

You MUST follow the [draft-commit-message](../draft-commit-message/SKILL.md) skill to draft/edit the commit message for the rebase, remembering to include the issue number at the end of the first line and at the bottom of the commit message.