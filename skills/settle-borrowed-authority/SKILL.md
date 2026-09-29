---
name: settle-borrowed-authority
description: Walk the human through every call borrowed under a grant — the names in one pass, every other call case by case — then record the approvals, take the markers out, and leave the PR mergeable.
argument-hint: "The PR (number or URL) — omit when this session already knows it"
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [approvals, naming, decisions, prs, git, sessions]
---

# settle-borrowed-authority

> **human-only.** Start this only when a human asks for it by name. Every case below ends in a human answer, and a session running this without one in the room would be approving its own loans — the exact thing the markers exist to prevent.

The close-out of the grant skills. `grant-naming-authority`, `grant-decision-authority`, and both `implement-unattended` modes leave loans standing — markers in the tree, a table on the PR, parked calls beneath it. This skill is the human back at the keyboard, settling them: every open call answered, recorded, and its marker removed, until nothing blocks the merge but the human's own decision to merge.

**How a loan is held and settled is the [borrowed-authority ledger](../grant-decision-authority/ledger.md).** Read it first. This skill adds the session around its settle-up: finding the PR, the naming pass, the parked calls, and reflecting the answers everywhere they need to land. It works the same from the session that built the PR — which already knows it — or cold, handed nothing but a PR number.

## Process

### 1. Find the PR and rebuild the cases

Three inputs, in whatever order they arrive:

- **The PR.** From the argument if one was given; otherwise the PR this session opened or has been working on. Neither → ask. `gh pr view <n> --json number,title,body,state,headRefName,headRefOid,comments` gets the table, the parked list, and any answers already given on it.
- **A checkout of the PR branch.** The branch's own worktree where this session built it there; otherwise make one, the way the `implement` skill's step 2 does — never the primary checkout, which other sessions share.
- **The case list, rebuilt from the tree, never from memory.** `grep -rn "TEMPORARY AGENT" .` over the head commit is the marked loans. The PR body's parked list is the rest — invisible to the grep, because nothing was written for them.

Cross-check grep against table before asking anything:

| Mismatch | What it means |
|---|---|
| Marker with no row | A borrowed call that never reached the PR — add its row from the marker's own detail line, and say so |
| Row with no marker | Settled in an earlier round, or never marked at all — the PR's commits and comments say which |
| Row already answered on the PR | Settled. It carries into step 4's record, not into the questions |

Number the cases — kind, the call, the alternative, the site — and show the human the list, settled rows marked settled, parked calls numbered at the end. This list is what makes step 2's single pass an informed answer rather than a blind one.

**Done when:** the human has seen one numbered list holding every open marker and every parked call, with every mismatch stated on it.

### 2. Settle the names in one pass

**Names are settled together by default.** A set of names is judged partly as a set: whether they read as one vocabulary is half of whether each one is right. Show every open naming row in one table, with #, the name, the alternative weighed, why this one, and the site. Then ask one `AskUserQuestion`:

- **Approve all names as borrowed:** every naming row is accepted exactly as it stands in the tree.
- **One by one:** each naming row gets its own question, the way decisions do in step 3.
- The harness adds **Other** on its own. Answers like "all but #3" and "#2 should be `FooClock`" land there. Take them at their word: apply a replacement the human stated as that row's answer, and ask any row they carved out as its own question.

If the cross-check flagged a naming row, leave it out of the pass and ask it on its own afterwards. After an approve-all, say it back in one line, for example "all N names approved as borrowed", so step 4's record rests on something the human did, not on an inference. With no open naming rows, skip this step.

**Done when:** every naming row has a human answer, either from the pass or from its own question.

### 3. Settle everything else case by case

**Decisions, one at a time:** this is the ledger's settle-up, with an example added to every case. One case, one question, in list order, and the next only after this one is answered. Present the case first, short enough to hold in one glance: what was chosen, the real alternative weighed, why, and where it lands.

**Then give an example before asking.** Pick one concrete situation (an input, an event, a caller) and say what happens in it under the borrowed call and what happens under the alternative. Choose a situation where the two differ; one where they behave the same shows nothing.

Then ask the question with these options: **keep the borrowed call**, named as what is in the tree now; **switch to the alternative**; **explain further with another example (if possible)**. "Explain further" gives the fuller story (call sites, the spec passage it touches, what each choice costs downstream) and a second, different situation. If there is no second situation that tells the two apart, say so rather than repeat the first. Then ask the same question again.

**Every code case carries its throw verdict.** Ask no case whose call is code without it; this covers every decision case except those that change only names or only prose. Before presenting the case, invoke the `can-this-throw` skill over the case's site and the code its call produced, and open the case with its verdict line and its paths. If the verdict is (b) or (c), say so plainly before the options.

**The human can still approve the rest at once.** If they ask to approve the remaining decision rows as borrowed, whether up front or in a case's **Other**, do it only when the cross-check was clean and only after showing every remaining code case's throw verdict in one list. The approve-all stands once the human has seen that list. Then say it back in one line. That's the human choosing not to be walked through the calls, which is theirs to choose. The ledger's warning about batching is about *you* grouping questions.

**Parked calls, always one at a time:** the question is the parked question itself. Give what you would have chosen, the other reading you saw, and an example of each in the same situation, then offer **explain further with another example (if possible)** as an option. A parked call the human **defers** stays parked — it keeps its place in the PR body, and it keeps blocking whatever it blocks. No approve-all reaches a parked call: it has no chosen answer to accept, since it was parked because it was never safe to make alone. A parked call about code gets a throw answer too: for each option, `can-this-throw`'s verdict as far as the code in the tree can show it. Mark it as a forecast, since nothing was written.

Apply each answer before asking the next case, per the ledger: change what the answer changed — everywhere, not just at the marker — remove that case's marker and detail line, and note who answered, the date, and the alternative for step 4's record.

**Done when:** every case has a human answer or an explicit deferral, every non-naming case was asked with an example in front of it, every code case was answered with its throw verdict in view, and the tree holds markers only for the deferrals.

### 4. Reflect it everywhere it needs to land

Four homes, in this order:

- **The tree.** Markers out — deferred ones stay — differing answers applied everywhere, and the repo's gate green over the branch: an approval reflected on a red gate is not reflected. New commits on the PR branch, never a rewrite of the head a review already judged, pushed.
- **The PR body.** Replace the table with the approvals it produced, per the `approval-policy` skill — who approved, the absolute date, what, and the alternative, one line per case. Deferred cases keep their rows under a heading that says they still block the merge; answered parked calls move from the parked list into the approvals.
- **The reviewer note.** It stays while any row still stands; it comes out with the last one.
- **Wherever the repo's own authority routes an approval.** A naming authority that records approvals in a resolution comment or a spec outranks the PR-body default — `approval-policy` says how to find out.

**Done when:** `grep -rn "TEMPORARY AGENT" .` over the pushed head returns only the deferrals — or nothing — every answered case has its approval recorded in exactly one home, and the leave-no-trace rule holds: the passages the markers left read as if the loan never happened.

### 5. Close

Name each case and its outcome — kept, switched, deferred — and say plainly whether the merge is now open. The merge itself stays the human's, through `/squash-merge-and-clean-up`.

## Where this sits in the flow

The grant skills and both `implement-unattended` modes open the loans; `/review-pr-in-worktree` flags their markers; `/respond-to-pr-review` settles them when the answers arrive inside a review response. **This is the settle-up as its own act** — no review round in hand, just a PR blocked on its table and a human ready to answer it — from the session that built it or from one that has only its number.
