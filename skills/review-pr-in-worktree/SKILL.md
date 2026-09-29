---
name: review-pr-in-worktree
description: Review a pull request in a throwaway worktree — read what it actually committed, run the tests, check it against the ticket and specs that are its true plan, and report every form of drift.
argument-hint: "The PR (number or URL), and the repo (owner/name) if not the current one"
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [prs, github, review, git, worktrees, specs, tests]
---

# review-pr-in-worktree

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

Check a PR out into a worktree of its own and review it there: what it committed, whether its tests pass, whether it matches the plan it claims to implement, and where it has drifted from that plan. Report; do not repair.

**This skill settles which PR; the rest is shared.** Checking the PR out, reviewing it round by round, and tearing the checkout down again is the [worktree review](./worktree-review.md), which `/review-pr-in-lane` follows too. The review itself — what to read, how to run the gate, what to hold the code to, and the fixed report that ends in `yes` or `no` — is `/review-a-pr-and-report`, which that flow calls for every round.

## Process

### 1. Establish the repo and the PR — use what you have, ask rather than search

Resolve both from what is **already in front of you**, in this order:

1. The argument, if given (a PR number, a PR URL, `owner/name`).
2. The current working directory's git remote and branch — `gh` infers the repo inside a clone, and `gh pr status` names the PR for the current branch.
3. Unambiguous existing context — this session opened the PR, or the human named one earlier and nothing since suggests otherwise.

If none of those settles it, or two of them disagree, **ask, immediately** — one question, as the first thing you do. Never pick a PR because it is the newest, because it is the only open one, or because its title looks like what the human was talking about. A review of the wrong PR wastes the whole session and reads as authoritative while it does it.

**Done when:** you have exactly one repo and one PR number, reached without a search.

### 2. Review it in a worktree

Follow [worktree-review.md](./worktree-review.md) from its first section, with that repo and PR. It surveys the PR, checks it out detached, reviews each round and prints the verdict block, parks for you between rounds, cleans up, and confirms what became of the PR.

**Done when:** that flow's last section has reported the PR's state.

## Where this sits in the flow

`/create-pr-for-branch` opens it, **`/review-pr-in-worktree`** judges it — as many rounds as the author needs — and `/squash-merge-and-clean-up` lands it. Each is a separate door a human opens, because each answers to a different person: the author, the reviewer, the maintainer. The merge usually happens in another session entirely, which is why the flow ends by asking the tracker what became of the PR rather than assuming its own report was the last word.

Inside this one, `/review-a-pr-and-report` is the reviewing itself, split out because it is the same judgment wherever the checkout came from. The worktree review is what makes that checkout safe and takes it away again, shared with `/review-pr-in-lane`, which differs only in how the PR is found: it waits for a lane to open one.
