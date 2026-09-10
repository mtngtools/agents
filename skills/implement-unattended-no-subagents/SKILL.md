---
name: implement-unattended-no-subagents
description: Build settled tickets with the human away and no fan-out — naming and decision authority granted, built and reviewed by this session alone on one branch, handed back as a single PR with every borrowed call tabled on it.
argument-hint: "The ticket or tickets (issue URLs or numbers), or the spec to build"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [building, tickets, approvals, naming, worktrees, prs]
---

# implement-unattended-no-subagents

> **human-only.** Start this only when a human asks for it by name. This skill hands out naming and decision authority; a skill chaining into it would be manufacturing the human's approval on their behalf. If you arrived here from anywhere but a human naming it, stop.

`/implement-unattended` minus the fan-out. Same grants, same ledger, same handback, and the same single pull request at the end — **one session doing the building itself, in one worktree, on one branch, serially.**

**One round, one PR** — see [one round, one PR](../implement-unattended/unattended.md#one-round-one-pr). Here that is the easy half: there is one branch, so there is nothing to integrate. What it asks of you is that several tickets built serially still land as one pull request rather than one apiece.

Reach for this rather than the fanned-out version when:

- **The repo restricts subagent use.** That restriction outranks any skill, and this is how you work unattended inside it.
- **The tickets are not independent** — a blocking edge or a shared file makes them one serial unit, and a fan-out over them produces branches that cannot both land.
- **There is one ticket.** Fanning out over one buys nothing but a layer between you and the work. Several tickets are just as ordinary here — they build one after another on the one branch, and they land in the one PR.
- **You want the session's own attention on it**, undivided. A single agent that reads the whole tree beats six that each read a slice, when the work is subtle rather than wide.

Read [working unattended](../implement-unattended/unattended.md) first. It is the whole of what changes when there is nobody to ask, and it points at the ledger every borrowed call is held in.

## Process

### 1. Take the terms before touching anything

Read `unattended.md`, and through it the borrowed-authority ledger and `grant-decision-authority`'s **What may be borrowed** section. Then run the ledger's grep:

```
grep -rn "TEMPORARY AGENT" .
```

Hits mean a previous session left loans standing. They go in your PR table as well — a marker you did not make is not yours to remove.

**Done when:** you can state the bounds you are working under, and the tree is either clean of markers or you have listed the ones you inherited.

### 2. Settle the base, then work in a worktree

`/implement`'s steps 1 and 2, unchanged:

```
git fetch origin
git worktree list
gh pr list --state open
git worktree add .claude/worktrees/<round> -b <branch> <base>
```

A freshly fetched `origin/main` is the default — never local `main`, never the current `HEAD`. Inside the repo, not beside it.

**One worktree and one branch for the whole round**, whether that round is one ticket or five. Name both for the round rather than for a single ticket — a branch called after ticket #12 that also carries #13 and #14 misreads at a glance, and the PR it opens misreads with it.

**Where an open PR overlaps the round, that is normally a question for the human, and they are not here.** So it becomes a decision made under the grant: base on `main`, stack on the PR, or leave the affected ticket out — pick one, mark it, and put it in the table as a row. Do not stack on an open PR silently.

**Done when:** the session is in one worktree, on one branch that every ticket in the round will land on, and you can say in one line why that base.

### 3. Build it

`/implement`'s step 3: `/tdd` at pre-agreed seams, typecheck regularly, single test files regularly, the full suite once at the end.

**Several tickets go one at a time, in the order their edges imply**, each committed separately onto the one branch. Separate commits are what keeps a multi-ticket PR readable; separate branches are what would split it. Finish and gate a ticket before starting the next, so a failure has one ticket's worth of diff behind it.

The difference is what happens at a call you cannot approve:

- **In bounds** — make it, write the marker at the site of the act, and continue.
- **Out of bounds** — park it. Do everything that does not depend on it, write down the question and what you would have chosen, and carry on. A parked call that blocks the whole ticket ends the ticket, not the session.

**Done when:** every ticket in the round is either committed to the branch with its criteria met, or named as unfinished with the reason, and the gate is green over the branch as a whole.

### 4. Review it yourself

Nobody else is going to read this before the human does. Run `/code-review`, and go over the same dimensions `subagent-review` fans out — **serially, one at a time, rather than all at once**, because the point of separating them is that each gets its own attention:

spec and ticket conformance · correctness and failure modes · comments the diff falsified · what should have changed and didn't · tests · tree and commit hygiene · every `TEMPORARY AGENT` marker in the range.

That last one is the check that every borrowed call actually reached the table.

**Done when:** each dimension has been walked and its findings either fixed or written into the handback.

### 5. One PR, one table, one note

Commit to the worktree's branch — **never to `main`**. Then `/create-pr-for-branch`: **one pull request for the round, however many tickets it carries.**

The body leads on the ledger's table, generated from one grep over the branch, with the reviewer note verbatim beneath it. Parked calls go under the table as their own list. Then a `Closes #n` line for every ticket the branch actually carries, and a named line for every ticket that did not land.

**Open it normally, not as a draft.**

**Splitting the round across more than one PR is a decision, not a default** — it wants a reason the human would accept, recorded in every PR body and in the handback. A ticket that could not be brought in is unfinished, not a second PR.

**Done when:** one PR exists, its table matches the grep over its branch exactly, every landed ticket is named in it, and nothing has been merged.

### 6. Hand back

The handback shape is in `unattended.md`: what got done, what you borrowed, what you parked, what you could not finish, what you dropped, and the one thing you would ask if you could ask one thing.

Then stop. **Do not wait, poll, or schedule a re-check.**

## Where this sits in the flow

`/specs-to-tickets` writes the tickets; **`/implement-unattended-no-subagents`** builds them with nobody watching and stops at one open PR. `/review-pr-in-worktree` judges it when someone is back, `/respond-to-pr-review` is where the table's rows get answered and the markers come out, and `/squash-merge-and-clean-up` lands it.

`/implement-unattended` is the same mode with the build fanned out across subagents and reviewed by them — and it still hands back one PR, integrating the branches to do it.
