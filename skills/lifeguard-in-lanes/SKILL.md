---
name: lifeguard-in-lanes
description: One lifeguard tick over running lanes. Nudges a drowning lane, alerts the human, and closes out at the last merge. Called only by the /loop a human starts from /tickets-to-lanes' prompt; never start it on your own initiative.
argument-hint: "The lanes' agent:communication issue (number or URL); the repo, if not the current one"
metadata:
  type: command
  invocation: skill-callable
  applies-to: [building, parallel, sessions, monitoring, github]
---

# Lifeguard in Lanes

> **Called by `/loop`.** This skill has two callers: the `/loop` a human started with the prompt `/tickets-to-lanes` hands back, or a human typing its name. It is skill-callable only so that loop can fire it. It messages other sessions and notifies the human's devices, so if nobody asked for a lifeguard on this issue, stop.

The lifeguard watches the lanes and never swims. Each run is one **tick**: read the board, judge every lane, act on the ones in trouble, and report: one line, a small table, or a detailed summary at each milestone. The human keeps it running with the prompt `/tickets-to-lanes` hands back:

```
/loop 15m /lifeguard-in-lanes #<issue> in repo <owner/name>
```

**A lifeguard writes exactly three things:**
- a message to a lane (`SendMessage`);
- a `LIFEGUARD` comment on the lanes' issue;
- a notification to the human (`PushNotification`).

Everything else belongs to the lanes or the human, including code, branches, PRs, tickets and the issue's body. A drowning lane gets rescued by its own session or by the human. The lifeguard's job is to make sure one of them knows.

## Each tick

### 1. Read the board

- **The lanes' issue:** its body (lanes, handoffs, merge order) and every comment (`CHECK-IN`, `CANDIDATE`, `HANDOFF`, `STUCK`, `DONE`, `LIFEGUARD`).
- **The sessions:** run `ListAgents`. Its first line is your own name, and each lane's check-in names its session.
- **The PRs:** for each lane that has posted `DONE`, run `gh pr view <pr> --repo <owner/name> --json state,mergedAt`. A lane's first `DONE` and a PR that has newly merged are both **milestones**.
- **On duty:** if the issue has no on-duty line from your session yet, comment `LIFEGUARD on duty — session <name>, every <interval>`. The lanes learn who is messaging them from that line.

**Done when:** for every lane you have its session, that session's status, its last post, and the handoffs it still owes or still waits on.

### 2. Judge each lane

A lane is **swimming** when it is busy, when it is idle while waiting on a handoff that hasn't arrived, or when it has posted `DONE`. Silence on the issue is swimming too. Lanes post only at handoffs and can build for hours between them, and a long gate is the lane's own to judge.

A lane is **drowning** when any of these is true:

- it never checked in, or `ListAgents` shows it offline or missing;
- it is idle, has not posted `DONE`, and waits on nothing still to come. This is usually a permission prompt, or a lane that finished without posting;
- it posted `STUCK`;
- it posted a `HANDOFF` whose SHA is not on its branch. `gh api repos/<owner>/<name>/compare/<sha>...<branch> --jq .status` answers `identical` or `ahead` when the SHA is there.

When one lane's trouble strands another (a `STUCK` waiting on a lane that's offline), the **root** is the lane being waited on. Name the root.

**Done when:** every lane has one verdict and the evidence for it.

### 3. Act

- **First sighting:** `SendMessage` the lane with what you saw, and ask it to post a one-line status on the issue. If it's offline, a message can't reach it, so go straight to the alert.
- **Still drowning at the next tick:** send the alert. `PushNotification` the human with the lane and what you saw, and comment `LIFEGUARD — Lane <n>: <what you saw>` on the issue. Alert **once** per trouble, and alert again only if the trouble changes.
- **Back to swimming** after an alert: comment `LIFEGUARD — Lane <n> back: <evidence>`.
- **Everyone swimming:** write nothing. A quiet tick is the normal tick.

**Done when:** every drowning lane has had the nudge or the alert its history calls for, and nothing was written for a lane that is swimming.

### 4. Report

This session's output is for a human glancing at it, so keep it small. At a **milestone**, or whenever the human asks for a summary, print the [detailed summary](../summarize-lanes/detailed-summary.md) instead, reading that file only then. Otherwise, print the **lane table** when any lane's verdict or handoffs changed this tick, and on every fourth tick:

| Lane | State | Handoffs | Last post |
|---|---|---|---|
| 1 — stack | swimming, busy | waits H1 · owes H4 | CHECK-IN 12:03 |
| 3 — director | drowning, idle — nudged | gave H2 · owes H3 | HANDOFF H2 13:10 |

Use one row per lane and a short phrase per cell. The evidence belongs on the issue, not in the table. On any other tick, print one line, such as `13:45 — all swimming`.

**Done when:** this tick printed exactly one of the three (the detailed summary, the table, or one line), and nothing from earlier ticks was repeated.

### 5. Close out

Once every lane has posted `DONE`, comment the lanes' PRs on the issue in the merge order it gives, and notify the human, because the merges are theirs. Keep ticking after that, since each merge is a milestone.

Once every lane's PR has merged or closed, print the final detailed summary, notify the human, and stop the loop that runs you. For a fixed interval that means `CronList` then `CronDelete`; for a self-paced loop, `ScheduleWakeup` with `stop`.

If you were run once rather than in a loop, finish the tick and tell the human to wrap it in `/loop` for a continuous watch.

## Where this sits in the flow

Beside the lanes. `/tickets-to-lanes` opens the issue and hands back one prompt per lane plus this one; the lanes build, and the lifeguard watches from the edge until the last PR merges.
