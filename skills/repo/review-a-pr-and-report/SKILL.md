---
name: review-a-pr-and-report
description: Read a PR's committed diff, run the repo's gate, hold it to its spec and ticket, and report the drift in a fixed shape ending in yes or no. Called by /review-pr-in-worktree; do not reach for it on your own initiative.
argument-hint: "The repo, the PR number, and the head SHA — plus the round number and previous SHA on a re-review"
metadata:
  type: command
  invocation: skill-callable
  applies-to: [prs, github, review, specs, tests, drift]
---

# review-a-pr-and-report

The method of a PR review and the fixed shape of its report. Read what the PR committed, run the repo's own gate against it, hold it to the spec and the ticket that are its true plan, name every form of drift, and answer whether it should be merged.

**Not model-discoverable in practice.** A human may name it, and `/review-pr-in-worktree` calls it. Do not invoke it because a diff happens to be in front of you — an unasked-for review is noise, and a review run outside a prepared checkout reviews the wrong thing.

**This skill never writes.** No commits, no fixes, no pushes, no PR comments, no merge — even when the fix is one line and obvious. A review that edits its subject stops being a review. Findings go to the human, who decides what happens next.

## What the caller owns, and what this skill needs

The caller prepares the ground; this skill reads it. Before step 1, you must have:

| | |
|---|---|
| **The repo and the PR number** | Exactly one of each, settled — never guessed from "the newest" or "the only open one" |
| **The head SHA** | The commit under review. Everything below judges *that* commit, not whatever the branch points at ten minutes from now |
| **The base ref** | What the PR merges into |
| **A checkout at the head SHA** | Clean, and not the primary checkout if other sessions share it. Every command below runs there |
| **What has already been said** | Existing reviews and comments — a point a human already made is theirs, not a fresh finding |
| **The round** | `1`, or the round number and the previous round's head SHA — see [Rounds](#rounds) |

If the caller supplied these, take them. If a human invoked this skill directly, get them yourself before reading a line of diff:

- `gh pr view <n> --json number,title,body,state,author,headRefName,baseRefName,headRefOid,files,commits,closingIssuesReferences`
- `gh pr checks <n>` — a Markdown-only PR may report no checks; that is a pass, not a failure
- `gh pr view <n> --json reviews,comments`

The caller also owns everything *after* the report: whether to review another round, whether to clean up, and whether the PR ever gets merged. This skill ends at the report.

## 1. Review what was committed — not the working tree

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
- **Anything untracked in the checkout is a finding** — it was clean, so a file that appears is build output that wants ignoring, or a scratch file that should never have been in the tree.
- **Look for what should have changed and didn't** — a renamed concept the call sites still use by its old name, a new branch of behaviour with no error path, a config key added in one environment file and not its siblings.

**Done when:** you have read every changed file that matters and can say what the PR does in your own words, without quoting its description.

## 2. Run the tests — the repo's own gate, in the checkout

Find the gating command; do not invent one. In order: the repo's `AGENTS_REPO.md` or `AGENTS.md`, its CI workflow, then its build file (`prepublishOnly`, a test script, the .NET or language-native equivalent). If nothing documents a gate, say so under **What I could not verify** and treat the PR as **unverified** rather than guessing at a command and reporting its output as meaningful.

- Run it **in the review checkout**, never in the primary one.
- **Run the whole gate in one call.** The default is the widest run the gate supports — the full suite, one invocation. A pass per test class turns a two-minute gate into twenty minutes of tool calls, and every one of them costs a round trip whether it finds anything or not.
- **Split only when one call genuinely cannot do it**, and then split as coarsely as it allows. Two reasons qualify, and only these: the run outlasts the longest timeout the tool will take — raise it first, a slow gate is a long call, not many short ones — or the repo says this suite must not run all at once. Where the runner takes a repeatable filter (`-class`, `--filter`, a project list), pack as many into each call as it will hold — the limit is the length of a call, not the number of classes in it. Name the split and its reason in the report.
- **Narrow runs are for re-checking, not for the first pass.** Once the gate has run and something failed, a single class or test is the right instrument for confirming a diagnosis, or for checking one thing you doubt. Going narrow before there is a failure is guessing where one might be, and it is how a gate ends up run twenty times and never run whole.
- Respect the repo's stated way of running its own suite. Where a suite needs exclusive access to something shared — containers, ports, a daemon, a database — a concurrent run in another session produces failures that belong to the collision, not the code. A repo that says to run such a suite serially has named its own limit; that is the second of the two reasons to split, and it goes in the report.
- **Report the output as it came.** A failing test is a finding, not a problem to fix and move past. Quote the failure.
- **A green suite is not automatically a pass.** New behaviour with no new test is a finding of its own; so is a test that was changed in this PR to accommodate the code.

**Done when:** you can state, with output to back it, that the gate passes, fails with named failures, or was exempt or unrunnable and why.

### The exemption: a Markdown-only PR has no gate to run

Test this **before** running anything, because the answer decides whether you run the suite at all:

```
git diff --name-only <merge-base>...<head-sha>
```

**The test is the file extension, and nothing else: does every changed path end in `.md`?** One path that does not — a `.cs`, a `.csproj`, a `.json`, a workflow, a lockfile, an image, a `.txt` — and the gate above is required exactly as written. Not the directory it sits in, not "it is really only docs", not a config change the author calls trivial. A rule that can be argued with is not a gate.

Where every path is Markdown:

- **Do not run the suite.** It cannot be exercising this diff, and a green run quoted in the report would offer evidence the review does not actually have.
- Record it in **Test Coverage** in one line — `No gate run: all N changed paths are Markdown` — and name them or their count.
- **CI reporting no checks is a pass, not a gap.** A check that did run and failed still gets reported, but say plainly whether this diff could have caused it. A red gate that a Markdown-only diff cannot reach belongs to the base branch, and it does not block this PR.
- **The verdict is not blocked by the absent gate.** This is the one case where a PR with no passing suite behind it can still be `yes`.

**Nothing else relaxes.** Prose is what this repo's specs are made of, so a Markdown-only PR is *more* exposed to the findings that matter here, not less: a spec backfilled to describe code built elsewhere, a doc that now asserts something untrue, a term that contradicts the glossary, a criterion the ticket asked for and the prose does not deliver. Steps 3 and 4 run in full, and every one of their findings still blocks.

## 3. Read the true plan, and hold the code to it

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

## 4. Hunt the drift

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

## 5. Report — these sections, in this order, every time

The report is **fixed**. Same headings, same order, every PR, so a human reading their fifth review of the week knows where each answer lives without hunting for it. A section with nothing to say says so in one line; it is never dropped, and nothing is added between the ones below — save the one re-review section [Rounds](#rounds) defines.

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
<The gate command and what it actually returned — failures quoted, not summarised — plus how it was run if it had to be split across calls, and why. Then what the new behaviour's tests cover, what they leave uncovered, and any test this PR changed to fit the code. For a Markdown-only PR: the one line from step 2's exemption, and what CI reported.>

### Drift
<The forms from step 4 that fired, one line each, each citing its authority. "None found" if none did.>

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

**A PR whose gate could not be run cannot get a `yes`** — unverified is not the same as passing, and the bullet says so. **The single exception is the Markdown-only PR of step 2**, which has no gate to fail: there, the absent suite is not a bullet, and the verdict rests entirely on spec compliance and drift.

Throughout: refer to files as clickable paths with line numbers, and to issues and PRs by title and link, never a bare `#42`. Where the answer is `yes`, resist padding the sections above to look thorough — a review that manufactures findings costs more than it returns.

**Done when:** every heading above is present and filled, and the last one reads either exactly `yes` or `no` with its bullets.

## Rounds

A review that finds something usually gets answered — the author pushes a fix to the **same PR**. The caller decides when that happens and puts the checkout on the new head SHA; this skill then runs steps 1 through 5 again, with three differences:

- **Diff the rounds, not just the PR.** `git diff <previous-head>..<new-head>` is what the author actually did in response; the full `<merge-base>...<new-head>` is still what gets judged. Read both. A fix that also quietly changes something the last round approved is the thing this catches.
- **Re-run the gate. Every time.** A pass from the previous round belongs to the previous commit and carries nothing forward. A Markdown-only exemption is re-tested too — the new round may have added code.
- **Account for every bullet.** Each reason from the last round's `no` gets a verdict: fixed, not fixed, or fixed in a way that broke something else.

The report keeps its fixed sections, with **one** permitted addition on rounds after the first: a `### Since the last review` section immediately after the fact line, one line per prior bullet. The fact line names the round and the previous SHA.

**If the head SHA has not moved, say so and stop** — re-reading the same commit produces a second opinion on nothing and reads as though something changed.

## Cost

Reading the changed files whole and reading the ticket and specs is the bulk of the work, and it is the necessary cost — drift is exactly what a diff-only pass cannot see. What to skip: unchanged files that nothing in the diff touches, the full history of the base branch, and any file already covered by a human reviewer's comment the caller handed you.

## Where this sits in the flow

`/review-pr-in-worktree` is the door a human opens: it settles which PR, checks it out into a worktree of its own, calls **`/review-a-pr-and-report`** for each round, and cleans up after. This skill is the judgment inside that — the part that would be the same wherever the checkout came from. It reads and reports; it never merges, and it never says what happens next.
