---
name: squash-merge-and-clean-up
description: Squash-merge this session's PR, then remove the branches and worktree it created.
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [prs, github, branching, git, worktrees]
---

# squash-merge-and-clean-up

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

Land the PR this session opened and clear what this session created: leave the worktree, squash-merge, delete both branches, remove the worktree.

**Session-scoped throughout.** Other agent sessions run concurrently in the same repo and their worktrees and branches sit in the same listings. Touch only what **this** session made — the worktree it entered, the branch it pushed, the PR it opened. Everything else is someone's work in progress.

**Two gates, split on ownership.** Gate one covers everything this session owns: the merge and the removal of its own worktree and branches. Gate two covers the one step that touches **shared** state — fast-forwarding the primary checkout, which every other session reads.

## 1. Survey

Gather, and hold for the plan:

- `gh pr view <n> --json number,title,state,mergeable,mergeStateStatus,headRefName,baseRefName`
- `gh pr checks <n>` — a docs-only PR may report no checks, which is a pass, not a failure
- `git worktree list` — note which entries this session created; `locked` marks another session's
- `git branch -vv` — the `+` prefix marks a branch checked out in some worktree

## 2. Gate one: plan the merge and the removal, then ask

Write the plan for steps 3 to 5 as a short numbered list naming the actual PR number, branch name and worktree path. Put it to the human with `AskUserQuestion` as a **yes/no** question:

> Should I {plan}?

Options are **Yes, run it** and **No, stop here**. One question, plan inlined, no sub-choices. Proceed only on yes.

## 3. Leave the worktree first

Exit the worktree **before** merging, and return the session to the primary checkout.

**This ordering is the whole point of the step.** `gh pr merge` run from inside a worktree merges server-side and *then* fails locally with `fatal: '<base>' is already used by worktree at …`, because its local step tries to check out the base branch that the primary checkout holds. It exits non-zero on a merge that succeeded, and `--delete-branch` never runs. Retrying reads as the obvious fix and is wrong — the PR is already merged.

Use `ExitWorktree` with `action: "keep"` where available; the worktree is removed in step 5, once its content is confirmed landed.

## 4. Squash-merge, then verify

```
gh pr merge <n> --squash --subject "<type>(<scope>): <subject> (#<issue>) (#<pr>)"
```

The subject follows the repo's convention: the conventional-commit subject, the issue number, then the PR number.

**Confirm from the API, not the exit code:**

```
gh pr view <n> --json state,mergedAt,mergeCommit
```

`state: MERGED` with a `mergeCommit.oid` is the completion criterion. When it says merged, the merge is done however the command exited.

## 5. Remove what this session created

Explicit, in order, each with its precondition:

1. **Confirm nothing is lost** — `git fetch origin --prune`, then `git diff --stat <worktree-HEAD> origin/<base>`. An empty diff proves the squash carried the content. The local commit is discarded by removal, so check before removing, not after.
2. **Remove the worktree** — `git worktree remove <path>`, then `git worktree prune`. Skip any entry marked `locked` or that this session did not create.
3. **Delete the local branch** — `git branch -D <branch>`. Step 2 is what frees it: a branch checked out in a worktree cannot be deleted.
4. **Delete the remote branch** — `git push origin --delete <branch>`. By hand, because `--delete-branch` on the merge often will not have run.

## 6. Gate two: fast-forward the primary checkout

Everything above touched only this session's own state. This step moves the checkout **every concurrent session shares**, so it is asked separately even after a yes at gate one.

Report the checkout's state first — whether it is clean, and how far behind — then ask with `AskUserQuestion`:

> Fast-forward the primary checkout from {current} to {origin/base}?

Options are **Yes, fast-forward** and **No, leave it**. On yes, `git merge --ff-only origin/<base>`, and only when the checkout is clean and `0` ahead. A **no** is a clean finish, not a failure.

Close by reporting what was removed and anything left standing. An untracked file you did not write, or a `locked` worktree, belongs to another session or the human: name it and leave it.
