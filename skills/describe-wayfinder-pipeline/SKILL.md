---
name: describe-wayfinder-pipeline
description: "The wayfinder pipeline: the long route from a loose idea to merged code, stage by stage, and the test that picks it or /quick-change. Use when sizing a change, choosing which skill starts a piece of work, or explaining how a map becomes a PR."
metadata:
  type: skill
  invocation: model-discoverable
  applies-to: [planning, wayfinder, specs, tickets, building, routing]
---

# The wayfinder pipeline

Two routes end in the same place: an issue, spec and tests in line with the code, and a PR built in a worktree off `origin/main`. **The long route** charts the decisions first, settles them into specs, slices the specs into build tickets, then builds. **The short route** is `/quick-change`, which skips the map and goes straight from issue to PR, and earns that only when nothing is left to decide.

This skill is reference. It runs no stage. Nearly every stage is `human-only`, so you name the **door** and the human opens it.

## Which route

**Quick** when all four hold:

- **Decided.** A few direct questions settle whatever is open, and each answer closes its question rather than opening two more.
- **Small.** One session builds it; one PR a reviewer reads in a sitting carries it.
- **Inside the spec.** The specs it touches take an edit, not a new spec or a new ADR, and it contradicts no ADR.
- **Done is nameable.** You can write its acceptance criteria now.

**Long** when any fails. The usual signals:

- **Fog.** The request is "work out how to…", or asking about it grows a design tree: decisions hanging off decisions.
- A new ADR, a new spec, or a new package or assembly.
- A contract other assemblies or repos consume changes shape.
- Several tickets' worth, or work other sessions must coordinate on.

Size the change, not the sentence: read the code and spec it lands in before calling it. The test recommends; the human picks. A human may send a long change down the quick route, and that call goes on its issue per `approval-policy`.

## The stages

Each stage's output is the next one's input. Skip one and the next has nothing to read: `/specs-to-tickets` stops cold on a map with no `## Specs settled`.

| # | Stage | Door | Takes | Leaves behind |
|---|---|---|---|---|
| 1 | **Chart** | `/wayfinder <idea>` | A loose idea | A `wayfinder:map` issue: Destination, fog under **Not yet specified**, child decision tickets (`wayfinder:research` · `prototype` · `grilling` · `task`) wired by blocking edges |
| 2 | **Resolve** | `/wayfinder <map>` | The map's frontier | One decision per session, worked through `grilling` + `domain-modeling` (research tickets fan out as subagents): a resolution comment, the ticket closed, a line in **Decisions so far**. Repeat until the way is clear |
| 3 | **Settle** | `/decisions-to-specs <map>` | Resolved decisions | Spec files and ADRs, written in a worktree and opened as a PR; each decision ticket stamped superseded; `## Specs settled` on the map |
| 4 | **Slice** | `/specs-to-tickets <map>` | Settled specs, once their PR merges | `ready-for-agent` build tickets via `to-tickets`, one spec at a time, linked under the map as sub-issues, with blocking edges; `## Tickets` on the map |
| 5 | **Build** | `/implement <ticket>`, or an unattended mode (below) | A build ticket | Code, tests and spec edits committed on a branch in a worktree of its own |
| 6 | **Land** | `/create-pr-for-branch` → `/review-pr-in-worktree` → `/squash-merge-and-clean-up` | The branch | A merged PR. The build ticket closes on merge, not before |

**Build doors**, by how the human is placed:

- `/implement`: one ticket, human reachable, stops at the commit.
- `/implement-unattended`: human away; naming and decision authority granted, tickets fanned out across subagents, one PR back with every borrowed call tabled. `/implement-unattended-no-subagents` is the same, done serially.
- `/subagent-implement`: several independent tickets at once, one PR each.
- `/tickets-to-lanes`: several unattended sessions over one map, coordinating on one issue.

Borrowed calls are settled afterwards through `/settle-borrowed-authority`; review answers through `/respond-to-pr-review`.

## What holds at every stage

- **One decision per session** on the map (research tickets excepted).
- **Decisions live in tickets until stage 3, then in the spec.** Build from the tree, never from a resolution comment, so no build ticket exists before its spec PR merges.
- **Every repo write happens in a worktree of its own**, off a freshly fetched `origin/main` unless a human chose another base.
- **Approvals go on the tracker** per `approval-policy`; the tree holds the rule, never the conversation.
- **Merging is the human's.**

## Pointing the human at a door

You can't open a `human-only` door yourself, so hand over the line to type:

- **No map yet:** `/wayfinder <the idea>`, or `/wayfinder #<issue>` when an issue already holds the request (a `/quick-change` issue that turned out foggy, say).
- **A map exists:** invoke `read-the-map` on it; it names the next door. `whats-next` turns that into a paste-ready prompt for a fresh session.
- **Many maps:** `/check-wayfinder-maps` surveys them all.

Say which signal from [Which route](#which-route) sent the work here, in one line, so the human can overrule it.
