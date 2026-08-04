---
name: specs-to-tickets
description: Turn the specs settled from a wayfinder map into implementation tickets — read the map's Specs-settled notes, then run Matt's /to-tickets against each spec.
argument-hint: "The wayfinder map (issue URL or number)"
disable-model-invocation: true
---

# Specs → Tickets

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

**Done when:** every approved spec has produced its tickets on the tracker.

### 4. Record the tickets back on the map

Append a `## Tickets` section to the map body, grouped by spec — each spec followed by the implementation tickets it produced (name + link). The map stays the single running guide, now pointing all the way from decision → spec → ticket.

```markdown
## Tickets
<!-- written by /specs-to-tickets -->
- `src/<Package>/spec/<feature>.md` → #31, #32, #33
- `spec/<cross-cutting>.md` → #34, #35
```

**Done when:** the map lists the implementation tickets for every sliced spec.

### 5. Report

List the tickets created per spec, and point at `/implement` as the next step — **fresh context per ticket**, blockers-first. Remind the user that these implementation tickets are the build backlog; the wayfinder decision issues stay closed as rationale.

## Where this sits in the flow

`/wayfinder` → decision issues → `/decisions-to-specs` → spec files + `## Specs settled` → **`/specs-to-tickets`** → `/to-tickets` per spec → GitHub implementation tickets → `/implement`.
