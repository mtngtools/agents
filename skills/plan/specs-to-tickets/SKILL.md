---
name: specs-to-tickets
description: Turn the specs settled from a wayfinder map into implementation tickets — read the map's Specs-settled notes, run Matt's /to-tickets against each spec, and link every ticket under the map as a sub-issue.
argument-hint: "The wayfinder map (issue URL or number)"
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [planning, wayfinder, specs, tickets]
---

# Specs → Tickets

> **human-only.** Start this only when a human asks for it by name. If you arrived here from another skill, stop and get explicit confirmation before running any step.

Downstream of `/decisions-to-specs`. That skill settled a wayfinder map's decisions into repo spec files and recorded them on the map under `## Specs settled`. This skill reads those notes and produces the **implementation tickets** — by handing each spec to Matt's `/to-tickets`.

This is the boundary where planning becomes building: the decision issues and the specs are behind you; from here it's tracer-bullet tickets → `/implement`.

## Process

### 1. Load the map and its settled specs

Resolve the map from the argument (issue URL or number). Read the `## Specs settled` section written by `/decisions-to-specs` — the list of spec files with their disposition (`new` / `updated` / `ADR`).

If the section is missing, **stop** and tell the user to run `/decisions-to-specs <map>` first — there are no settled specs to slice.

**Done when:** you have the list of spec files and their dispositions.

### 2. Confirm the scope to slice

Present the spec files to the user. Default set = every `new` and `updated` spec. **Exclude `ADR` lines** — an ADR records why a decision was made, not a buildable spec; nothing is sliced from it. Confirm the set before creating any tickets.

**Done when:** the user has approved which specs to slice.

### 3. Slice each spec with `/to-tickets`

For each approved spec, run **`/to-tickets <spec-path>`**, one spec at a time. Let it do its own job — vertical tracer-bullet slices, blocking edges, published to the tracker (GitHub). Don't re-implement its logic here; this skill only routes the right specs into it.

Keep the issue numbers it creates, per spec and in dependency order — the next step consumes them.

**Done when:** every approved spec has produced its tickets on the tracker.

### 4. Link every ticket to the map as a sub-issue

`/to-tickets` leaves the parent issue alone by design, so the linking is this skill's job. Read the repo's issue-tracker doc (`docs/agents/issue-tracker.md`), **Wayfinding operations**, for how this repo links a child to a map, and follow it. Where the doc is silent:

```bash
gh api --method POST repos/<owner>/<repo>/issues/<map>/sub_issues -F sub_issue_id=<ticket-db-id>
```

`<ticket-db-id>` is the ticket's numeric **database id** (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`), not its `#number`. Where sub-issues aren't enabled, fall back to a task list in the map body plus `Part of #<map>` at the top of each ticket body.

Link in dependency order, blockers first, so the map's child list reads in build order. An issue has one parent: the map is it. The edges `/to-tickets` set between tickets are **dependencies**, and they stay dependencies.

These tickets are the map's build backlog, not wayfinder children — they carry `ready-for-agent` and no `wayfinder:<type>` label, which is what keeps the wayfinder frontier to decision work.

**Done when:** every ticket from step 3 appears in the map's sub-issue list.

### 5. Record the tickets back on the map

Append a `## Tickets` section to the map body, grouped by spec — each spec followed by the implementation tickets it produced (name + link). The sub-issue list from step 4 is the live link; this section adds the spec grouping that list can't carry, so the map points all the way from decision → spec → ticket.

```markdown
## Tickets
<!-- written by /specs-to-tickets -->
- `src/<Package>/spec/<feature>.md` → #31, #32, #33
- `spec/<cross-cutting>.md` → #34, #35
```

**Done when:** the map lists the implementation tickets for every sliced spec.

### 6. Report

List the tickets created per spec, and point at `/implement` as the next step — **fresh context per ticket**, blockers-first. Remind the user that these implementation tickets are the build backlog; the wayfinder decision issues stay closed as rationale.

## Where this sits in the flow

`/wayfinder` → decision issues → `/decisions-to-specs` → spec files + `## Specs settled` → **`/specs-to-tickets`** → `/to-tickets` per spec → GitHub implementation tickets → `/implement`.
