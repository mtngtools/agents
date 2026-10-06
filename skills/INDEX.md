# Skills Index

Definitive skills for `mtngtools` organization. Every skill declares two things in its frontmatter:

- **`metadata.type`** — what kind of thing it is: `command` for a discrete operation, `skill` for one that composes other skills. Gemini and Antigravity map this onto their own surfaces; `rules/` uses the same vocabulary.
- **`metadata.invocation`** — who is allowed to start it.

Every skill is its own directory at the root of `skills/`, named for the skill. The categories below group them for reading; they are not directories, and a new skill lands beside the others rather than under a heading.

## Invocation categories

| Category | Who may start it | Frontmatter |
|---|---|---|
| `human-only` | Only a human, by name. Another skill that reaches for it must stop and get explicit confirmation first. | `disable-model-invocation: true` |
| `skill-callable` | A human by name, or another skill chaining into it without asking. | *(flag omitted)* |
| `model-discoverable` | The model may reach for it unprompted when the description matches the task. | *(flag omitted)* |

**The harness has two states, not three.** `disable-model-invocation: true` keeps a skill out of the model's skill listing, and a skill that is not in the listing cannot be invoked by the model **at all** — including when another skill tells it to. There is no caller identity to check: one skill "calling" another is Claude calling the Skill tool, the same mechanism as Claude reaching for it unprompted. So `skill-callable` and `model-discoverable` share one frontmatter, and only their descriptions separate them.

**Being in the listing costs one line.** Just the name and the description load; the body loads when the skill is invoked, not before. The listing is capped at 1% of the context window, and on overflow Claude Code strips descriptions starting with the skills invoked least — a stripped description keeps the name and loses the words a caller matches on. Keep `skill-callable` descriptions short.

**Holding a line the harness won't hold.** A `skill-callable` skill that starts getting reached for unprompted is fixed in its `description`: name who calls it, and say it is not to be invoked on the model's own initiative. That is a convention agents follow, not something enforced — the same footing as the gate paragraph every `human-only` skill carries in its body, which tells an agent arriving by any route other than a human naming it to stop and confirm first.

## Repository skills

Git, branching, commits, and PR workflows.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [commit-wip](./commit-wip/SKILL.md) | Commit current changes as WIP | `skill-callable` | commits, git, wip |
| [commit-with-issue](./commit-with-issue/SKILL.md) | Commit with issue reference | `skill-callable` | commits, git, issues |
| [commit-without-issue](./commit-without-issue/SKILL.md) | Commit without issue reference | `skill-callable` | commits, git, no-issue-tracker |
| [commit-local-main](./commit-local-main/SKILL.md) | Commit onto local `main` where that's allowed; never pushes | `human-only` | commits, git, main, local-only |
| [create-branch-not-pushed](./create-branch-not-pushed/SKILL.md) | Create branch for unpushed commits | `skill-callable` | branching, git, commits |
| [create-develop-branch](./create-develop-branch/SKILL.md) | Create timestamped develop branch | `skill-callable` | branching, git |
| [draft-commit-message](./draft-commit-message/SKILL.md) | Draft conventional commit message | `skill-callable` | commits, git, messages |
| [create-issue-commit](./create-issue-commit/SKILL.md) | Create issue and commit changes | `human-only` | issues, commits, git, github |
| [create-issue-to-rebase-wip](./create-issue-to-rebase-wip/SKILL.md) | Create issue for WIP, rebase with it | `human-only` | issues, git, rebase, wip |
| [create-pr-for-branch](./create-pr-for-branch/SKILL.md) | Create PR for current branch | `human-only` | prs, github, github-api |
| [pull-back-from-main](./pull-back-from-main/SKILL.md) | Pull back from main, delete branch | `human-only` | branching, git |
| [rebase-wip-with-issue](./rebase-wip-with-issue/SKILL.md) | Rebase WIP commits with issue | `human-only` | git, rebase, issues, wip |
| [review-pr-in-worktree](./review-pr-in-worktree/SKILL.md) | Review a PR in its own worktree: committed diff, tests, plan, drift | `human-only` | prs, github, review, git, worktrees, specs |
| [review-a-pr-and-report](./review-a-pr-and-report/SKILL.md) | The review itself: committed diff, gate, spec, drift, fixed report | `skill-callable` | prs, github, review, specs, tests |
| [respond-to-pr-review](./respond-to-pr-review/SKILL.md) | Answer a review item by item — fix or discuss each, record approvals on the tracker | `human-only` | prs, github, review, git, tickets, approvals |
| [squash-merge-and-clean-up](./squash-merge-and-clean-up/SKILL.md) | Squash-merge the session's PR, remove its branches and worktree | `human-only` | prs, github, branching, git, worktrees |
| [bump-submodules](./bump-submodules/SKILL.md) | Ask why, then catch every submodule pointer up to its origin `main` in a worktree, make the gate green, open the PR against an issue holding the reason | `human-only` | submodules, git, worktrees, tests, issues, prs, github |

The split is ownership of state. Local, reversible work — staging a commit, cutting a branch, drafting a message — is `skill-callable`, so a commit flow can chain through `draft-commit-message` without stopping to ask. Anything that rewrites history or touches GitHub is `human-only`. `review-pr-in-worktree` is `human-only` for a different reason: it writes nothing at all, but it renders a judgment someone else acts on, and that is asked for, not volunteered. It keeps only the worktree — settling which PR, checking it out, tearing it down — and calls `review-a-pr-and-report` for the reviewing, which is `skill-callable` because it is the same judgment wherever the checkout came from. That name is generic enough to attract unprompted invocation, so its description holds the line the harness cannot: it names its caller and says not to self-start. `respond-to-pr-review` is `human-only` because every item's disposition is the human's — a review landing in the conversation is not permission to start answering it, and an agent left to answer alone fixes what it agrees with and quietly drops the rest. It is also where approvals given in conversation get written down: on the PR body or the issue, never as a rationale paragraph committed to the tree. `commit-local-main` is `human-only` for a fourth reason: it is not a step in a flow but a suspension of a rule — the `git-and-github` ban on committing to `main` — and the human typing its name is the whole of the authorization, which a skill chaining into it would manufacture for itself. `bump-submodules` is `human-only` for the plain reason — it pushes and opens a PR — and for one of its own: a pointer it moves may be one somebody pinned off `main` on purpose, so it asks why first, shows the set, and opens an issue holding the reason and the approved set before anything moves — the PR closes that issue — and it stops at the PR like `implement` does.

## Planning skills

The wayfinder pipeline — decisions become specs, specs become tickets — plus the survey that says which map to enter.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [check-wayfinder-maps](./check-wayfinder-maps/SKILL.md) | Survey all wayfinder maps; report what's ready to build | `human-only` | planning, wayfinder, github, survey |
| [read-the-map](./read-the-map/SKILL.md) | Read one map — verdict and next door; owns the checklist the survey follows | `skill-callable` | planning, wayfinder, github, maps |
| [are-decisions-from-this-session-saved](./are-decisions-from-this-session-saved/SKILL.md) | Ask what planning would be lost if the session ended; record each system decision or forget it with approval | `skill-callable` | planning, wayfinder, specs, tickets, sessions |
| [whats-next](./whats-next/SKILL.md) | Hand back a short, copyable prompt for the next session on the current map | `skill-callable` | planning, wayfinder, sessions, prompts |
| [re-ask-questions](./re-ask-questions/SKILL.md) | Re-ask open questions one at a time — overview, pros/cons table, clear recommendation | `skill-callable` | planning, questions, decisions |
| [plan-mtng-tools-vue](./plan-mtng-tools-vue/SKILL.md) | Plan Vue component/composable spec | `skill-callable` | planning, vue, frontend, specs |
| [decisions-to-specs](./decisions-to-specs/SKILL.md) | Settle a map's decisions into repo spec files and ADRs, written in a worktree of its own | `human-only` | planning, wayfinder, specs, adrs, worktrees |
| [specs-to-tickets](./specs-to-tickets/SKILL.md) | Slice a map's settled specs into implementation tickets | `human-only` | planning, wayfinder, specs, tickets |
| [to-tickets](./to-tickets/SKILL.md) | Slice one spec or plan into tracer-bullet tickets with blocking edges | `skill-callable` | planning, tickets, tracker, slicing |
| [describe-wayfinder-pipeline](./describe-wayfinder-pipeline/SKILL.md) | The long route explained — each stage and its door, and the test that picks it or `/quick-change` | `model-discoverable` | planning, wayfinder, specs, tickets, building, routing |

**Pipeline order:** `/wayfinder` → `/decisions-to-specs` → `/specs-to-tickets` → `/to-tickets` (per spec) → [`/implement`](./implement/SKILL.md), the last of which is listed under Build. Each operates on one map; `/check-wayfinder-maps` reads across all of them and tells you which one to enter, and by which door. `/are-decisions-from-this-session-saved` sits at the other end of a session, checking that what it decided about the system — not about how it was worked — reached a surface at all. `/whats-next` closes the same seam from the other side, handing over the prompt that starts the following session on the same map, off a `/read-the-map` reading. `/read-the-map` defines what reading a map means; `/check-wayfinder-maps` runs that same checklist across every map, in bulk. The two middle steps write to the repo and the tracker, so each is a door you open yourself. So is the survey — it writes nothing, but a sweep of every map on a repo is asked for, not volunteered. `/read-the-map` is the callable read: one map, and what `/whats-next` chains through. `/to-tickets` is the other callable one, and for the same reason as `review-a-pr-and-report`: `/specs-to-tickets` must be able to chain into it per spec without stopping, so `human-only` would break the very step that calls it. Its description holds the line the harness cannot — it names its caller and says not to self-start, because publishing tickets to a shared tracker is asked for, never volunteered.

`describe-wayfinder-pipeline` is the pipeline written down once: each stage, its door, what it takes and what it leaves behind, and the **Which route** test that sends a change down it or down `/quick-change`. It is `model-discoverable` for `approval-policy`'s reason: the moment to explain the route is when work arrives, and nobody stops to type a name for that. It runs nothing. Every door it names is `human-only`, so it hands the human the line to type.

## Build skills

Turning settled tickets into code.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [implement](./implement/SKILL.md) | Build a settled ticket in a worktree of its own, off a base chosen on purpose | `human-only` | building, tickets, worktrees, git, tests |
| [subagent-implement](./subagent-implement/SKILL.md) | Build several independent tickets at once — one subagent per ticket, one worktree each, one base | `human-only` | building, tickets, subagents, worktrees, parallel |
| [subagent-review](./subagent-review/SKILL.md) | Review committed code with one subagent per dimension, no PR required, findings verified before they reach you | `human-only` | review, subagents, git, specs, drift |
| [implement-unattended](./implement-unattended/SKILL.md) | Build with the human away: both grants on, work fanned out, integrated onto one branch, reviewed, handed back as one PR with every borrowed call tabled | `human-only` | building, subagents, approvals, naming, prs |
| [implement-unattended-no-subagents](./implement-unattended-no-subagents/SKILL.md) | The same mode and the same one PR, serially, by this session alone | `human-only` | building, approvals, naming, worktrees, prs |
| [tickets-to-lanes](./tickets-to-lanes/SKILL.md) | Split a map's build backlog into lanes, open the issue they coordinate on, hand back one unattended prompt per lane | `human-only` | building, wayfinder, parallel, sessions, prompts |
| [lifeguard-in-lanes](./lifeguard-in-lanes/SKILL.md) | Watch running lanes one tick at a time: nudge a drowning lane, alert the human if it stays under | `skill-callable` | building, parallel, sessions, monitoring, github |
| [review-pr-in-lane](./review-pr-in-lane/SKILL.md) | One lane's PR reviewer: wait for the lane's PR, then review it in a worktree as `review-pr-in-worktree` does | `human-only` | prs, github, review, worktrees, parallel, sessions |
| [summarize-lanes](./summarize-lanes/SKILL.md) | Print the detailed lane summary on request: each lane's tickets, PR and merge status, linked | `human-only` | building, parallel, sessions, github, prs |
| [quick-change](./quick-change/SKILL.md) | The short route for a small, decided change — issue, worktree off `origin/main`, spec and tests in line, one PR | `human-only` | building, issues, worktrees, specs, tests, prs |

`implement` is `human-only` for the reason the whole category is: it writes code, cuts a branch, and picks the base everything after it inherits. A skill chaining into it would be choosing that base on the human's behalf. Its framework-specific counterpart, [build-mtng-tools-vue](./build-mtng-tools-vue/SKILL.md), is listed under Frontend; the two differ in what they know about the stack, not in what they do with the tree.

**The other four are `/implement` with one axis moved.** `subagent-implement` moves *who builds* — several tickets at once, one agent each — and buys nothing unless the tickets are genuinely independent, which is the parent's job to establish before anything is created. `subagent-review` moves *who reads it afterwards*, and exists for the case `/review-pr-in-worktree` cannot serve: committed code with no PR to read. The two `implement-unattended` modes move *whether the human is reachable*, and so carry the grants from General on top; they differ from each other only in whether the building is fanned out, which is why they share one [working unattended](./implement-unattended/unattended.md) reference — including its rule that a round hands back **one** pull request however many tickets went into it, which is what most sets them apart from `subagent-implement`, whose branches each get their own. The two subagent skills likewise share one [dispatch](./subagent-implement/dispatch.md) reference — isolation, the shell hazard, the brief, the prohibitions, and the rule that a subagent's report is a claim to be checked rather than a result to be repeated.

All four are `human-only`, and the two unattended ones twice over: fanning out multiplies whatever the base decision got wrong, and handing out authority is the act `commit-local-main` and the grant skills are `human-only` for. A skill chaining into either would be manufacturing the human's approval *and* choosing how many agents to spend on it.

`tickets-to-lanes` is the same fan-out one level up: several unattended sessions over one map instead of several subagents inside one session. It starts none of them. It splits the backlog so that lanes share as few files and blockers as possible, opens the `agent:communication` issue where they hand work to each other, and gives the human one prompt per lane to paste. It is `human-only` for the unattended modes' reason, and for one of its own. The handoffs it writes are what the unattended modes' shared [in a lane](./implement-unattended/unattended.md#in-a-lane) section acts on: pushing before the PR, waiting on a sibling, and stacking on a sibling's unmerged branch. A skill chaining into it would be handing those out itself.

`lifeguard-in-lanes` runs beside the lanes, one tick per `/loop` firing, and never builds. Its writes are exactly three: a message to a lane, a `LIFEGUARD` comment on the lanes' issue, and a notification to the human. A human still starts it, but it is `skill-callable` because the human starts it through `/loop`. Each loop firing reaches the model, which starts the skill through the Skill tool, and that tool refuses any `human-only` skill, so a `human-only` lifeguard never gets past its first tick. Its description holds the line the harness cannot: it names the loop as its caller and says never to start it unprompted.

`review-pr-in-lane` is `review-pr-in-worktree` for a PR that doesn't exist yet. The human starts one beside each lane. It waits for the lane's PR, and also for any lanes that PR is stacked on to merge into `main`, so the diff shows only its own lane. Then it follows the same [worktree review](./review-pr-in-worktree/worktree-review.md) flow, which lives in that file so both skills share it. It is there so the major review starts without the human watching, not so a lane can act on it: the report goes to the human alone. It is `human-only` because a human starts it, and because it waits inside a single run and never needs a loop to fire it.

`summarize-lanes` prints the detailed summary whenever the human asks, and writes nothing. The format lives in its [detailed-summary.md](./summarize-lanes/detailed-summary.md), which the lifeguard reads directly at milestones. It reads the file rather than invoking the skill, because a looped lifeguard can't start a `human-only` skill. That lets `summarize-lanes` stay `human-only` and cost nothing in the skill listing.

`quick-change` is the short route to the whole pipeline's ending (an issue, a worktree off `origin/main`, spec and tests in line, a PR) for a change that is small and already decided. It sizes the request against `describe-wayfinder-pipeline`'s test first, and hands back the long route's door when the test fails or fog turns up mid-build. Unlike `/implement` it opens the issue it works from, with a yes, and the PR it ends in. It is `human-only` for `bump-submodules`'s plain reason: it pushes and opens a PR.

## General skills

Cross-cutting operations.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [concise-copy](./concise-copy/SKILL.md) | Refine copy and documentation | `skill-callable` | writing, documentation, content |
| [75-concise](./75-concise/SKILL.md) | Cut text to ~75% — a light trim | `skill-callable` | writing, documentation, reduction |
| [50-concise](./50-concise/SKILL.md) | Cut text to ~50% | `skill-callable` | writing, documentation, reduction |
| [25-concise](./25-concise/SKILL.md) | Cut text to ~25% | `skill-callable` | writing, documentation, reduction |
| [10-concise](./10-concise/SKILL.md) | Cut text to ~10% — bites hardest | `skill-callable` | writing, documentation, reduction |
| [approval-policy](./approval-policy/SKILL.md) | Where an approval gets recorded and what it must say — tracker holds who decided, tree never holds the discussion | `model-discoverable` | approvals, decisions, specs, tickets, prs, naming |
| [grant-naming-authority](./grant-naming-authority/SKILL.md) | Choose gated names and keep going — marked in the tree, tabled on the PR, approved by a human before merge | `human-only` | naming, specs, commits, prs, approvals |
| [grant-decision-authority](./grant-decision-authority/SKILL.md) | The same loan over any call a human would normally make, within bounds the skill enumerates | `human-only` | decisions, approvals, specs, tickets, prs |
| [can-this-throw](./can-this-throw/SKILL.md) | Trace every throw to its catch or to where it fails the app; one verdict — (a) caught with continuance, (b) can fail the app, (c) other | `model-discoverable` | code, exceptions, review, decisions, approvals |
| [settle-borrowed-authority](./settle-borrowed-authority/SKILL.md) | Settle a PR's borrowed calls with the human — names in one pass, the rest in small related groups — record approvals, take markers out | `human-only` | approvals, naming, decisions, prs, sessions |

The numbered variants share one [reduction method](./concise-copy/reduce.md); the percentage is a ceiling, never a floor on meaning.

`approval-policy` is where the other skills send an approval once it is given. The two grants are its opposite number: they lend the human's answer while they are away, and it says where the answer goes once it is real.

**The grants are one mechanism at two scopes**, so they share one [borrowed-authority ledger](./grant-decision-authority/ledger.md) — the verbatim `TEMPORARY AGENT` markers, the `grep -rn` that enumerates every open loan, the PR table generated from that grep, the note telling reviewing agents what to check, the merge gate, and the one-question-at-a-time settle-up. Each skill only says what may be borrowed: `grant-naming-authority` reaches gated names, `grant-decision-authority` reaches those plus the ordinary judgment calls around them, and its **What may be borrowed** section — in bounds, out of bounds, and "when unsure, out of bounds" — is the substance of the pair. The tree is the ledger of record: it is what survives a session that dies mid-loan, and what lets a session that was never there rebuild the table correctly.

Both are `human-only` for the same reason as `commit-local-main`: each suspends a rule rather than performing a step, and the human typing its name *is* the authorization it hands out. A skill chaining into either would be manufacturing the human's approval on their behalf. The [`implement-unattended`](./implement-unattended/SKILL.md) modes under Build carry both grants at once — and being invoked by a human is what satisfies these two skills' gate paragraphs when they do.

`can-this-throw` answers the question a human otherwise asks of every code decision: can this throw in a way nothing catches? It traces every throw source through every call site to a catch and its continuance, or to the boundary where it fails the app, and gives one verdict. It is `model-discoverable` for the same reason as `approval-policy`: it applies whenever a code decision reaches a human, and the answer is worth nothing if it arrives only when someone remembers to ask. `settle-borrowed-authority` won't ask a code case without it.

`settle-borrowed-authority` is the pair's close-out: the human back, answering the ledger's table — the names in one pass, the other calls in small related groups, one question per call — with each approval recorded per `approval-policy` and each marker removed. It is `human-only` for the plainest reason of the three: every case it processes ends in a human answer, and without one in the room it would be approving the agent's own loans. `respond-to-pr-review` settles the same markers when the answers arrive inside a review response; this is the settle-up as its own act, invoked from the session that opened the PR or handed nothing but its number.

## Frontend skills

Vue component and composable workflows.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [build-mtng-tools-vue](./build-mtng-tools-vue/SKILL.md) | Build Vue component/composable | `skill-callable` | building, vue, frontend, specs |

Its planning counterpart, [plan-mtng-tools-vue](./plan-mtng-tools-vue/SKILL.md), is listed under Planning.

## Loading

Reference skills in output as `/skill-name` (e.g., `/commit-with-issue`). If not installed in your system, pull from [`mtngtools/agents`](https://github.com/mtngtools/agents) `skills/` folder.

`approval-policy` is the repo's first `model-discoverable` skill, and it is the shape the category is for: a convention that applies whenever the work reaches it, not an operation someone asks for. A human settling something mid-task is not going to stop and type a skill name, and the recording is worthless if it only happens when they remember to — so the model reaches for it on its own, the way `mtng-tools-vue` applies whenever a Vue SFC is being written. `/decisions-to-specs`, `/specs-to-tickets`, `/to-tickets` and `/respond-to-pr-review` each say to follow it outright, because each one ends in decisions a human made out loud.

**Referencing another skill:** name it — "invoke the `draft-commit-message` skill" — rather than linking to its `SKILL.md`. A link invites an agent to read the file straight through, past the category it declares; naming the skill makes the reference an invocation, which the category governs. Linking to a non-skill reference file, like [reduce.md](./concise-copy/reduce.md), is fine.
