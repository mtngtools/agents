# The lanes' issue

The template `tickets-to-lanes` fills in step 4. Replace every `<…>`, add one row per lane and per handoff, and drop any friction bullet that doesn't apply. Everything below the rule is the issue body.

**Title:** `Agent communication: <N> <fan-out skill> lanes building map #<map>`

---

## Parent

Map #<map>. This issue is where the <N> `<fan-out skill>` sessions building the map's backlog coordinate. <Human> started them on <date>. It is not a build ticket: the `agent:communication` label marks it, and only <Human> closes it.

## Lanes

| Lane | Integration branch | Tickets, in build order | Waits on |
|---|---|---|---|
| <n> — <area> | `round/<map>-<area>` | #a → #b; #c ∥ #d | Lane <m>'s #x, for #y (H<k>) |
| <n> — <area> | `round/<map>-<area>` | #e ∥ #f; #g | nobody |

Build only your own lane's tickets. A ticket in another lane is taken, even if it looks free. Build every ticket that doesn't wait before you start waiting.

## Handoffs

| | From | Hands off | When | To | Unblocks |
|---|---|---|---|---|---|
| H<k> | Lane <n> | #x, #y | both integrated, gate green; before #z | Lane <m> | #t |
| H<k> | Lane <n> | the whole lane | PR open | Lane <m> | #u |

## Protocol

Each lane runs under `<fan-out skill>`'s *in a lane* section. You push at the handoffs you give, wait on the handoffs you take and on nothing else, and stack those onto your integration branch. The steps below show how.

**Check in.** First, run `ListAgents`. Its first line gives your session's name. Then comment here: `CHECK-IN Lane <n> — session <name>`. The other lanes' check-ins give you the names to message.

**Hand off.** Push your integration branch with an explicit refspec (`git push origin round/<map>-<area>:round/<map>-<area>`). Then comment here, and `SendMessage` each receiving lane, the same line:
`HANDOFF H<k> — Lane <n>: #x #y on round/<map>-<area> @ <sha>, <gate> green`

**Take a handoff.** Run `git fetch origin round/<map>-<area>` and merge that SHA into your integration branch. Branch the tickets that were waiting off your integration branch after that merge. If the sibling's PR has already merged, merge `origin/main` instead. Either way, add a stacking row to your PR's table: *stacked on Lane <n> @ <sha>, handoff H<k> on #<this issue>*.

**Wait.** Watch this issue with `Monitor`, polling its comments every 5 minutes for your handoff line. When a sibling's message arrives, read it with `ReadNotifications`. If the lane you wait on has posted nothing here for 90 minutes and `ListAgents` shows it offline or missing, do three things: comment `STUCK Lane <n> — waiting on H<k>`, message the other lanes, and hand back what you have.

**Finish.** Open one PR for your lane against `main`, normally rather than as a draft. Its body lists every lane it stacks on, with the SHA, and the merge order below. Then comment `DONE Lane <n> — PR #<pr>` and message every lane that waits on you.

**Messages and this issue.** Post here everything another lane needs, and send a message too so a waiting lane wakes. Messages can be missed, so re-read this issue's comments before you wait and before you finish.

## What goes on the map, not here

This issue is for coordination only: check-ins, handoffs, stuck and done. Everything else goes where it would go if you were working alone:
- new fog or a map-level finding: a comment on #<map>;
- a ticket-level finding: a comment on that ticket;
- borrowed and parked calls: your PR.

Don't edit #<map>'s body. With <N> sessions rewriting one body, each loses the others' lines.

## Expected friction

- **Shared spec pages.** <page>'s status line records Lane <n>'s #a and Lane <m>'s #b. Expect conflicts there, and keep both entries.
- **Merge order, for <Human>:** <Lane … first, then …, the tail last>. After each merge, the branches stacked on it merge `origin/main`, so their diffs show only their own lane.
- **Environment.** <a known gate failure that is not the branch's fault, and how to confirm it on clean `main`>
- **Not in any lane:** #<n> (<reason>), …
