---
name: pre-pr-naming-approval
description: Borrow naming approval for 12 hours while the human is away — every borrowed name marked loudly in the tree, and settled with them when they return.
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [naming, specs, commits, prs, sessions]
---

# pre-pr-naming-approval

> **human-only.** Start this only when a human asks for it by name. Granting yourself naming authority is exactly the thing this skill is careful about — if you arrived here from another skill, stop.

A **loan**. The human is stepping away; work that needs a gated name would otherwise stop at the proposal and wait. This lends you their answer for twelve hours so the work continues, on three terms: every borrowed name is **marked in the tree where it lands**, nothing merges while a marker stands, and the human **settles every loan** when they return.

The loan buys motion, never a decision. A name used under it is still open.

## The marker

One line, **verbatim**, in every file that fixes a borrowed name:

```
TEMPORARY AGENT NAMING APPROVAL, IF THIS IS FOUND IN PR REVIEW FLAG AS PROBLEM
```

Character-exact, because it is the **ledger**. `grep -rn` over the working tree enumerates the open loans, which is what makes the settle-up complete and what survives a session that dies mid-loan. Any private notes you keep are a cache of that grep; the tree is the source of truth.

Under it, on its own line, the case:

```
granted <grant>, expires <expiry> — <name> over <alternative>; <the reason>
```

In the file's own comment syntax, at the **declaration** of the name and in the **spec passage** that fixes it. One marker per naming act — a type with its members, an enum with its cases, a section root with its keys — at the site the act happens, not at every use.

## 1. Read the room

**Existing markers mean you are here to settle, not to borrow.** Run the grep first:

```
grep -rn "TEMPORARY AGENT NAMING APPROVAL" .
```

Hits mean a previous window left loans standing: go to section 4 and settle them. A clean tree means a fresh window: continue.

Then find **this repo's** naming authority — it decides which names are even gated, what an acceptable proposal contains, and where an approval gets recorded. It is usually stated inline in `AGENTS.md` or `CLAUDE.md` with a pointer to a fuller page (`docs/`, an ADR). Read the pointer target too: the inline block is the summary, and the exemptions live in the page. Where the repo has no naming authority, there is nothing to borrow — say so and stop.

Stamp the window from the clock, never from memory:

```
date -u +%Y-%m-%dT%H:%MZ
date -u -v+12H +%Y-%m-%dT%H:%MZ   # BSD/macOS; GNU: date -u -d '+12 hours' +%Y-%m-%dT%H:%MZ
```

Report the grant, the expiry, and the authority you found, then get on with the work.

## 2. Borrowing a name

When the work reaches a gated name, **do the whole proposal anyway** — the repo's authority defines its shape, and it is typically the full set, a real alternative, and why. The proposal is the thing the human answers in section 4; skipping it turns the settle-up into a fresh naming session instead of a review.

Then pick the name you would have proposed, write the marker at its sites, and continue.

Two limits on what a loan covers:

- **Gate generously.** A name you are unsure about is borrowed and marked. An unnecessary marker costs one question at settle-up; a missing one costs a rename after review.
- **Adoption is not a loan.** A name taken from a named upstream — another repo, the framework, a wire schema, a third-party API — needs no approval where the repo's authority says so. Cite the source in place, as that authority requires, and leave the marker off.

## 3. What the loan is good for

**Commits, freely.** Note the loan in the commit body; keep it out of the subject, which outlives the branch.

**A PR, reluctantly.** Prefer to leave the work uncommitted-to-GitHub until the loan is settled — a marked PR is a review trap for whoever wanders into it. Where one is genuinely needed, open it **draft**, and lead the body with the marker line and the list of open loans. It becomes mergeable when a later commit removes the markers, which is section 4's job; no marker, no merge.

**Not a merge.** Merging is the human's call regardless.

**At expiry**, the window shuts on *new* loans only. Stop borrowing, keep working on everything else, and leave the marked work standing exactly as it is — unpicking names on your own authority is the same unilateral act in the other direction.

## 4. Settle up

The human is back. Rebuild the case list from the grep, not from memory.

**One case, one question**, with `AskUserQuestion` — the shape the repo's authority asks for, which is generally the set, the alternative, and the reason. Options are the borrowed name (marked as what is in the tree now), the alternative you weighed, and an option to hear more before deciding. Ask about the next case only after this one is answered: a batch of naming questions gets one answer that covers none of them.

Per answer:

1. Where the answer is a different name, rename it everywhere — code, spec, config keys, wire names, tests.
2. Remove that case's marker and its detail line, leaving the surrounding comment or spec passage reading as if the loan never happened.
3. Record the approval where the repo's authority says approvals are recorded — the resolution comment, the PR, the spec.

**The process leaves no trace in the repo.** Issues and PR notes may carry it; a file in the tree may not. Someone reading that spec paragraph next month should find a settled name, not the archaeology of how it got approved.

Done when `grep -rn "TEMPORARY AGENT NAMING APPROVAL" .` returns nothing, every case has an answer behind it, and the removal is committed. Where a PR was opened, that commit is what takes it out of draft.

Close by naming each settled case and its outcome, and anything the human deferred rather than answered — a deferred name is still an open name, and stays marked.
