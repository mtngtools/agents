---
name: check-wayfinder-maps
description: Survey every wayfinder map on a repo's tracker and report which ones are ready for implementation, which still have decisions open, and what the next move is on each.
argument-hint: "The repo (owner/name), if not the current one"
disable-model-invocation: true
---

# Check Wayfinder Maps

A read-only survey across **all** wayfinder maps on one repo. `/wayfinder` works one map at a time and never steps back to ask *"where is the whole effort?"* — this skill answers that, in a single table: what's decided, what's ready to build, what's still fog, and what's blocking what.

**This skill never writes.** No claims, no comments, no closes, no map edits. It reads the tracker and reports. If the survey surfaces work worth doing, hand it to `/wayfinder`, `/decisions-to-specs`, or `/specs-to-tickets` — don't do it here.

## Process

### 1. Establish the repo — use what you have, ask rather than search

Resolve the repo from what is **already in front of you**, in this order:

1. The argument, if given (`owner/name`).
2. The current working directory's git remote — `gh` infers this automatically inside a clone.
3. Unambiguous existing context — the session has been working in one repo, or the user named one earlier and nothing since suggests otherwise.

If none of those settles it, or two of them disagree, **ask the user which repo, immediately** — one question, as the first thing you do.

The line is between *knowing* and *hunting*. Context already in hand is free and usually right; use it. What's out of bounds is going to look: scanning the filesystem for candidates, walking directories for `.git`, listing the org's repos to pick a likely one. A search that lands on the wrong repo costs far more than a question, and it makes the user wait for the wrong answer.

**Done when:** you have exactly one repo, reached without a search.

### 2. Learn how this tracker expresses wayfinding

Read the repo's issue-tracker doc (`docs/agents/issue-tracker.md`) — specifically its **Wayfinding operations** section — for how *this* repo expresses maps, child tickets, and blocking. Defaults, where the doc is silent:

- **Map** — an issue labelled `wayfinder:map`
- **Child ticket** — a GitHub sub-issue of the map, labelled `wayfinder:<type>`
- **Blocking** — GitHub native issue dependencies; fall back to a `Blocked by: #n` line in the body
- **Claimed** — the ticket has an assignee

**Done when:** you know how to list a map's children and read their blocking state.

### 3. Pull the maps and their children

List every **open** map, then for each, list its children with state, assignee, and labels. Also list the repo's open non-wayfinder issues — the implementation tickets from `/specs-to-tickets` — since a map's readiness depends on whether its specs have been sliced yet.

**Done when:** you have every open map, every child ticket's state, and the open build backlog.

### 4. Read each map body

Read each map's full body. The readiness signal lives there, not in the ticket list — a map with zero open children can still be unstarted fog. Look for:

- **Destination** — what "done" means for this map
- **Notes** — `Ready for tickets`, `Stub map`, `Blocked by #n`, deferral pegs, execution overrides
- **Decisions so far** — what's settled
- **Specs settled** — written by `/decisions-to-specs`; its presence means the map is past deciding
- **Ready for `/specs-to-tickets`** — an explicit slice-me marker, and any *don't re-slice / needs revision* caveats
- **Not yet specified** — the fog. Weigh it: a few additive deferrals is not the same as an uncharted contract surface.

**Done when:** you can state, for every map, whether anything remains to *decide* and whether its specs have been *sliced*.

### 5. Judge readiness

Assign each map exactly one verdict. The two questions, in order: **are the decisions done?** then **are the specs sliced into build tickets?**

| Verdict | When |
|---|---|
| **Yes — already sliced** | No open decisions, specs settled, implementation tickets exist on the tracker |
| **Yes — not yet sliced** | No open decisions, specs settled, no build tickets yet → `/specs-to-tickets` |
| **Yes, with caveats** | Sliced, but existing tickets carry revision notes or a later decision invalidated them |
| **No — needs charting** | Stub map, or fog that no ticket covers yet → `/wayfinder` |
| **No — blocked on #n** | Waiting on another map's output |
| **No — deliberately deferred** | The map's own Notes peg it to a future event; leave it alone |

A map's own Notes outrank your inference. "Ready for tickets" written on the map is the author's judgment; report it as theirs, and flag it only if the fog visibly contradicts it.

**Done when:** every open map has one verdict and a one-line next move.

### 6. Report the table, then what it adds up to

Output the table below, then a short prose section. Refer to every map and ticket **by name**, linked — never a bare `#42`.

```markdown
| Map | Open decision tickets | Ready for implementation | Next move |
|---|---|---|---|
| [<map title> (#n)](url) | <count + named open tickets, or "0 — all closed"> | **<verdict>** | <one line> |
```

Order the table by readiness — buildable maps first, deferred stubs last — so the top of the table is the takeable work.

Follow it with **What that adds up to**: three or four short paragraphs, no bullets-for-their-own-sake, calling out

- how many maps are through their decisions, and which offer the cheapest work
- every genuinely open decision ticket across all maps (usually a very short list) and whether any two are the same underlying question
- **the bottleneck** — the map the most other maps wait on
- **the quiet gap** — a map spun out of another map's charting and then never followed up. These hide well: zero open tickets, so they look finished, but their body is one line

**Done when:** the user can pick their next session off the table without opening a single issue.

## Cost

Reading every map body is the bulk of the work and it is the necessary cost — readiness isn't visible from titles or ticket counts. Do **not** read child ticket bodies; their state, labels, and assignee are enough. Zoom into one only when a map's Notes point at it as needing revision and you can't tell why.

## Where this sits in the flow

Outside the flow, looking down at it. `/wayfinder` → `/decisions-to-specs` → `/specs-to-tickets` → `/implement` all operate on **one** map; **`/check-wayfinder-maps`** reads across all of them and tells you which one to enter, and by which door.

Run it when returning to an effort after time away, when several maps are live at once and it's unclear which is takeable, or before planning a stretch of work.
