---
name: bump-submodules
description: Ask why, then move every submodule pointer to its origin main in a worktree of its own, bring the gate back to green, and open the PR against an issue that holds the reason.
argument-hint: "An issue to work under, if one exists; which submodules, if not all of them"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [submodules, git, worktrees, tests, issues, prs, github]
---

# bump-submodules

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

Ask the human why the bump is being made. Then find every submodule the superproject pins behind its `origin/main`, show what a bump would move, and — on their yes — open an issue that records the reason and the set, move the pointers in a worktree of this session's own, adapt the repo to what came in until the gate is green again, and open the PR that closes the issue. Stop there: `/squash-merge-and-clean-up` lands it.

**The bump is the change; everything else on the branch exists to keep the bump green.** Adapting a call site to a renamed upstream type belongs here. Fixing a test that already failed on `main` does not, and neither does anything the bump did not make necessary. A reviewer has to be able to read the PR as a bump.

**The reason outlives the PR, so it lives on the issue.** A pointer move with no stated reason is unreadable a month later — nobody can tell a deliberate catch-up from an accident, or say what would break if it were reverted. The issue is where the reason is kept, and the PR closes it.

**A worktree because the primary checkout is shared.** Moving a submodule checkout and running the gate over it is exactly the disturbance other sessions cannot afford, and a pointer left half-moved in the shared checkout reads as someone's uncommitted work.

## Process

### 1. Ask why the bump is being made

**With an issue given** as the argument, read it first — `gh issue view <n>`. A body that already says why is the answer, and the question below is skipped. A body that does not gets the answer added as a comment once you have it.

Otherwise ask, one question, before anything is fetched or created:

> Why is this bump being made?

Offer these, and take the free text as the answer when it comes:

- **Something here needs what upstream now has** — which change, and what is waiting on it
- **Upstream fixed something this repo hits** — which failure or bug
- **Routine catch-up** — nothing is waiting on it

The first two need a specific. A bare pick — "something here needs it" with nothing named — is a category, not a reason; ask once more for the change, the fix, or the thing that is waiting. **If you are Claude Code**, `AskUserQuestion` is the tool, and it supplies the free-text **Other** for you.

Hold the reason verbatim, with who gave it and today's date. It shapes the survey in step 3 and is written down in step 4.

**Done when:** you can say in one line why this bump is happening, and who said so.

### 2. Settle the base and build the worktree

**The base is a freshly fetched `origin/main` of the superproject** — never the local `main`, never the current `HEAD`. There is no stacking question here: a bump is only ever a bump of what `main` records.

```
git fetch origin
git worktree list
gh pr list --state open --json number,title,headRefName,files
```

Two things end the skill before it starts. An **open PR whose files include a submodule path or `.gitmodules`** is the bump already in flight — say which PR, and stop; a second one is a duplicate. A **worktree already at this skill's path** is a previous run — name it and ask before touching it.

```
git worktree add .claude/worktrees/bump-submodules-<YYYY-MM-DD> -b chore/bump-submodules-<YYYY-MM-DD> origin/main
```

**Inside the repo, not beside it.** A path outside the working directory makes the harness ask the human to trust it — a prompt that has nothing to do with the work, and that no allowlist suppresses. **If you are Claude Code**, enter it with `EnterWorktree` and its `path` argument, which needs the worktree to already appear in `git worktree list` — hence the order above. **Any other agent** — make that directory your working directory by whatever mechanism your harness provides, or pass the path explicitly (`git -C <worktree-path> …`) on every command.

**A new worktree's submodule directories are empty.** Populate them before anything else:

```
git submodule update --init --recursive
```

Some harnesses refuse shell redirection inside a worktree. If a heredoc or `>` is refused, use file-writing tools and plain single-purpose commands rather than fighting it.

**Done when:** the session is in the worktree, on its own branch off `origin/main`, with every submodule checked out at the pointer `main` records.

### 3. Survey: what each submodule pins, and what its `main` holds

For each `path` in `.gitmodules`:

```
git -C <path> fetch origin
git -C <path> rev-parse HEAD                                # the pinned commit
git -C <path> rev-parse origin/main                         # the target
git -C <path> merge-base --is-ancestor HEAD origin/main     # is the pin on main at all?
git -C <path> rev-list --count HEAD..origin/main            # how far behind
git -C <path> log --oneline HEAD..origin/main               # what a bump brings in
```

**`origin/main` is the target.** A submodule with no `main` takes the branch its `origin/HEAD` points at, and the plan says so. Read the repo's own guidance on the submodule first — `AGENTS.md`, `docs/`, a spec — because a repo that says its pointer must sit on a tag, or that moving it is a two-repo flow, has already narrowed what a bump means there: the target becomes what that guidance allows, and the plan names it.

Sort each submodule into one of three:

- **Current** — pinned at the target. Nothing to do.
- **Behind** — pinned at an ancestor of the target. A candidate; hold its commit list for the issue and the PR body.
- **Off `main`** — the pinned commit is not an ancestor of the target. Somebody pinned a branch on purpose — in-flight work upstream that `main` has not taken yet — and moving it would drop those commits from the superproject. **Not a candidate.** Name it, show what it holds that `main` does not (`git -C <path> log --oneline origin/main..HEAD`), and leave it out of the plan unless the human puts it back in.

**Check the reason against what comes in.** When step 1 named a specific upstream change or fix, find it in the commit lists. Present, the plan says which submodule carries it. Absent, say so before asking — a bump that does not contain the thing it was for is the human's to reconsider, not yours to run anyway.

**Nothing behind:** say so, remove the worktree and branch you just made — `git worktree remove --force <path>` (populated submodules refuse a plain remove even when clean), then `git branch -D <branch>` — and stop. No issue and no PR for nothing.

**Done when:** every submodule is sorted, each behind one has its old SHA, new SHA, count, and commit list held, and the reason has been found in what comes in or reported missing.

### 4. Ask before moving anything, then open the issue

**The plan travels by two routes, because each alone fails.** Text printed just before a tool call may never render, and a plan inlined in the question arrives as one unreadable paragraph. So print the plan as ordinary output, **and** carry the same text verbatim as every option's `preview`.

```markdown
PLAN:
Reason: <the reason from step 1, verbatim>
- `sub/foo`  abc1234 → def5678  (12 commits behind `origin/main`; carries <the named change>)
- `sub/bar`  current — no change
- `sub/baz`  off `main` — pinned 3 commits `main` lacks — LEFT OUT
Branch `chore/bump-submodules-<date>` off `origin/main`, in `.claude/worktrees/bump-submodules-<date>`
```

Then one question:

> Bump the submodules shown in the preview?

Options are **Yes, bump them** and **No, stop here**, each carrying the full `PLAN:` as its `preview`. Free text is how the human narrows the set or pulls an off-`main` submodule back in; never enumerate it, and take the answer at face value.

On **no**: remove the worktree and branch as in step 3, and stop.

**On yes, open the issue** — unless the argument gave one, in which case add the reason there as a comment if its body lacks it, and use that issue from here on. Write the body with a file-writing tool and follow the repo's issue-tracker guidance for labels and shape where it has one:

```
gh issue create --title "chore(submodules): bump <names> to origin/main" --body-file <file>
```

The body, in this order:

1. **Why** — the reason verbatim, then `— given by <name>, <date>`.
2. **What moves** — the table from the plan: submodule, old → new, count, and where the reason's named change sits.
3. **What is left out** — off-`main` submodules and anything the human removed at the ask, each with its reason.

This is the approval record too, per the `approval-policy` skill: the human's yes, by name and date, against the set and the alternatives left out. Hold the issue number; the commit references it and the PR closes it.

**Done when:** the human has said yes to a named set, and one issue holds the reason and that set.

### 5. Move the pointers

For each approved submodule:

```
git -C <path> checkout --detach <new-sha>
git add <path>
```

Only the pointer moves. `.gitmodules` is untouched — a bump changes which commit, never the URL or branch — and no other path is staged.

**Done when:** `git diff --cached --submodule=log` shows exactly the approved moves and nothing else.

### 6. Run the gate, then make it green

Run the gate the repo names for local use — its `AGENTS.md` or `CLAUDE.md` says which script. A repo that names none gets its build and full test suite. CI runs the full gate after the push in step 8, and its verdict counts the same as this one.

**A failure is the bump's until proven otherwise, and proving it is one flip.** Put the pointer back on the old commit, rerun only the failing test, then put it forward again:

```
git -C <path> checkout --detach <old-sha>
git -C <path> checkout --detach <new-sha>
```

- **Fails on the old pointer too:** pre-existing. Not this branch's. Name it in the PR body and leave it; fixing it here turns a bump into something a reviewer cannot read as one.
- **Passes on the old, fails on the new:** the bump caused it. Fix it.

**A fix adapts this repo to what upstream now is.** A renamed type, a changed signature, a moved file, generated code to regenerate from a changed schema: the fix lives in the superproject's own code. A test that asserted the old upstream behaviour is updated to assert the new one, and the PR body says which and why.

Four things look like fixes and are not: moving a pointer back partway to a commit where the test passed (a smaller bump nobody approved); editing inside a submodule (the pointer then names a commit that is not on `main`, an off-`main` pin made here); skipping or deleting a test; relaxing the gate. Each of those is a decision, and it is the human's — ask, one question, with what you would do and why.

**Ask, one question, when the fix needs a call the human owns** — a name something outside the assembly will use, an upstream change the repo's own spec contradicts, an upstream commit that is itself broken. For the last, offer to leave that submodule at its old pointer and bump the rest; a set that changes here is noted on the issue.

**The same failure three times without converging is a stop, not a fourth try.** Say what fails, what you tried, and ask.

Rerun the full local gate after the last fix, over the pointer moves and every adaptation together.

**Done when:** the gate is green, and `git status` shows only the pointer moves and the adaptations they required.

### 7. Commit — one commit

The pointer moves and their adaptations go in **one commit**, so no commit on the branch carries a bump that does not build. Invoke the `draft-commit-message` skill — type `chore`, scope `submodules` unless the repo's convention says otherwise — referencing the issue from step 4.

Commit to the worktree's branch. **Never commit to `main`.**

**Done when:** one commit on the branch, containing exactly what step 6 left green, with the issue on its first line.

### 8. Push, open the PR, watch CI

```
git push -u origin <branch>
gh pr create --base main --title "<the commit subject>" --body-file <file>
```

Write the body with a file-writing tool — the worktree may refuse a heredoc — in this order:

1. **Why** — the reason in one line, and a link to the issue that holds it.
2. **The table** — submodule, old → new as short SHAs, the count, and a compare link (`<upstream url>/compare/<old>...<new>`).
3. **What came in** — per submodule, the `--oneline` list from step 3.
4. **What was adapted and why** — each fix mapped to the upstream change that forced it.
5. **What was left** — pre-existing failures from step 6, off-`main` submodules from step 3, each with its reason.
6. `Closes #<issue>` — the issue from step 4.

Then `gh pr checks <n> --watch`. **Red sends you back to step 6** with the CI log as the failure; the fix is a second commit on the branch, squashed at merge. A job that went red in seconds with no log is the runner, not the code — read the run's annotations before touching anything.

**Done when:** the PR is open, it closes the issue, and its checks are green.

### 9. Hand back and stop

Give the PR's URL and a markdown link to it, and the issue's. Then, for someone who did not watch: why, what moved, what was adapted, what was left and why, and where the worktree is.

**Leave the worktree standing.** `/squash-merge-and-clean-up` removes it after the merge, which is the human's call. Do not merge, do not poll after green, do not schedule a re-check — an open, green PR is the complete ending of this skill.

## Where this sits in the flow

**`/bump-submodules`** opens it; `/review-pr-in-worktree` judges it; `/squash-merge-and-clean-up` lands it, closes the issue through the merge, and takes the worktree away. Each is a separate door a human opens. Bumping a submodule that also needs adoption work in the superproject — a wire schema whose new shape the code has to consume, say — is a build ticket for `/implement`, not a bump: this skill only ever catches the pointer up to what `main` already holds.
