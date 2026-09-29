---
name: tickets-to-lanes
description: Split a wayfinder map's build backlog into lanes, open the GitHub issue the lanes coordinate on, and hand back one copyable /implement-unattended prompt per lane.
argument-hint: "The map (number or URL) and the target number of lanes; the repo, if not the current one"
disable-model-invocation: true
metadata:
  type: skill
  invocation: human-only
  applies-to: [building, wayfinder, tickets, parallel, sessions, prompts, github]
---

# Tickets to Lanes

> **human-only.** Start this only when a human asks for it by name. It plans how many sessions to spend on a map, and its handoffs are what let those sessions push, wait and stack on each other's unmerged work. A skill chaining into it would be deciding both on the human's behalf. If you arrived here from anywhere but a human naming it, stop.

One `/implement-unattended` session builds one round and hands back one PR. This skill plans several at once over one map. It splits the map's open build tickets into **lanes** (one session, one integration branch and one PR each), turns every blocker that crosses lanes into a named **handoff**, opens one GitHub issue where the lanes coordinate, and hands back a prompt per lane.

**The output is prompts.** The human pastes each one into a fresh session, and the lanes start there. This session reads the map, writes the issue, and stops.

Each lane runs under the unattended modes' *in a lane* section (`implement-unattended/unattended.md`). That section lets a lane push at the handoffs it gives, wait on the handoffs it takes, and stack those onto its integration branch. This skill decides which handoffs exist, and the issue names them, together with the human who started the lanes and the date.

## Process

### 1. Settle the inputs

You need the map, the repo and the target lane count. Take them from the arguments or this session; the repo defaults to the current one. Ask for whatever is missing, **the map first**. Ask for the lane count in step 3, once the backlog can back a recommendation.

**Done when:** you have one map number and one `owner/name`.

### 2. Read the map and its backlog

Invoke `read-the-map` fresh. Lanes are for a map it calls **Yes — already sliced**, or **Yes, with caveats**, in which case the caveated tickets stay out of every lane. For any other verdict, stop and name the door the map wants instead.

For each open build ticket, gather:

- **Its blockers:** open native edges, plus any gate its body or comments name. Read a named gate against the ticket's acceptance criteria. A ticket whose criteria are complete on `main` is not gated by a decision ticket that only calls it *related*.
- **What it writes:** source files and spec pages, from its body and the spec section it was sliced from.
- **Whether it's in flight:** an assignee, an open PR or remote branch naming it, or a live session holding it (`ListAgents`).

Every ticket goes either into the **pool** with its blockers and files, or **out of every lane** with a reason. The reasons: gated on an open decision ticket, `needs-triage`, or already in flight. Open `wayfinder:*` children are the human's and are always out.

**Done when:** every open child of the map is either in the pool or out with its reason.

### 3. Split the pool into lanes

Apply these rules in order:

1. **Same file, same lane.** Tickets that write the same source file stay together, for example one ticket extending another's class. Two lanes may share a spec page's status line; list that as expected friction.
2. **Chains stay together.** A serial run of blockers stays in one lane. If a lane's later ticket depends on a ticket that is free and small, the lane takes that ticket too, so no handoff is needed.
3. **One tail.** One lane owns the serial run to the map's destination, plus whatever free tickets it can take to start with.
4. **Every lane starts working.** Each lane has at least one ticket it can build immediately.
5. **Fewest handoffs.** Among the splits that pass rules 1–4, pick the one with the fewest cross-lane blockers, then the most even load.

**Check the target count against the graph.** There are too many lanes when one would still start by waiting after rules 2 and 3 have given it every free ticket they can. There are too few when a lane's PR would carry tickets from areas that share no files and no chain. If the graph argues for a different count, ask before writing anything: offer the target and your number, with a one-line reason for each.

Then name everything:

- **Lanes:** `Lane <n> — <area>`, with integration branch `round/<map>-<area>`. List its tickets in build order, `→` for serial and `∥` for parallel.
- **Handoffs:** `H<k>`, one for each cross-lane blocker. Record who hands off, which tickets, when (integrated with the gate green, or the whole lane once its PR opens), to whom, and what it unblocks. A lane builds the tickets it hands off before its other work.
- **Merge order:** lanes that stack on nobody merge first. A lane merges after every lane it took a handoff from, and the tail merges last.

**Done when:** every pool ticket is in exactly one lane, every cross-lane blocker is a handoff, every lane has work it can start, and the count is either the target or the one the human chose.

### 4. Open the lanes' issue

Fill [comms-issue.md](./comms-issue.md), then create the issue on the map's repo:

- **Label** it `agent:communication`, creating the label if the repo lacks it.
- **Attach** it as a sub-issue of the map. Verify with a GET of the map's sub-issues; the POST's response echoes the parent and proves nothing.
- **Human:** the name from `gh api user` or `git config user.name`, and today's date.
- **Gate:** the repo's check command, from its `AGENTS.md`. List any known environmental gate failures as expected friction.
- **Fan-out skill:** `/implement-unattended`, or `/implement-unattended-no-subagents` when the repo's `AGENTS.md` restricts subagents. The issue and the prompts name the same one.

**Done when:** the issue exists, carries the label, appears among the map's sub-issues on a fresh read, and its tables match step 3.

### 5. Hand back the prompts

Give these in order and nothing else:

1. The issue's link, confirmed as a labelled sub-issue of the map.
2. One line on the count: why it fits, or what the human chose over what.
3. The merge order.
4. `Written for: the fresh sessions you'll paste these into — <N> lanes, <N> reviewers, one lifeguard.`
5. One block per lane, under a bold label `Lane <n> — <area> (<k> tickets)`:

```
/implement-unattended #a #b #c in repo <owner/name>
You are Lane <n> (<area>) of <N> sessions building map #<map>. Coordinate on #<issue>: read it first and follow it.
Integration branch: round/<map>-<area>. Work in worktrees.
Waits: #t waits for Lane <m> to hand off #x (H<k>). Build #a → #b first.
Hands off: #x to Lane <m> as soon as it is in and green (H<k>).
```

For a lane that waits on no one, write `Waits: nobody.`. For the tail, write `Hands off: nothing. You are last, and your PR merges after the others.`

6. One block per lane, under **Reviewer <n> — for Lane <n>**, for a separate session that waits for that lane's PR and reviews it:

```
/review-pr-in-lane Lane <n> of #<issue> in repo <owner/name>
```

7. One more block, under **Lifeguard — watches the lanes**, for a separate session that runs `lifeguard-in-lanes` until every lane's PR has merged:

```
/loop 15m /lifeguard-in-lanes #<issue> in repo <owner/name>
```

8. One line: close the issue once every lane is done, because map reads count any open non-wayfinder child as build backlog.

**Done when:** every lane has a lane block and a reviewer block the human can paste unchanged, every ticket in the pool appears in exactly one lane block, and the lifeguard block names the issue.

## Where this sits in the flow

After `/specs-to-tickets` has sliced the map. `/whats-next` hands over one session's prompt; this skill hands over N at once for the same map. Each lane then runs the fan-out skill and ends at its own PR, which its `/review-pr-in-lane` session reviews as soon as it opens, while `/lifeguard-in-lanes` watches from the issue. `/squash-merge-and-clean-up` lands the PRs in the merge order the issue names.
