---
name: implement-unattended
description: Build settled tickets with the human away — naming and decision authority granted, work fanned out across subagents, integrated onto one branch, and handed back as a single reviewed PR with every borrowed call tabled on it.
argument-hint: "The tickets (issue URLs or numbers), or the map they came from"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [building, tickets, subagents, approvals, naming, worktrees, prs, parallel]
---

# implement-unattended

> **human-only.** Start this only when a human asks for it by name. This skill hands out authority *and* turns on parallel building; a skill chaining into it would be manufacturing both on the human's behalf. If you arrived here from anywhere but a human naming it, stop.

`/implement` builds one ticket with the human reachable. This builds settled tickets with the human **gone**: authority granted so the work does not stall, fanned out so more than one ticket moves, reviewed independently because nobody else will read it, and handed back with every call you had to make written down where they will see it.

**However many tickets go out, one pull request comes back.** The fan-out is how the work gets built, not how it gets handed over — the ticket branches are internal to this session, they merge into one integration branch, and that branch is the whole of what the human is asked to read. See [one round, one PR](./unattended.md#one-round-one-pr).

**Four things turn on at once,** and the human typing this name is the authorization for all four:

| | |
|---|---|
| **Naming authority** | Terms in `grant-naming-authority` |
| **Decision authority** | Terms in `grant-decision-authority` — its bounds are what make this safe |
| **Parallel build** | Method in `subagent-implement` |
| **Independent review** | Method in `subagent-review` |

Read [working unattended](./unattended.md) first. It is the whole of what changes when there is nobody to ask, and it points at the ledger every borrowed call is held in.

**The failure mode here is not stalling — it is overreach.** A session that parks a call costs the human one line in a handback. A session that quietly makes a call it should not have costs them a merged PR they did not mean to approve.

## Process

### 1. Take the terms before touching anything

Read `unattended.md`, and through it the borrowed-authority ledger and `grant-decision-authority`'s bounds. Then run the ledger's grep:

```
grep -rn "TEMPORARY AGENT" .
```

Hits mean a previous session left loans standing. They go in your PR table as well — a marker you did not make is not yours to remove.

Check `AGENTS.md`/`CLAUDE.md` for anything the repo says about subagent use. **A repo that restricts fan-out outranks this skill**, and where it does, `/implement-unattended-no-subagents` is the skill you actually want.

**Done when:** you can state the bounds you are working under, and the tree is either clean of markers or you have listed the ones you inherited.

### 2. Establish independence, then settle one base

Both are `subagent-implement`'s steps 1 and 2, unchanged, and neither relaxes because the human is away — they get stricter, because there is nobody to catch a bad split.

- **Independence:** blocking edges, file overlap, and ordering nobody wrote down. Say per ticket: parallel, serial behind #n, or out of this round.
- **The base:** a freshly fetched `origin/main` unless an open PR overlaps the set. **That overlap is normally a question for the human, and they are not here** — so it is a decision made under the grant: pick, mark it, and put it in the table as a row. Do not stack on an open PR without saying so.

**One base for the whole fan-out** — and the base of the integration branch every ticket lands on in step 4. Different bases per ticket means there is nothing to integrate them onto. In a lane, that base also takes each handoff the lane merges in: see [in a lane](./unattended.md#in-a-lane).

**Done when:** the parallel set is named, every exclusion has a reason, and the base is settled in one line.

### 3. Create the worktrees, then dispatch

Worktrees are created by you, never by the subagents — concurrent creation races the same git index. One per parallel ticket, all off the settled base, inside the repo.

Brief per `subagent-implement`'s step 4, plus what this mode adds:

- **The authorization line**, naming this skill and the date. Without it an agent stalls at the first confirmation gate with nobody to answer.
- **The authority it holds** — naming and decision, on the same terms you hold them, with the ledger's marker rules and the two verbatim marker strings. A subagent that borrows without marking has made a decision nobody can find.
- **The bounds**, copied in. An agent that hits one **parks** the call and returns it; it never routes around it.
- **Commit, do not push, do not open a PR.** The branch is a stage on the way to one shared pull request, not a deliverable of its own. An agent that opens its own PR has split the round.

**Done when:** every parallel ticket has an agent in its own worktree, briefed with its authority and its bounds.

### 4. Verify, integrate, review

Nobody else is going to read this, so all three happen and none is optional.

**Verify** per ticket, per `subagent-implement`'s step 5: read the committed diff against the criteria yourself, run the gate on that branch yourself, treat untracked files as findings, and treat an empty return as a failure rather than a clean run.

**Integrate** — every branch, every time, not only where the units overlapped. Merge them all into a single **integration branch** off the settled base, in its own worktree, and run the gate once over that. Branches that pass alone can still break together, and every agent was blind to the others by design.

```
git worktree add .claude/worktrees/<round> -b <integration-branch> <base>
git -C .claude/worktrees/<round> merge <ticket-branch>   # one per ticket, in edge order
```

Name the branch for the round, not for any one ticket it carries.

**The integration branch is the deliverable** — this overrides `subagent-implement`'s step 5, where the same merge is a throwaway probe because each ticket branch is going on to a PR of its own. Here they are not. It is not a probe and it is not thrown away: it is what the PR is opened from, and from here on it is the only branch that matters. A ticket branch that will not merge cleanly into it is a ticket that did not land — resolve it if the conflict is yours to resolve, and otherwise leave it out, say so, and put it in the handback as unfinished rather than holding the whole round for it.

**Review** with `subagent-review`'s method, over the integration branch's range against the base. That range is what the PR shows, and it is the only range in which the merged result is visible — reviewing the ticket branches one at a time cannot see what putting them together produced. Its **borrowed authority** dimension matters more here than anywhere: it greps the markers across the integrated tree and is the check that every borrowed call actually reached the one table.

**Done when:** each ticket branch is read and gated by you, the integration branch exists and its gate is green, and the review's findings are verified against the tree.

### 5. One PR, one table, one note

**One pull request, opened from the integration branch, however many tickets it carries.** Via `/create-pr-for-branch`.

The body leads with the ledger's table — generated from one grep over the integration branch, never from memory — and the reviewer note verbatim beneath it. Parked calls go under the table as their own list. Then a `Closes #n` line for every ticket the branch actually carries, and a named line for every ticket that did not land, so the round's shape is legible from the PR alone.

**Open it normally, not as a draft.** Review is what this PR wants; the table blocks the merge, not the reading.

**Splitting the round across more than one PR is a decision, not a default.** It needs a reason the human would accept — work that cannot pass the gate together, or a branch that had to be left out — and that reason goes in every PR body and in the handback. Convenience is not one, and neither is "there were several tickets".

**Done when:** one PR exists, its table matches the grep over its branch exactly, every landed ticket is named in it, and nothing has been merged.

### 6. Hand back

The handback shape is in `unattended.md`: what got done, what you borrowed, what you parked, what you could not finish, what you dropped, and the one thing you would ask if you could ask one thing.

Then stop. **Do not wait, poll, or schedule a re-check** — the human is away, and an unattended session that ends is working correctly. A lane's handoff waits come before this step, never after it: see [in a lane](./unattended.md#in-a-lane).

## Where this sits in the flow

`/specs-to-tickets` writes the tickets; **`/implement-unattended`** builds them with nobody watching and stops at one open PR. `/review-pr-in-worktree` judges it when someone is back, `/respond-to-pr-review` is where the table's rows get answered and the markers come out, and `/squash-merge-and-clean-up` lands it.

`/implement-unattended-no-subagents` is this same mode without the fan-out — the right one when the repo restricts subagent use, when the tickets are not independent, or when one ticket is all there is.
