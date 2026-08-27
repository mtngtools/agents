# Skills Index

Definitive skills for `mtngtools` organization. Every skill declares two things in its frontmatter:

- **`metadata.type`** — what kind of thing it is: `command` for a discrete operation, `skill` for one that composes other skills. Gemini and Antigravity map this onto their own surfaces; `rules/` uses the same vocabulary.
- **`metadata.invocation`** — who is allowed to start it.

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
| [commit-wip](./repo/commit-wip/SKILL.md) | Commit current changes as WIP | `skill-callable` | commits, git, wip |
| [commit-with-issue](./repo/commit-with-issue/SKILL.md) | Commit with issue reference | `skill-callable` | commits, git, issues |
| [commit-without-issue](./repo/commit-without-issue/SKILL.md) | Commit without issue reference | `skill-callable` | commits, git, no-issue-tracker |
| [create-branch-not-pushed](./repo/create-branch-not-pushed/SKILL.md) | Create branch for unpushed commits | `skill-callable` | branching, git, commits |
| [create-develop-branch](./repo/create-develop-branch/SKILL.md) | Create timestamped develop branch | `skill-callable` | branching, git |
| [draft-commit-message](./repo/draft-commit-message/SKILL.md) | Draft conventional commit message | `skill-callable` | commits, git, messages |
| [create-issue-commit](./repo/create-issue-commit/SKILL.md) | Create issue and commit changes | `human-only` | issues, commits, git, github |
| [create-issue-to-rebase-wip](./repo/create-issue-to-rebase-wip/SKILL.md) | Create issue for WIP, rebase with it | `human-only` | issues, git, rebase, wip |
| [create-pr-for-branch](./repo/create-pr-for-branch/SKILL.md) | Create PR for current branch | `human-only` | prs, github, github-api |
| [pull-back-from-main](./repo/pull-back-from-main/SKILL.md) | Pull back from main, delete branch | `human-only` | branching, git |
| [rebase-wip-with-issue](./repo/rebase-wip-with-issue/SKILL.md) | Rebase WIP commits with issue | `human-only` | git, rebase, issues, wip |
| [review-pr-in-worktree](./repo/review-pr-in-worktree/SKILL.md) | Review a PR in its own worktree: committed diff, tests, plan, drift | `human-only` | prs, github, review, git, worktrees, specs |
| [review-a-pr-and-report](./repo/review-a-pr-and-report/SKILL.md) | The review itself: committed diff, gate, spec, drift, fixed report | `skill-callable` | prs, github, review, specs, tests |
| [squash-merge-and-clean-up](./repo/squash-merge-and-clean-up/SKILL.md) | Squash-merge the session's PR, remove its branches and worktree | `human-only` | prs, github, branching, git, worktrees |

The split is ownership of state. Local, reversible work — staging a commit, cutting a branch, drafting a message — is `skill-callable`, so a commit flow can chain through `draft-commit-message` without stopping to ask. Anything that rewrites history or touches GitHub is `human-only`. `review-pr-in-worktree` is `human-only` for a different reason: it writes nothing at all, but it renders a judgment someone else acts on, and that is asked for, not volunteered. It keeps only the worktree — settling which PR, checking it out, tearing it down — and calls `review-a-pr-and-report` for the reviewing, which is `skill-callable` because it is the same judgment wherever the checkout came from. That name is generic enough to attract unprompted invocation, so its description holds the line the harness cannot: it names its caller and says not to self-start.

## Planning skills

The wayfinder pipeline — decisions become specs, specs become tickets — plus the survey that says which map to enter.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [check-wayfinder-maps](./plan/check-wayfinder-maps/SKILL.md) | Survey all wayfinder maps; report what's ready to build | `human-only` | planning, wayfinder, github, survey |
| [read-the-map](./plan/read-the-map/SKILL.md) | Read one map — verdict and next door; owns the checklist the survey follows | `skill-callable` | planning, wayfinder, github, maps |
| [are-decisions-from-this-session-saved](./plan/are-decisions-from-this-session-saved/SKILL.md) | Ask what planning would be lost if the session ended; record each system decision or forget it with approval | `skill-callable` | planning, wayfinder, specs, tickets, sessions |
| [whats-next](./plan/whats-next/SKILL.md) | Hand back a short, copyable prompt for the next session on the current map | `skill-callable` | planning, wayfinder, sessions, prompts |
| [plan-mtng-tools-vue](./plan/plan-mtng-tools-vue/SKILL.md) | Plan Vue component/composable spec | `skill-callable` | planning, vue, frontend, specs |
| [decisions-to-specs](./plan/decisions-to-specs/SKILL.md) | Settle a map's decisions into repo spec files and ADRs, written in a worktree of its own | `human-only` | planning, wayfinder, specs, adrs, worktrees |
| [specs-to-tickets](./plan/specs-to-tickets/SKILL.md) | Slice a map's settled specs into implementation tickets | `human-only` | planning, wayfinder, specs, tickets |

**Pipeline order:** `/wayfinder` → `/decisions-to-specs` → `/specs-to-tickets` → `/implement`. Each operates on one map; `/check-wayfinder-maps` reads across all of them and tells you which one to enter, and by which door. `/are-decisions-from-this-session-saved` sits at the other end of a session, checking that what it decided about the system — not about how it was worked — reached a surface at all. `/whats-next` closes the same seam from the other side, handing over the prompt that starts the following session on the same map, off a `/read-the-map` reading. `/read-the-map` defines what reading a map means; `/check-wayfinder-maps` runs that same checklist across every map, in bulk. The two middle steps write to the repo and the tracker, so each is a door you open yourself. So is the survey — it writes nothing, but a sweep of every map on a repo is asked for, not volunteered. `/read-the-map` is the callable read: one map, and what `/whats-next` chains through.

## General skills

Cross-cutting operations.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [concise-copy](./general/concise-copy/SKILL.md) | Refine copy and documentation | `skill-callable` | writing, documentation, content |
| [75-concise](./general/75-concise/SKILL.md) | Cut text to ~75% — a light trim | `skill-callable` | writing, documentation, reduction |
| [50-concise](./general/50-concise/SKILL.md) | Cut text to ~50% | `skill-callable` | writing, documentation, reduction |
| [25-concise](./general/25-concise/SKILL.md) | Cut text to ~25% | `skill-callable` | writing, documentation, reduction |
| [10-concise](./general/10-concise/SKILL.md) | Cut text to ~10% — bites hardest | `skill-callable` | writing, documentation, reduction |

The numbered variants share one [reduction method](./general/concise-copy/reduce.md); the percentage is a ceiling, never a floor on meaning.

## Frontend skills

Vue component and composable workflows.

| Name | Description | Invocation | Applies to |
|------|-------------|-----------|-----------|
| [build-mtng-tools-vue](./front-end/build-mtng-tools-vue/SKILL.md) | Build Vue component/composable | `skill-callable` | building, vue, frontend, specs |

Its planning counterpart, [plan-mtng-tools-vue](./plan/plan-mtng-tools-vue/SKILL.md), is listed under Planning.

## Loading

Reference skills in output as `/skill-name` (e.g., `/commit-with-issue`). If not installed in your system, pull from [`mtngtools/agents`](https://github.com/mtngtools/agents) `skills/` folder.

No skill here is `model-discoverable` yet — every one of them is something a human asks for. The category exists for convention skills that a model *should* pick up on its own when it recognises the work, the way `mtng-tools-vue` applies whenever a Vue SFC is being written.

**Referencing another skill:** name it — "invoke the `draft-commit-message` skill" — rather than linking to its `SKILL.md`. A link invites an agent to read the file straight through, past the category it declares; naming the skill makes the reference an invocation, which the category governs. Linking to a non-skill reference file, like [reduce.md](./general/concise-copy/reduce.md), is fine.
