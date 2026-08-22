# Skills Index

Definitive skills for `mtngtools` organization. Each skill has frontmatter indicating discovery mode and applicable tags.

## Repository skills

Git, branching, commits, and PR workflows. All human-initiated.

| Name | Description | Applies to |
|------|-------------|-----------|
| [commit-wip](./repo/commit-wip/SKILL.md) | Commit current changes as WIP | commits, git, wip |
| [commit-with-issue](./repo/commit-with-issue/SKILL.md) | Commit with issue reference | commits, git, issues |
| [commit-without-issue](./repo/commit-without-issue/SKILL.md) | Commit without issue reference | commits, git, no-issue-tracker |
| [create-branch-not-pushed](./repo/create-branch-not-pushed/SKILL.md) | Create branch for unpushed commits | branching, git, commits |
| [create-develop-branch](./repo/create-develop-branch/SKILL.md) | Create timestamped develop branch | branching, git |
| [create-issue-commit](./repo/create-issue-commit/SKILL.md) | Create issue and commit changes | issues, commits, git, github |
| [create-issue-to-rebase-wip](./repo/create-issue-to-rebase-wip/SKILL.md) | Create issue for WIP, rebase with it | issues, git, rebase, wip |
| [create-pr-for-branch](./repo/create-pr-for-branch/SKILL.md) | Create PR for current branch | prs, github, github-api |
| [draft-commit-message](./repo/draft-commit-message/SKILL.md) | Draft conventional commit message | commits, git, messages |
| [pull-back-from-main](./repo/pull-back-from-main/SKILL.md) | Pull back from main, delete branch | branching, git |
| [rebase-wip-with-issue](./repo/rebase-wip-with-issue/SKILL.md) | Rebase WIP commits with issue | git, rebase, issues, wip |

## Planning skills

The wayfinder pipeline — decisions become specs, specs become tickets — plus the survey that says which map to enter. All human-initiated.

| Name | Description | Applies to |
|------|-------------|-----------|
| [check-wayfinder-maps](./plan/check-wayfinder-maps/SKILL.md) | Survey all wayfinder maps; report what's ready to build | planning, wayfinder, github, survey |
| [decisions-to-specs](./plan/decisions-to-specs/SKILL.md) | Settle a map's decisions into repo spec files and ADRs | planning, wayfinder, specs, adrs |
| [specs-to-tickets](./plan/specs-to-tickets/SKILL.md) | Slice a map's settled specs into implementation tickets | planning, wayfinder, specs, tickets |
| [plan-mtng-tools-vue](./plan/plan-mtng-tools-vue/SKILL.md) | Plan Vue component/composable spec | planning, vue, frontend, specs |

**Pipeline order:** `/wayfinder` → `/decisions-to-specs` → `/specs-to-tickets` → `/implement`. Each operates on one map; `/check-wayfinder-maps` reads across all of them and tells you which one to enter, and by which door.

## General skills

Cross-cutting operations. All human-initiated.

| Name | Description | Applies to |
|------|-------------|-----------|
| [concise-copy](./general/concise-copy/SKILL.md) | Refine copy and documentation | writing, documentation, content |
| [75-concise](./general/75-concise/SKILL.md) | Cut text to ~75% — a light trim | writing, documentation, reduction |
| [50-concise](./general/50-concise/SKILL.md) | Cut text to ~50% | writing, documentation, reduction |
| [25-concise](./general/25-concise/SKILL.md) | Cut text to ~25% | writing, documentation, reduction |
| [10-concise](./general/10-concise/SKILL.md) | Cut text to ~10% — bites hardest | writing, documentation, reduction |

The numbered variants share one [reduction method](./general/concise-copy/reduce.md); the percentage is a ceiling, never a floor on meaning.

## Frontend skills

Vue component and composable workflows. All human-initiated.

| Name | Description | Applies to |
|------|-------------|-----------|
| [build-mtng-tools-vue](./front-end/build-mtng-tools-vue/SKILL.md) | Build Vue component/composable | building, vue, frontend, specs |

Its planning counterpart, [plan-mtng-tools-vue](./plan/plan-mtng-tools-vue/SKILL.md), is listed under Planning.

## Loading

Reference skills in output as `/skill-name` (e.g., `/commit-with-issue`). If not installed in your system, pull from [`mtngtools/agents`](https://github.com/mtngtools/agents) `skills/` folder.

All current skills are human-initiated — humans must request them. Older skills declare this as `metadata.discoverable: false`; newer ones as `disable-model-invocation: true`.
