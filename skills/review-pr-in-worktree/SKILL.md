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

**This skill owns the worktree, not the judgment.** Settling which PR, checking it out, and tearing the checkout down again live here. The review itself — what to read, how to run the gate, what to hold the code to, and the fixed report that ends in `yes` or `no` — is `/review-a-pr-and-report`, called at step 4 and again for every later round.

**Nothing in this flow writes.** No commits, no fixes, no pushes, no PR comments, no merge — even when the fix is one line and obvious. A review that edits its subject stops being a review. Findings go to the human, who decides what happens next. `/squash-merge-and-clean-up` merges; this skill only ever says whether it should.

**A worktree because the review must not disturb anything.** Other sessions share the primary checkout, and reviewing means checking out someone else's head, running their tests, and leaving build output behind. All of that happens in a directory this skill creates and — with permission — removes.

## Process

### 1. Establish the repo and the PR — use what you have, ask rather than search

Resolve both from what is **already in front of you**, in this order:

1. The argument, if given (a PR number, a PR URL, `owner/name`).
2. The current working directory's git remote and branch — `gh` infers the repo inside a clone, and `gh pr status` names the PR for the current branch.
3. Unambiguous existing context — this session opened the PR, or the human named one earlier and nothing since suggests otherwise.

If none of those settles it, or two of them disagree, **ask, immediately** — one question, as the first thing you do. Never pick a PR because it is the newest, because it is the only open one, or because its title looks like what the human was talking about. A review of the wrong PR wastes the whole session and reads as authoritative while it does it.

**Done when:** you have exactly one repo and one PR number, reached without a search.

### 2. Survey before you check anything out

Read the PR from the API first — it is cheap, and it tells you what the review has to cover:

- `gh pr view <n> --json number,title,body,state,author,headRefName,baseRefName,headRefOid,files,commits,closingIssuesReferences`
- `gh pr checks <n>` — a Markdown-only PR may report no checks; that is a pass, not a failure
- `gh pr view <n> --json reviews,comments` — a point already made by a human is theirs, not a fresh finding

Hold the **head SHA**. Everything below is a review of that commit, not of whatever the branch points at ten minutes from now. This survey is also what step 4 is handed, so it is not repeated there.

**Done when:** you know what the PR claims to do, which issue it closes, what CI thinks, and what has already been said.

### 3. Build the review worktree, detached at the head SHA

```
git fetch origin pull/<n>/head
git worktree add <worktree-path> --detach <head-sha>
```

**Put it inside the repo, not beside it, and name it for the PR.** Moving a session into a directory outside the project is a thing most harnesses ask the human to approve separately from any tool permission — a prompt that has nothing to do with the review, and that no allowlist entry suppresses. A path inside the working directory avoids it entirely. The name matters at step 5: the worktree listing is shared with every other session on the machine, and a fixed, predictable path is what lets cleanup tell yours from theirs.

**If you are Claude Code** — use `.claude/worktrees/review-pr-<n>`, the harness's own worktree home, so nothing new has to be trusted. Enter it with `EnterWorktree` and its `path` argument, which requires the worktree to already appear in `git worktree list`, hence the order above.

**Any other agent** — use a repo-local directory your tooling already ignores; `.worktrees/review-pr-<n>` is a safe default, and `.git/info/exclude` will hide it without touching the tracked `.gitignore`. Then make that directory your working directory for the rest of the review, by whatever mechanism your harness provides. From step 4 on, every command must target the worktree; if your harness has no way to move, pass the path explicitly (`git -C <worktree-path> …`) rather than reviewing from the primary checkout by accident.

**Detached on purpose.** No branch is created, so nothing here can be pushed by accident and nothing collides with the author's own checkout of the same branch.

Some harnesses restrict shell redirection inside a worktree. If a heredoc or `>` is refused, use file-writing tools and plain single-purpose commands instead of fighting it.

**Done when:** the session is in a clean worktree whose `HEAD` is the head SHA from step 2.

### 4. Review and report — call `/review-a-pr-and-report`

The worktree is ready, so hand the review to the skill that owns it. Give it what step 2 already established, so it does not go back to the API for any of it:

- the repo and the PR number
- the **head SHA** it is judging, and the base ref
- the worktree path — every command it runs belongs there, never in the primary checkout
- the existing reviews and comments from step 2
- the round: `1` here, and on later rounds the round number plus the previous round's head SHA

It reads the committed diff, runs the repo's gate — or records why there is none, a Markdown-only PR having no gate to run — holds the code to its spec and ticket, hunts the drift, and produces the fixed report ending in `yes` or `no`. Do not restate its verdict, soften it, or append your own; hand it to the human as it came.

**Done when:** the report exists, with its last section reading either exactly `yes` or `no` with its bullets.

### 5. Gate: review again, or clean up

A review that finds something usually gets answered — the author pushes a fix to the **same PR**, and the same worktree is the cheapest place to look at it. So the review does not end at the report; it parks there and asks what comes next. Ask the human one question, with these two options:

> The review is reported. What next?

- **Review again** — changes have been pushed to the PR
- **Clean up** — remove the worktree and everything this session made for this review

A free-text answer is always a third possibility: never enumerate it as an option, and never treat the two above as exhaustive. Take the answer at face value and do only what it asks. **If you are Claude Code**, `AskUserQuestion` is the tool, and it supplies the free-text **Other** for you.

**Stay parked between rounds.** Waiting is the correct state — do not clean up because the report is written, do not poll the PR for new commits, and do not start reviewing again until asked. The worktree costs nothing sitting still.

#### On "review again"

```
git fetch origin pull/<n>/head
```

Read the new head SHA. **If it has not moved, say so and ask again** — re-reading the same commit produces a second opinion on nothing and reads as though something changed.

If it has moved, `git checkout <new-head-sha>` in the worktree — still detached — and call `/review-a-pr-and-report` again, this time naming the round number and the previous round's head SHA. Its **Rounds** section owns what changes between rounds: diffing round against round, re-running the gate every time, and giving every bullet from the last `no` a verdict. Then this gate again — rounds repeat until the human ends them.

#### On "clean up"

Remove what **this session** made for **this review**, and nothing else. Other sessions have worktrees and branches in the same listings; a `locked` entry or a directory this session did not create is someone's work in progress.

1. **Leave the worktree before removing it** — move your working directory back to the primary checkout, without deleting anything as part of leaving. A directory cannot be removed from inside itself, and "leave" must not be the same act as "delete": item 2 below is where deletion is decided. **If you are Claude Code**, that is `ExitWorktree` with `action: "keep"` — which will not remove a worktree entered by path anyway.
2. `git worktree remove <worktree-path>`, then `git worktree prune`.
3. Any scratch, log, or report file this review wrote outside the worktree.
4. Anything the gate left running — containers, a daemon, a stray port — that this session started. Test suites are the usual source, and they do not always clean up after themselves.

**No branch is deleted here, ever.** The head branch belongs to the PR and its author, and the review worktree was detached precisely so this step has no branch of its own to consider. Deleting branches is `/squash-merge-and-clean-up`'s job, after a merge this skill does not perform.

**Done when:** the human has ended the rounds and either the worktree is gone with its residue, or it is standing on purpose and they know where it is.

### 6. Close by confirming what happened to the PR

The last thing to establish is whether the thing you reviewed actually landed — usually it was merged elsewhere, by the human or by another session running `/squash-merge-and-clean-up`, and this session would otherwise never learn the outcome of its own review.

```
gh pr view <n> --json state,mergedAt,mergeCommit,headRefOid
```

Report one of:

- **Merged** — give the merge commit, and say whether `headRefOid` is the SHA you last reviewed. If the PR moved after your final round, the review does not cover what merged; say that plainly.
- **Still open** — say so, name who is expected to land it, and do not offer to merge it yourself.
- **Closed unmerged** — say so. The review stands as the record of why, if that is why.

Check once and report. **Do not poll, wait, or schedule a re-check** — an open PR is a complete, correct ending for this skill.

## Where this sits in the flow

`/create-pr-for-branch` opens it, **`/review-pr-in-worktree`** judges it — as many rounds as the author needs — and `/squash-merge-and-clean-up` lands it. Each is a separate door a human opens, because each answers to a different person: the author, the reviewer, the maintainer. The merge usually happens in another session entirely, which is why this skill ends by asking the tracker what became of the PR rather than assuming its own report was the last word.

Inside this one, `/review-a-pr-and-report` is the reviewing itself, split out because it is the same judgment wherever the checkout came from. This skill is what makes that checkout safe and takes it away again.
