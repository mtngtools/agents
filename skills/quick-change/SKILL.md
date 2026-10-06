---
name: quick-change
description: Make a small, already-decided change the short way — an issue, a worktree off origin/main, spec and tests kept in line, a PR — and send anything foggier down the wayfinder pipeline.
argument-hint: "The change to make, or the issue that holds it"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [building, issues, worktrees, specs, tests, prs]
---

# Quick change

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

The short route to the long route's ending: an issue, a branch off `origin/main` in a worktree of its own, spec and tests in line with the code, and a PR. It skips the map, the decision tickets and the slicing, and earns that only while the change stays **quick**: small and already decided. The long route lives in the `describe-wayfinder-pipeline` skill.

**Every repo change happens inside the worktree step 4 creates.** Steps 1–3 only read the tree and write to the tracker. Other sessions share the primary checkout, so an edit made there lands in their build, or on a branch swapped out from under you. An edit that seems to want making sooner (a quick experiment to size the change, a fix spotted while reading) waits for the worktree.

## Process

### 1. Size it

Invoke `describe-wayfinder-pipeline` and apply its **Which route** test. Read the code and spec the change lands in first; the test sizes the change, not the request's wording.

- **Quick:** say so in one line, naming why, and go on.
- **Long:** stop. Name the signal that tripped and the door the pipeline gives, as a line the human can type. Then ask one question: take the long route, or go quick anyway. A "go quick" is theirs to give, and it goes on the issue in step 3.

**Done when:** the change is sized in one line, and either it is quick or the human chose quick anyway.

### 2. Confirm the ending

Skip any question the request already answers. Ask each on its own:

- **Where does it end?** A PR against `main` (the usual answer), a pushed branch with no PR, or a local commit only.
- **Review before the PR?** Yes runs `/code-review` at step 7; the human saying yes authorizes its subagents. No means the human reviews the PR separately once it is open.

**Done when:** the ending and the review answer are both in hand.

### 3. Hold it on an issue

**Named an issue:** read it; it is the scope.

**No issue:** as soon as the basic scope is clear (what changes, where, what done looks like), propose one and ask to create it: a title, and a body with the problem, the change, and acceptance criteria. Create it on a yes, following the repo's issue-tracker doc. If they decline, the PR body is the record and the branch drops the number.

**Keep it current.** The issue is where this change's story lives, so it tracks what you find:

- Scope moves (an extra file, a criterion added or dropped): edit the body.
- A human decides something (a name, a call, "go quick anyway"): a comment, per `approval-policy`.

**Done when:** an issue number is in hand (or the human declined one) and its body states the scope as you understand it now.

### 4. Base it and branch into a worktree

```
git fetch origin
git worktree list
gh pr list --state open
```

**The base is the freshly fetched `origin/main`**: never the local `main`, never the primary checkout's `HEAD`. A base the human named overrides it. If an open PR or another worktree touches the same files, say so and ask once: branch from `origin/main` anyway, stack on that PR's branch (the PR then targets it), or wait for it to merge.

**Name the branch `<type>/<issue>-<slug>`**: `<type>` is the conventional-commit type the change will carry (`fix`, `feat`, `docs`, `refactor`, `test`, `chore`); `<slug>` is two to five kebab-case words from the issue title. Example: `fix/771-timer-label-clip`.

```
git worktree add .claude/worktrees/<issue>-<slug> -b <type>/<issue>-<slug> <base>
```

**Inside the repo, not beside it**, so the harness asks nobody to trust a new path. **If you are Claude Code**, enter it with `EnterWorktree` and its `path` argument, which is why the worktree is created first. Called without one, the harness makes a worktree of its own on a `worktree-…` branch. **Any other agent**: make it your working directory, or pass `git -C <worktree-path>` on every command. If a heredoc or `>` is refused inside the worktree, use file-writing tools instead of fighting it.

**Done when:** you are working in the worktree, on the named branch, and have said the base and branch in one line.

### 5. Spec first

Check you are in the worktree (`git rev-parse --show-toplevel` names it) before the first edit. Find the spec that governs the code you will touch; the repo's `AGENTS.md` says where specs live. When behaviour, a contract, or a rule changes, edit that spec in this change, ahead of or alongside the code. State the rule as if it was always designed that way; how it changed belongs on the issue and the PR.

- **Gated names** follow the repo's naming authority: propose each as its own question.
- **A call that is the human's**: ask it on its own, with your recommendation, and record the answer on the issue.
- **Fog**: see [Leaving for the long route](#leaving-for-the-long-route).

**Done when:** every spec the change touches says what the code will do, or you can say in one line why none changes.

### 6. Build it, test it

Test first where a seam allows (`/tdd`): a test that goes **red** without the change and green with it. A change with no behaviour (prose, a comment, config text) needs none; say so. Run the repo's fast gate, which its `AGENTS.md` names.

**Done when:** every acceptance criterion on the issue is met and is covered by a test or a stated reason, and the gate is green.

### 7. Review if asked, then commit

If the human asked for review at step 2, run `/code-review` against the base, then fix each finding or put it to the human.

Commit through `draft-commit-message`, giving it the issue. **Never commit to `main`.**

**Done when:** the work is committed on the branch.

### 8. Push and open the PR

Stop here if the ending is a local commit. Otherwise push with an explicit refspec (`git push -u origin <branch>`), and stop there if the ending is a pushed branch.

For a PR, open it against the base. Its body says what changed, which specs moved, and how it is tested; carries `Closes #<issue>`; and lists any approvals given during the work per `approval-policy` (who, when, and the alternative they turned down). **Never merge it.**

**Done when:** the PR is open and you have reported its link and its CI state as it stands.

## Leaving for the long route

**Fog** can show up after step 1: a decision that grows a second one, a new ADR, scope spilling into another assembly. When it does, stop building. Write what you found on the issue, leave the worktree and branch where they are, and put step 1's question again with the door `describe-wayfinder-pipeline` gives, usually `/wayfinder #<issue>`.

## Where this sits

`/quick-change` is the short route; `describe-wayfinder-pipeline` explains the long one. Both end in a PR that `/review-pr-in-worktree` reads and `/squash-merge-and-clean-up` lands.
