# Subagent dispatch

Shared by `subagent-implement`, `subagent-review`, and `implement-unattended`. Each names **what** to fan out. This file is **how** a fan-out is run without the agents ruining each other's work or handing you a pile of claims you can't check.

## The parent keeps everything that is decided once

A subagent is a worker, not a session. It has no memory of the conversation, no way to reach the human, and no idea what the other agents are doing. So the parent — the session a human invoked — keeps every decision that must be made once and inherited:

| The parent decides | Because |
|---|---|
| **The base** everything is built on | One wrong base, taken independently by six agents, is six wrong branches |
| **The isolation** — which worktree each agent gets | Two agents in one checkout is a corrupted tree, not a race you win sometimes |
| **The split** — what each agent gets and why it's independent | The agents cannot see each other, so overlap is invisible from inside |
| **The authority** the work is running under | A subagent cannot be granted anything the parent was not |
| **What the human is told** | The report is one voice, from the session that was asked |

A subagent decides only what is inside its own brief.

**A repo that bans subagent use outranks this file.** Check `AGENTS.md`/`CLAUDE.md` before dispatching; where a repo restricts fan-out to particular kinds of work, that restriction stands and a human invoking a skill does not lift it.

## Isolation

**Every agent that writes gets its own git worktree.** Not a directory, not a branch it shares — a worktree, created by the parent before dispatch, named for the unit of work:

```
git worktree add .claude/worktrees/<unit> -b <branch> <base>
```

**Inside the repo, not beside it.** A path outside the working directory makes most harnesses ask the human to trust it — a permission prompt with nothing to do with the work, and no allowlist suppresses it. There is no human to answer it here, so the dispatch simply stalls.

**Agents that only read may share one checkout**, provided the brief forbids writing to it — which it must, in those words. A reviewer that "just fixes the typo it found" has silently edited the thing every other reviewer is reading.

### The shell hazard

**Subagents may share the parent's shell and working directory.** Where they do, a `cd` in one agent moves all of them, and the symptom is not an error — it is a commit landing on the wrong branch, or a push going to a ref nobody named. It surfaces days later as a PR containing work from a ticket it never touched.

So every brief, without exception, carries this:

- **Never `cd`.** Address the worktree explicitly on every command: `git -C <worktree-path> …`, and absolute paths for everything else.
- **Push explicit refspecs.** `git -C <path> push origin <branch>:<branch>`, never a bare `git push`.
- **Never commit without naming the worktree.** `git -C <path> commit`.

Some harnesses also refuse shell redirection inside a worktree. Where a heredoc or `>` is refused, use file-writing tools and plain single-purpose commands rather than fighting it.

## The brief

A subagent's brief is the whole of its context. Everything it needs is in there, or it does not exist. Every brief carries all of:

1. **Who authorized this, and when.** Name the skill a human invoked and the date. Without it, an agent that reaches a `human-only` skill's gate paragraph — or any "confirm before proceeding" — stops and waits for a human it cannot reach.
2. **The one unit of work.** One ticket, one dimension, one file set. An agent given two does neither well and reports on one.
3. **Its worktree path**, and the shell rules above, verbatim.
4. **The base and branch** it is on, and that it may not change either.
5. **The authority it holds** — none, naming, or decision — copied from what the parent was granted, and the marker rules if it holds any.
6. **What it must return**, in the shape you will consume. Say "your final message is the return value, consumed by another agent" — otherwise you get a chatty summary written for a human who will never read it.
7. **The prohibitions**, below.

Write the brief so a stranger could run it. That is exactly what is about to happen.

## What a subagent never does

State these in the brief. They are not defaults:

- **Never merges, never pushes to `main`, never opens or reviews a PR.** Those are the parent's, and mostly the human's.
- **Never invokes a `human-only` skill.** No human is naming it, and the brief's authorization line covers this work only.
- **Never asks a question.** There is nobody there. A question becomes a **return item**: state the question, what you would have chosen, and carry on with everything that does not depend on it.
- **Never touches another agent's worktree**, or the primary checkout, or a branch it was not given.
- **Never grants itself authority.** If the brief says no naming authority, an unapprovable name is a return item, not a marker.
- **Never reports work it did not verify.** "Tests pass" means it ran them and read the output.

## Results are claims, not facts

**A subagent's report is evidence, not a result.** They overstate completion, describe intent as outcome, and report green suites they ran partially or not at all. Before any finding or claim reaches the human, the parent checks it against the tree:

- **Claims of done** — read the diff the agent actually committed, and run the gate yourself on that branch. An agent reporting green on a suite it did not run is the most common way a fan-out produces work that looks finished and isn't.
- **Claims of green, collectively** — branches that each pass alone can still break together, and every agent was blind to the others by design. Where the units touch anything in common, merge them into a scratch branch off the same base and run the gate once over that. It is not a branch you keep; it is the only place the interaction is visible before review.
- **Findings** — confirm each against the code before it is repeated. A finding you cannot reproduce is dropped, not softened.
- **Silence** — an agent that returned nothing failed. Say so; do not read it as "nothing to report".

**Never fabricate or predict a result that has not come back.** An agent still running has no findings yet, and saying otherwise is the one failure mode a human cannot detect from the outside.

## The two shapes

**Build fan-out** — one agent per unit of work, each writing, each in its own worktree, all off one base the parent settled. Used by `subagent-implement`. The hard part is the split: units must be genuinely independent, and independence is the parent's to establish before anything is dispatched.

**Review fan-out** — one agent per dimension, all reading the same commit range, none writing. Used by `subagent-review`. The hard part is independence of a different kind: no reviewer may see another's findings, because agreement between agents that read each other is worth nothing.

## How many at once

Match the fan-out to the real split, not to a number. For **build**, that is however many genuinely independent units the ticket set has — usually two to five, and one is a perfectly good answer that means "don't fan out". For **review**, one per dimension you actually intend to report on.

**Say what you dropped.** If you capped the fan-out, sampled, or skipped a unit, put it in the report. A silently truncated sweep reads as complete coverage, and that is worse than no sweep.
