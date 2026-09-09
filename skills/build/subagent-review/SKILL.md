---
name: subagent-review
description: Review committed code with independent subagents, one per dimension, no PR required — findings verified against the tree before any of them reach you.
argument-hint: "What to review — a commit range, a branch, or a ref to compare against (defaults to the merge-base with origin/main)"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [review, subagents, git, specs, tests, drift]
---

# subagent-review

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

An independent review of code that is **committed but not necessarily on a PR** — a branch mid-build, the work `/implement` just finished, a fan-out's output before any of it is proposed. One subagent per dimension, none of them able to see the others, findings verified by you before they are repeated.

**Where the work is already on a PR, `/review-pr-in-worktree` is the better door.** It knows the PR's own context — the head SHA, the existing reviews, what a human already said — and it ends by asking whether to review another round. This skill exists for the case that has none of that: there is no PR, so there is no PR to read.

**Nothing here writes.** No commits, no fixes, no pushes, no comments — even when the fix is one line and obvious. A review that edits its subject stops being a review. Findings go to the human, who decides what happens next.

**How a fan-out is run** — isolation, the brief, the prohibitions, and how results are verified — is [subagent dispatch](../subagent-implement/dispatch.md), shared with `subagent-implement`. Read it before dispatching anything.

## Process

### 1. Settle exactly what is under review

The subject is a **commit range**, never the working tree. Uncommitted changes are not the thing being reviewed and their presence is itself a finding.

Take the range from the human where they named one. Otherwise:

```
git fetch origin
git merge-base origin/main HEAD
git log --oneline <merge-base>..HEAD
git diff --stat <merge-base>...HEAD
```

Three dots on the diff, not two. Two-dot folds in whatever `main` has done since and blames this branch for it.

**Confirm the range before reading a line of it** — say the base, the head SHA, and the commit count in one line. A review of the wrong range is worse than no review, because it reads as thorough.

**A dirty tree is a finding, and it is also a hazard.** Say so, and review the commits regardless — do not stash, do not clean, do not commit anything to tidy it up.

**Done when:** the base ref, the head SHA, and the commit list are established and stated.

### 2. Run the gate once, yourself, before dispatching

The repo's own gate — its check script, its test suite, its typecheck. **The parent runs it, not the reviewers.** Reviewers share one checkout, and a gate run writes build output; two agents running it concurrently is write contention in a checkout everyone else is reading.

Hand the result to every reviewer as context. A Markdown-only change may have no gate to run: record that, it is a pass, not a failure.

**Done when:** you have the gate's actual output, or a stated reason there is none.

### 3. Dispatch one agent per dimension

All reviewers read the **same range** in the **same checkout**, and the brief forbids writing to it in those words. Give each one dimension and nothing else — an agent given three reports well on one.

| Dimension | What it is looking for |
|---|---|
| **Spec and ticket conformance** | Every acceptance criterion, met or not. Work the ticket didn't ask for. A spec or ADR the change contradicts |
| **Correctness** | The failure modes: what happens on the empty case, the error path, the concurrent call, the value nobody validated |
| **Comments as claims** | Every doc comment, inline note, `TODO`, and example in a changed file asserts something about code that just moved. A comment this diff falsified is a finding, and the next reader believes it long after the code stopped matching. Include `TODO`s the change itself completed |
| **What should have changed and didn't** | A renamed concept the call sites still use by its old name. A new branch of behaviour with no error path. A config key added in one environment file and not its siblings |
| **Tests** | What the change added, what it left uncovered, and any test that passes without exercising the new behaviour |
| **Tree and commit hygiene** | Untracked files, build output that wants ignoring, commit messages against the repo's convention, merge commits where the repo squashes, a commit that undoes an earlier one in the same range without saying why |
| **Borrowed authority** | `grep -rn "TEMPORARY AGENT"` over the range's files. Every marker is an unapproved call: it belongs in the report whether or not a PR table exists yet |

Drop a dimension only deliberately, and say in the report that you dropped it.

Every brief carries, on top of what `dispatch.md` requires:

- **The range** — base, head SHA, and the commit list.
- **The gate output** from step 2.
- **Read the changed files whole**, not only the hunks, wherever the change is more than cosmetic. A diff shows what moved; it hides what the file now says.
- **Read only.** No edits, no commits, no `git checkout`, no stash. In those words.
- **Return findings as data**, each with a file, a line, what is wrong, and how it fails — not a narrative.

**No reviewer sees another's findings.** That is the entire value of the fan-out: agreement between agents that read each other is worth nothing.

**Done when:** every dimension has an agent, and each has the range and the gate output.

### 4. Verify every finding before it reaches the human

Findings are claims. `dispatch.md` says why; here it means, for each one:

1. **Reproduce it against the code.** Open the file at the line and confirm it says what the finding says it says.
2. **Drop what you cannot reproduce.** Not soften — drop. A finding that survives as a hedge is a finding the human has to re-check.
3. **Dedupe across dimensions.** The same defect arrives from two angles more often than not; report it once, with both angles as its reasoning.
4. **An agent that returned nothing failed.** Say so. Do not read silence as a clean dimension.

**Done when:** every reported finding has been confirmed by you in the tree, and the dropped ones are dropped.

### 5. Report

One report, one voice. In this order:

1. **What was reviewed** — base, head SHA, commit count, and the dimensions that ran. Name any that didn't.
2. **The gate** — its actual result, or why there was none.
3. **Findings**, worst first: file and line, what is wrong, how it fails.
4. **Anything capped, sampled, or dropped**, including agents that came back empty.
5. **A last line that is exactly `yes` or `no`**, answering: should this go forward as it stands? A `no` is followed by the bullets that have to be answered first.

Hand it over as it is. Do not soften it and do not append a second opinion of your own.

## Where this sits in the flow

`/implement` and `/subagent-implement` produce the commits; **`/subagent-review`** reads them before anybody proposes them. `/create-pr-for-branch` opens the PR, and from there `/review-pr-in-worktree` — which knows the PR — is the review that matters.

`/implement-unattended` runs this as its own last step, because with the human away there is nobody else to read the work before they come back to it.
