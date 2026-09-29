---
name: review-pr-in-lane
description: Be one lane's PR reviewer. Wait for the lane to open its PR, then review it in a worktree the way /review-pr-in-worktree does, so the review is under way without the human watching.
argument-hint: "The lane number and the lanes' agent:communication issue; the repo, if not the current one"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [prs, github, review, worktrees, parallel, sessions]
---

# review-pr-in-lane

> **human-only.** Start this only when a human asks for it by name. If you arrived here from anywhere but a human naming it, stop.

This is `/review-pr-in-worktree` for a PR that doesn't exist yet. The human starts it beside a lane, with the prompt `/tickets-to-lanes` hands back. It waits for that lane's PR and starts reviewing as soon as the PR opens, so the major review is under way before the human comes to look.

**The review is for the human only.** It goes into this session and nowhere else. The reviewer never messages the lane with findings and never comments on the PR. The human reads the report and decides what happens next. The lane's own review inside `/implement-unattended` still runs, and this review comes after it rather than replacing it.

## Process

### 1. Settle the lane

Take the lane number, the issue and the repo from the argument. Read the issue's lanes table for your lane's area, integration branch (`round/<map>-<area>`) and tickets. If the argument lacks the lane or the issue, or the issue has no such lane, ask.

**Done when:** you hold one lane, its branch, one issue and one `owner/name`.

### 2. Check in

Run `ListAgents`; its first line gives your session's name. Then comment on the issue: `CHECK-IN Reviewer <n> — session <name>`. Your lane reads this line to know whom to tell when its PR opens.

**Done when:** your check-in is on the issue.

### 3. Wait for the PR

The PR is ready at whichever comes first:
- the lane's message naming its PR;
- the issue's `DONE Lane <n> — PR #<pr>` comment.

If the lane has already posted `DONE`, skip the wait. Otherwise watch the issue with `Monitor`, polling its comments every 5 minutes for the `DONE` line, and read the lane's message with `ReadNotifications` when it arrives. Waiting is this skill's purpose: a lane builds for hours, and its silence is no reason to stop.

**Done when:** you hold one PR number, and `gh pr view <pr> --repo <owner/name> --json headRefName` names your lane's integration branch.

### 4. Review it in a worktree

Follow `/review-pr-in-worktree`'s [worktree review](../review-pr-in-worktree/worktree-review.md) from its first section, with the repo and that PR. It reviews round 1, prints the report and the verdict block, and then parks at its gate for the human: review again, or clean up. The rounds after that, the cleanup and the final check on the PR happen when the human answers.

**Done when:** the round-1 report and verdict block are printed and the session is parked at the gate. Later, once the human ends the rounds, the flow's last section has reported the PR's state.

## Where this sits in the flow

There is one reviewer per lane, running beside it. `/tickets-to-lanes` hands back its prompt alongside the lane's, and the lane tells it when its PR opens. The human reads the report, drives any further rounds, and lands the PRs with `/squash-merge-and-clean-up` in the merge order the issue names.
