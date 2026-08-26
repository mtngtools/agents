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

**This skill never writes.** No commits, no fixes, no pushes, no PR comments, no merge — even when the fix is one line and obvious. A review that edits its subject stops being a review. Findings go to the human, who decides what happens next. `/squash-merge-and-clean-up` merges; this skill only ever says whether it should.

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
- `gh pr checks <n>` — a docs-only PR may report no checks; that is a pass, not a failure
- `gh pr view <n> --json reviews,comments` — a point already made by a human is theirs, not a fresh finding

Hold the **head SHA**. Everything below is a review of that commit, not of whatever the branch points at ten minutes from now.

**Done when:** you know what the PR claims to do, which issue it closes, what CI thinks, and what has already been said.

### 3. Build the review worktree, detached at the head SHA

```
git fetch origin pull/<n>/head
git worktree add <path> --detach <head-sha>
```

Then enter it — `EnterWorktree` with `path` where available, which requires the worktree to already appear in `git worktree list`, hence the order above.

**Detached on purpose.** No branch is created, so nothing here can be pushed by accident and nothing collides with the author's own checkout of the same branch. Put the worktree somewhere clearly disposable and named for the PR.

Some harnesses restrict shell redirection inside a worktree. If a heredoc or `>` is refused, use file-writing tools and plain single-purpose commands instead of fighting it.

**Done when:** the session is in a clean worktree whose `HEAD` is the head SHA from step 2.

### 4. Review what was committed — not the working tree

The subject of the review is the committed diff against the merge base:

```
git merge-base origin/<base> <head-sha>
git diff --stat <merge-base>...<head-sha>
git diff <merge-base>...<head-sha>
git log --oneline <merge-base>..<head-sha>
```

Three dots, not two: two-dot would fold in whatever the base branch has done since, and blame the author for it.

- **Read the changed files whole**, not only the hunks, wherever the change is more than cosmetic. A diff shows what moved; it hides what the file now says.
- **Read the comments as claims about the code as it now stands** — every doc comment, inline note, `TODO`, and example snippet in a changed file asserts something. A comment this diff falsified is a finding: it outlives the rename, the extracted method, the removed branch, and the next reader believes it long after the code stopped matching. Include `TODO`s the PR itself completed and examples that would no longer compile.
- **Read the commits as well as the diff** — messages against the repo's convention (`type(scope): subject (#issue)`), no merge commits where the repo squashes, no commit that undoes an earlier one in the same PR without saying why.
- **Anything untracked in this worktree is a finding** — the checkout was clean, so a file that appears is build output that wants ignoring, or a scratch file that should never have been in the tree.
- **Look for what should have changed and didn't** — a renamed concept the call sites still use by its old name, a new branch of behaviour with no error path, a config key added in one environment file and not its siblings.

**Done when:** you have read every changed file that matters and can say what the PR does in your own words, without quoting its description.

### 5. Run the tests — the repo's own gate, in the worktree

Find the gating command; do not invent one. In order: the repo's `AGENTS_REPO.md` or `AGENTS.md`, its CI workflow, then its build file (`prepublishOnly`, a test script, the .NET or language-native equivalent). If nothing documents a gate, say so under **What I could not verify** and treat the PR as **unverified** rather than guessing at a command and reporting its output as meaningful.

- Run it **in the review worktree**, never in the primary checkout.
- Respect the repo's stated way of running its own suite. Where a suite needs exclusive access to something shared — containers, ports, a daemon, a database — a concurrent run in another session produces failures that belong to the collision, not the code. If the repo says to run such a suite serially or per test class, do that, and say so in the report.
- **Report the output as it came.** A failing test is a finding, not a problem to fix and move past. Quote the failure.
- **A green suite is not automatically a pass.** New behaviour with no new test is a finding of its own; so is a test that was changed in this PR to accommodate the code.

**Done when:** you can state, with output to back it, that the gate passes, fails with named failures, or could not be run and why.

### 6. Read the true plan, and hold the code to it

The PR description is the author's account of the work. It is not the plan. The plan lives upstream of it, and where they disagree the upstream wins:

| Authority | What it settles |
|---|---|
| **Spec / ADR** in the repo | The rule. Highest authority; the code implements it, never the reverse |
| **The ticket** the PR closes | This slice of it — scope and acceptance criteria |
| **The PR description** | Intent only — evidence of what the author meant, not of what was agreed |
| **Doc comments in the code** | A claim about that file, made by the same change under review — check it for truth, never treat it as a repo rule; the spec settles rules |

Fetch the closing issue (`gh issue view <n>`, plus its parent map or epic if the tracker links one) and open the spec files the ticket names. Then walk the ticket's acceptance criteria one at a time and mark each **delivered**, **missing**, or **not verifiable from the diff** — one line each, pointing at the file that satisfies it.

**If the PR edits a spec, separate that from the code changes and ask which moved first.** A spec edited to describe what was built is a decision being made silently in a build ticket. It may be the right call, but it needs the human's eyes, so it is always a finding unless the ticket explicitly asked for the spec change.

**Done when:** every acceptance criterion has a verdict, and every spec edit in the diff is accounted for.

### 7. Hunt the drift

Drift is the gap between what was agreed and what landed. Walk these deliberately — most of them are invisible if you only read the diff against the description:

| Form | The question | Where it shows |
|---|---|---|
| **Scope drift** | Does the diff do more, or less, than the ticket asked? | Files no criterion accounts for; criteria no file satisfies |
| **Spec drift** | Does the code contradict a spec or ADR? | Behaviour, names, defaults, error handling |
| **Backfilled spec** | Was the spec moved to match the code? | Spec edits in a build PR |
| **Plan drift** | Different approach than the one that was settled, undeclared | An ADR chose X; the code does Y and says nothing |
| **Test drift** | Were tests bent to fit the code? | Weakened assertions, skips, deletions, renames |
| **Doc drift** | Does prose now assert something untrue? | README and AGENTS files; comments in the changed files, and comments elsewhere this change quietly falsified |
| **Vocabulary drift** | Do new names match the domain language? | A new term for a concept the glossary already names |
| **Placement drift** | Does a file sit where the repo's own taxonomy says it belongs? | A dependency pointing the wrong way across a layer |
| **Base drift** | Is the review aimed at a moved target? | Merge base far behind `origin/<base>`; conflicts pending |
| **Residue** | What was left behind? | Commented-out code, debug logging, stray TODOs, dead additions |

**Stale comments hide in the files the PR never opened.** A rename, a changed default, a removed branch or an inverted flag falsifies prose the diff cannot show you. Grep the old name and the old behaviour's vocabulary across the tree, and check what the comments at the call sites still promise. Cite such a finding to the line in the diff that made it false, not to the untouched file's age — the comment was true until this PR, and it is this PR's to fix.

**Every finding cites its authority.** Name the spec, ticket, ADR, or repo standard the code contradicts, and quote the line. Anything you cannot anchor that way is a **preference**, not a finding: it belongs under **Additional Consideration**, never under **Drift**. A review that mixes the two teaches the human to discount all of it.

**Done when:** each form above has been considered and either produced a finding or been ruled out.

### 8. Report — these sections, in this order, every time

The report is **fixed**. Same headings, same order, every PR, so a human reading their fifth review of the week knows where each answer lives without hunting for it. A section with nothing to say says so in one line; it is never dropped, and nothing is added between the ones below — save the one re-review section step 9 defines.

```markdown
## PR #<n> — <title>

Round <n>, reviewed at `<head-sha>` <(previously `<sha>`), on rounds after the first> against `<base>` (merge base `<sha>`) · Plan: <issue title (#n)> · Specs read: <files> · Gate: `<command>`

### High-Level Overview
<Two to four sentences: what this PR does and why, in your words. Not a paraphrase of its description — if the two disagree, this section is where that shows.>

### More Detailed Summary
<The change, file group by file group: what each does, what it replaces, how the pieces connect. Enough that the human need not open the diff to follow the rest of the report.>

### Spec Compliance
<One line per acceptance criterion — delivered / missing / not verifiable — each pointing at the file that satisfies it. Then every place the code contradicts a spec or ADR, quoting the line it contradicts. Then every spec edit in the diff, and whether the ticket asked for it.>

### Test Coverage
<The gate command and what it actually returned — failures quoted, not summarised. Then what the new behaviour's tests cover, what they leave uncovered, and any test this PR changed to fit the code.>

### Drift
<The forms from step 7 that fired, one line each, each citing its authority. "None found" if none did.>

### Additional Consideration
<Worth the author's attention but backed by no spec, ticket, or standard. Preferences, explicitly labelled as such, and non-blocking by definition.>

### What I could not verify
<A gate that would not run, criteria invisible from the diff, files deliberately skipped and why. "Nothing — the review was complete" if that is true.>

### Should this be merged
<yes | no, plus bullets>
```

**The last section takes one of two shapes and nothing in between.**

A clean pass is the single word **`yes`**. No caveat appended, no "yes, but", no praise. It is one word because it is unambiguous, and because a hedge attached to an approval makes the human do the reviewer's job of deciding whether the hedge matters.

Anything else is **`no`**, followed by bullet points — one per reason, each naming what must change, where, and which authority demands it. If you catch yourself wanting to write "yes with notes", it is a **no**: move the notes into the bullets. A reservation big enough to qualify the verdict is big enough to block it; one that isn't belongs in **Additional Consideration**, where it costs the author nothing.

**A PR whose gate could not be run cannot get a `yes`.** Unverified is not the same as passing, and the bullet says so.

Throughout: refer to files as clickable paths with line numbers, and to issues and PRs by title and link, never a bare `#42`. Where the answer is `yes`, resist padding the sections above to look thorough — a review that manufactures findings costs more than it returns.

**Done when:** every heading above is present and filled, and the last one reads either exactly `yes` or `no` with its bullets.

### 9. Gate: review again, or clean up

A review that finds something usually gets answered — the author pushes a fix to the **same PR**, and the same worktree is the cheapest place to look at it. So the review does not end at the report; it parks there and asks what comes next. Put it to the human with `AskUserQuestion`, one question, these two options:

> The review is reported. What next?

- **Review again** — changes have been pushed to the PR
- **Clean up** — remove the worktree and everything this session made for this review

`AskUserQuestion` supplies its own free-text **Other** alongside them; do not add a third option for it. Take the answer at face value and do only what it asks.

**Stay parked between rounds.** Waiting is the correct state — do not clean up because the report is written, do not poll the PR for new commits, and do not start reviewing again until asked. The worktree costs nothing sitting still.

#### On "review again"

```
git fetch origin pull/<n>/head
```

Read the new head SHA. **If it has not moved, say so and ask again** — re-reading the same commit produces a second opinion on nothing and reads as though something changed.

If it has moved, `git checkout <new-head-sha>` in the worktree — still detached — and re-run steps 4 through 8, with three differences:

- **Diff the rounds, not just the PR.** `git diff <previous-head>..<new-head>` is what the author actually did in response; the full `<merge-base>...<new-head>` is still what gets judged. Read both. A fix that also quietly changes something the last round approved is the thing this catches.
- **Re-run the gate. Every time.** A pass from the previous round belongs to the previous commit and carries nothing forward.
- **Account for every bullet.** Each reason from the last round's `no` gets a verdict: fixed, not fixed, or fixed in a way that broke something else.

The report keeps its fixed sections, with **one** permitted addition on rounds after the first: a `### Since the last review` section immediately after the fact line, one line per prior bullet. The fact line names the round and the previous SHA. Then this gate again — rounds repeat until the human ends them.

#### On "clean up"

Remove what **this session** made for **this review**, and nothing else. Other sessions have worktrees and branches in the same listings; a `locked` entry or a directory this session did not create is someone's work in progress.

1. `ExitWorktree` with `action: "keep"` — it will not remove a worktree entered by path.
2. `git worktree remove <path>`, then `git worktree prune`.
3. Any scratch, log, or report file this review wrote outside the worktree.
4. Anything the gate left running — containers, a daemon, a stray port — that this session started. Test suites are the usual source, and they do not always clean up after themselves.

**No branch is deleted here, ever.** The head branch belongs to the PR and its author, and the review worktree was detached precisely so this step has no branch of its own to consider. Deleting branches is `/squash-merge-and-clean-up`'s job, after a merge this skill does not perform.

**Done when:** the human has ended the rounds and either the worktree is gone with its residue, or it is standing on purpose and they know where it is.

### 10. Close by confirming what happened to the PR

The last thing to establish is whether the thing you reviewed actually landed — usually it was merged elsewhere, by the human or by another session running `/squash-merge-and-clean-up`, and this session would otherwise never learn the outcome of its own review.

```
gh pr view <n> --json state,mergedAt,mergeCommit,headRefOid
```

Report one of:

- **Merged** — give the merge commit, and say whether `headRefOid` is the SHA you last reviewed. If the PR moved after your final round, the review does not cover what merged; say that plainly.
- **Still open** — say so, name who is expected to land it, and do not offer to merge it yourself.
- **Closed unmerged** — say so. The review stands as the record of why, if that is why.

Check once and report. **Do not poll, wait, or schedule a re-check** — an open PR is a complete, correct ending for this skill.

## Cost

Reading the changed files whole and reading the ticket and specs is the bulk of the work, and it is the necessary cost — drift is exactly what a diff-only pass cannot see. What to skip: unchanged files that nothing in the diff touches, the full history of the base branch, and any file already covered by a human reviewer's comment from step 2.

## Where this sits in the flow

`/create-pr-for-branch` opens it, **`/review-pr-in-worktree`** judges it — as many rounds as the author needs — and `/squash-merge-and-clean-up` lands it. Each is a separate door a human opens, because each answers to a different person: the author, the reviewer, the maintainer. The merge usually happens in another session entirely, which is why this skill ends by asking the tracker what became of the PR rather than assuming its own report was the last word.
