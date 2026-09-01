---
name: approval-policy
description: Where a human's approval gets recorded and what gets written — use whenever a human approves, declines, or decides something in conversation: a name, a scope call, a deviation from a spec, a finding waved off. The tracker holds who decided and why; the tree never holds the discussion.
metadata:
  type: skill
  invocation: model-discoverable
  applies-to: [approvals, decisions, specs, tickets, prs, naming]
---

# Approval Policy

**A decision reached in conversation and written nowhere is a decision that gets made again next week.** This skill says where an approval goes and what it has to say. Reach for it whenever a human settles something mid-work — you do not need to be asked.

**It records approvals; it never manufactures them.** Silence is not approval. Narration the human nodded at is not approval. A previous approval for a similar case is not approval for this one. If you are here because you want to write down an answer you have not actually been given, the thing to do is ask the question.

## What counts

An approval is the human answering something that was theirs to answer. In practice:

- **A gated name** — anything the repo's naming authority says needs their word.
- **A scope call** — a criterion dropped, a slice merged into another, work split into a new ticket.
- **A deviation** — code, spec, or ticket departing from what a spec, ADR, or ticket already says.
- **A finding declined** — a review item, a lint, a warning the human decided not to act on.
- **A choice between real options** — where you offered two and they picked one.

**A decline is an approval.** "We are not doing that, because X" is exactly as load-bearing as a yes, and it is the one that gets lost, because nothing lands in the tree to remind anyone it was decided.

## Where it goes

**Once, where the decision lives.** Not in three places; a decision recorded twice starts contradicting itself the first time one copy is edited.

| The decision is about | Its home |
|---|---|
| One change under review | **The PR body** — or the review reply, where the approval answers a specific finding |
| Something that outlives the change | **The issue** — scope, criteria, a decision the ticket's own text now contradicts |
| A wayfinder decision ticket | **Its resolution comment**, where the answer to that ticket is already recorded |
| An effort, not one ticket | **The map body**, in the section the map keeps for settled decisions |
| A name the repo's authority gates | **Wherever that authority says** — it outranks this table |

**The repo's own authority wins over all of it.** Read its naming-authority and issue-tracker docs before falling back to the table; a repo that has said where approvals are recorded has already made this decision.

**Record it before the thing that depends on it lands** — before the PR merges, before the ticket closes, before the map moves on. Afterwards is a reconstruction, and it shows.

## What to write

Four things, and the first is the one that gets dropped:

1. **Who approved it** — by name. `Approved by <name>` beats "this was approved", which hides the only fact a reader needs.
2. **When** — an absolute date, never "yesterday" or "earlier today".
3. **What was decided** — the answer itself, in one line.
4. **What the alternative was** — what you offered that they turned down. A decision with no visible alternative reads as the only option there ever was, and gets reopened by the next person who thinks of one.

```markdown
`FrameClock` over `PresentationTimer` — approved by Jason Bulson, 2026-09-01.
`PresentationTimer` named the policy rather than the mechanism.
```

## What never gets committed

**The discussion itself stays out of the tree.** Not as a comment in the code, not as a rationale paragraph in a spec, not a notes or findings file, not a "what we considered" section grown in a doc because it came up in conversation. A transcript in a file is stale the day after it is written, and it is read as binding long after it stops being true.

**The tree holds decisions; the tracker holds who decided them and why not the alternative.** The spec says what the rule is. The PR, the issue, or the resolution comment says who settled it, when, and against what.

**Two things that are not exceptions, because they are decision records rather than discussion:**

- **An ADR's considered-alternatives section.** ADRs exist to hold the alternative and the reason; that is the repo's own form for it, filled in as prose about the decision — not as an account of the conversation, and not naming who said what.
- **A one-line rationale in a spec**, where the spec's own shape has a place for it. One line, about the rule. The moment it grows into the argument that produced the rule, it belongs on the tracker.

Everything else — the back-and-forth, the options that died early, the exploration — is conversation. It goes in the PR comment, the issue, or the resolution comment, or it goes nowhere.

## Done when

- Every approval given in the session is written in exactly one home, chosen from the table or from the repo's own authority.
- Each one names who, when, what, and the alternative.
- Nothing about the discussion has been committed to a file — and if something was, it has been moved to the tracker and taken back out.
