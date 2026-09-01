---
name: respond-to-pr-review
description: Work through a PR review from another session item by item — fix it or discuss it, one at a time, and record on the tracker any approval the discussion produced.
argument-hint: "The review itself, pasted — plus the repo and PR number if the paste doesn't name them"
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [prs, github, review, git, tickets, approvals]
---

# Respond to PR Review

> **human-only.** Start this only when a human asks for it by name. A review landing in the conversation is not permission to start answering it — the human decides when the response happens, and every item's disposition below is theirs, not yours.

The other side of `/review-pr-in-worktree`. That skill produced a report and stopped; this one answers it — **item by item, each one fixed or discussed, never silently skipped** — and leaves the PR ready for the next round.

**Almost always the review is pasted straight after the invocation**, in the shape `/review-a-pr-and-report` produces: headings down to **Should this be merged**, ending in `yes` or `no` with bullets. Take that paste as the input. It may also arrive as a GitHub review, a comment thread, or a human's own prose — same process, and step 1 says how to get it.

**Every item ends in a disposition the human chose.** Fixed, or discussed to an outcome, or declined with a reason. A finding you disagree with is a **discuss**, never a quiet skip: the reviewer is another session with its own read of the specs, it can be wrong, and settling that is a conversation rather than a decision you make alone.

**Where the outcome of a discussion lives is the tracker, not the tree** — the `approval-policy` skill owns that, and [step 6](#6-record-what-the-discussion-settled) is where this flow applies it. It is the half that gets forgotten.

## Process

### 1. Settle what you are answering, and where

Three things before any of it:

- **The review.** If it was pasted, that is it. If not, pull what is actually on the PR — `gh pr view <n> --json reviews,comments` — and say which review you are answering when there is more than one. Never respond to a review you have only been told about.
- **The PR and its head SHA.** `gh pr view <n> --json number,title,body,state,headRefName,headRefOid,baseRefName,closingIssuesReferences`. The review judged one commit; if the branch has moved since, say so before starting — some items may already be answered.
- **A checkout of the PR branch to work in.** The branch's own worktree if this session or another built it there; otherwise make one, the way the `implement` skill's step 2 does. **Not the primary checkout** — other sessions share it.

**Done when:** you have the review's text, the PR and its head SHA, and a checkout on that branch.

### 2. Itemize the review before answering any of it

Turn the report into a **numbered list of items**, and show it to the human before the first question. What becomes an item:

| From the report | Becomes |
|---|---|
| Each bullet under **Should this be merged: no** | An item — these are the blocking ones |
| Each line under **Drift** | An item, carrying the authority it cites |
| Each criterion marked **missing** or **not verifiable** under **Spec Compliance** | An item |
| Each failure or gap under **Test Coverage** | An item |
| Each **Additional Consideration** | An item, labelled **non-blocking** |
| Each line under **What I could not verify** | An item — usually answered by information, not by a code change |

Give each item a number, one line of what it claims, where it points, and the authority it names — spec, ADR, ticket, or "preference". Numbers matter: they are what the human answers by, what the commits reference, and what the reply at step 5 is organised around.

**Do not start fixing while itemizing.** An obvious one-line fix is still an item with a disposition; deciding it yourself because it was easy is how the list stops matching what actually happened.

**Done when:** every part of the review appears exactly once in a numbered list the human has seen.

### 3. One item at a time: fix, discuss, or decline

Walk the list in order — blocking items first, non-blocking after. **One question per item**, with these options, and do not batch several items into one question:

- **Fix it** — the finding stands. Make the change now, in this checkout.
- **Discuss it** — the finding is contested, unclear, or bigger than the PR. Talk it through with the human until there is an outcome, then record it per `approval-policy`. "Discuss" is not a deferral; it ends in an answer.
- **Decline it** — the finding is understood and not being acted on. The human says why, and that reason is recorded on the PR, not dropped.
- **Split it out** — the item is real but belongs in its own ticket. File it, link it, and say so in the reply; the PR is not the place to grow scope.

Where the finding cites a spec or ADR, **read that authority yourself before asking.** A reviewer can misquote, and the human should be answering the actual rule rather than the report's summary of it.

**A discussion that changes a decision is the human's, and it is gated.** If the outcome renames something, changes a spec, or overrides an ADR, follow the repo's own naming authority for how that gets proposed and recorded — and it does not land quietly inside a fix commit.

**Done when:** every numbered item has one of the four dispositions, chosen by the human.

### 4. Land the fixes as new commits — never over the reviewed head

Commit the fixes on the PR branch, conventional message, referencing the item numbers where it helps the next round read them.

**Do not rewrite the reviewed commit.** No amend of the head the review names, no force-push, no rebase that moves it. The next round diffs `<reviewed-head>..<new-head>` to see what you did in response; a rewritten history destroys exactly that, and the reviewer has to start over.

Run the repo's gate before you say anything is fixed. An item marked fixed on a red gate is not fixed.

**Done when:** every **fix it** item has a commit behind it and the gate is green — or you have said plainly which failed and why.

### 5. Reply on the PR, once, with the human's approval

Draft one comment covering the whole list — item number, disposition, and evidence: the commit SHA for a fix, the outcome for a discussion, the reason for a decline, the new ticket for a split. Show the draft to the human and **get their go-ahead before posting**; the comment is outward-facing and it is signed by them, not by you.

Then post it, and say what the new head SHA is so the review's next round has a target.

**Do not merge, and do not ask to.** The merge is the human's, through `/squash-merge-and-clean-up`, after a round that comes back `yes`.

**Done when:** the reply is posted with the human's approval, and the new head SHA is named.

### 6. Record what the discussion settled

**Follow the `approval-policy` skill.** It owns where an approval goes, what has to be written, and why none of the discussion belongs in a committed file — this skill just makes sure it happens before the response ends.

Three things this flow adds to it:

- **Every disposition from step 3 is an approval**, including the declines. A finding waved off with a reason is exactly the thing that gets lost, because nothing lands in the tree to say it was ever decided.
- **A marker left by `/pre-pr-naming-approval` comes out of the tree here.** If the human settles a borrowed name during the response, delete its marker in the same commit as the fix, and record their answer per the policy.
- **Where the PR reply is the right home, write it into the step 5 draft** rather than posting a second comment saying the same thing.

**Done when:** every approval given in this session is recorded where `approval-policy` puts it, and nothing about the discussion has been committed to a file.

## Where this sits in the flow

`/implement` builds it, `/create-pr-for-branch` opens it, `/review-pr-in-worktree` judges it, **`/respond-to-pr-review`** answers that judgment, and `/squash-merge-and-clean-up` lands it once a later round comes back `yes`. The review and the response are deliberately different sessions: the reviewer holds the code to its plan without knowing what it cost to write, and this skill answers with the author's context, item by item, in front of the human who decides.
