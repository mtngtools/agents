# The borrowed-authority ledger

Shared by the two grant skills — `grant-naming-authority` and `grant-decision-authority` — and by the two `implement-unattended` modes, which carry the same grants. Each names **what** may be borrowed. This file is **how** a loan is held: marked in the tree, tabled on the PR, gated at the merge, and settled by a human.

## A loan buys motion, never a decision

The human is not here, and work that needs their word would otherwise stop at the proposal and wait. A loan lets the work continue on the answer you would have proposed. That answer is still **open**. It becomes a decision when a human says so, on the PR, before the merge — not when you write it, not when the tests pass, and not because nobody objected.

Three terms hold the whole thing together:

1. **Every borrowed call is marked in the tree, where it lands.**
2. **The PR carries a table of them and a note telling reviewers what to look for.**
3. **Nothing merges while a marker stands.**

## The markers

One line, **verbatim**, in every file that fixes a borrowed call:

```
TEMPORARY AGENT NAMING APPROVAL, IF THIS IS FOUND IN PR REVIEW FLAG AS PROBLEM
```

```
TEMPORARY AGENT DECISION APPROVAL, IF THIS IS FOUND IN PR REVIEW FLAG AS PROBLEM
```

Character-exact, because the tree is the **ledger**. `TEMPORARY AGENT` is the prefix both share, so one command enumerates every open loan of either kind:

```
grep -rn "TEMPORARY AGENT" .
```

That grep is the source of truth. The PR table is generated from it, and any notes you keep are a cache of it. This is what survives a session that dies mid-loan, and it is why the table can be rebuilt correctly by a session that was never there for the work.

Under the marker, on its own line, the case:

```
<name> over <alternative> — <the reason>
```

```
<the call> over <alternative> — <the reason>
```

In the file's own comment syntax, at the **site of the act** — the declaration of the name, the branch the decision produced, the spec passage that fixes it. One marker per act: a type with its members, an enum with its cases, a retry policy with its bounds. Not at every use.

## Read the room before borrowing anything

**Existing markers mean you are here to settle, not to borrow.** Run the grep first. Hits mean a previous window left loans standing — settle those before opening new ones, or you will hand the human a mixed pile and no way to tell which session made which call.

A clean tree means a fresh window.

## The PR table

Generate it from the grep, never from memory. It goes in the **PR body**, under its own heading, and it is the first thing in the body after the summary:

```markdown
## Temporary agent authority — approval required before merge

A human granted temporary authority for the calls below. **They are not approved.**
Each was chosen by an agent, marked in the tree, and is open until a human answers it here.

| # | Kind | The call | Alternative weighed | Why this one | Site |
|---|---|---|---|---|---|
| 1 | naming | `FrameClock` | `PresentationTimer` | names the mechanism, not the policy | `src/.../FrameClock.cs:14` |
| 2 | decision | retry 3× with backoff | fail fast | transient socket errors dominate the failure mode | `src/.../Transport.cs:88` |

**To approve:** reply in a comment, or edit this section, naming which rows you accept and
which you want changed. A row with no human answer blocks the merge.
```

Every row must carry a **real alternative** and a **reason**. A row whose alternative column reads "none" is not a proposal — it is an assertion that no choice existed, and it will be treated as one.

## The note for reviewing agents

Directly under the table, verbatim, so a reviewing agent reaches the right conclusion without having to infer it:

```markdown
> **Reviewers, human and agent:** this PR was written under temporary agent authority.
> The table above is the complete list of calls no human has approved. Check for a
> follow-up approval from a human in this PR's body or comments before treating any row
> as settled, and check that `grep -rn "TEMPORARY AGENT" .` over the head commit returns
> exactly the rows above. A marker in the diff is expected here, not a defect — an
> **unlisted** marker, or a listed row with no human answer, is the finding.
```

**Open the PR normally, not as a draft.** Review is exactly what this PR wants; the table blocks the merge, not the reading. A draft PR gets skipped by the people whose answers you need.

## What the loan is good for

- **Commits, freely.** Note the loan in the commit body; keep it out of the subject, which outlives the branch.
- **A PR, on the terms above** — table, note, markers in the diff.
- **Not a merge.** Merging is the human's call regardless, and here it is also gated: no marker may survive it.

## Settle up

The human is back, or has answered on the PR. Rebuild the case list from the grep, not from memory.

Run as its own session, this is the `settle-borrowed-authority` skill — it adds finding the PR, a pace question, and the parked calls around what follows. Inside a review response it is `respond-to-pr-review`'s recording step. Either way, the mechanics below are the mechanics.

**One case, one question**, with `AskUserQuestion` where your harness has it. Options are the borrowed call — marked as what is in the tree now — the alternative you weighed, and an option to hear more before deciding. Ask about the next case only after this one is answered: a batch of these gets one answer that covers none of them.

Where the human has already answered rows on the PR, those rows are settled; ask only about what they did not reach.

Per answer:

1. Where the answer differs from what you borrowed, change it everywhere — code, spec, config keys, wire names, tests, and the PR table row.
2. Remove that case's marker and its detail line, leaving the surrounding comment or spec passage reading as if the loan never happened.
3. Record the approval per the `approval-policy` skill — who, when, what, and the alternative — in the one home that policy names.

**Done when** `grep -rn "TEMPORARY AGENT" .` returns nothing, every row has a human answer behind it, the removal is committed and pushed, and the PR table has been replaced by the approvals it produced.

Close by naming each settled case and its outcome, and anything the human **deferred** rather than answered. A deferred call is still an open call: it keeps its marker, keeps its row, and keeps the merge shut.

## Leave no trace

**The process stays out of the tree.** The markers come out. Nothing replaces them — not a comment explaining what was once borrowed, not a rationale paragraph in a spec, not a "what we considered" section. Someone reading that passage next month should find a settled decision, not the archaeology of how it got approved.

The PR and the issue may carry the history. A file in the tree may not.
