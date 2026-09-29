# Detailed lane summary

This is the one format for a detailed lane summary. `summarize-lanes` prints it whenever a human asks. `lifeguard-in-lanes` prints it at a **milestone** (a lane opening its PR, or a lane's PR merging) in place of that tick's table or line. It is read-only: building it writes nothing.

## Gather

A lifeguard already holds the first two items from its tick. Gather the rest.

- **The lanes' issue:** from the body, each lane's area, branch and tickets in build order, the handoffs and the merge order. From the comments: `CHECK-IN`, `CANDIDATE`, `HANDOFF`, `STUCK`, `DONE`, `LIFEGUARD`.
- **The sessions:** `ListAgents` gives each lane's status, and the check-ins name the sessions.
- **Each lane's PR:** use the `DONE` line if there is one. Otherwise run `gh pr list --repo <owner/name> --head round/<map>-<area> --state all --json number,url,state`. Then, for the PR:

  ```
  gh pr view <pr> --repo <owner/name> --json url,state,mergedAt,mergeable,statusCheckRollup,body
  ```

- **Ticket states:** one call over the map's sub-issues covers every ticket:

  ```
  gh api repos/<owner>/<name>/issues/<map>/sub_issues --paginate --jq '.[] | {number, state, html_url}'
  ```

## The format

**Lanes on map [#<map>](<map url>) — <time> — <milestone, or "on request">**

| Lane | State | PR | Merge status |
|---|---|---|---|
| 1 — stack | busy | — | — |
| 2 — image and config | done | [#712](<url>) | open · checks green · mergeable |
| 3 — director | waiting on H1 | — | — |
| 4 — host | done | [#715](<url>) | merged 16:02 |

- **Lane 1 — stack:** [#676](<url>) open (c) · [#677](<url>) open (c) · [#678](<url>) open · [#680](<url>) open
- **Lane 2 — image and config:** [#667](<url>) in PR · [#673](<url>) in PR · [#685](<url>) unfinished

**Lane state** is the lane's session status and what it is doing: `busy`, `idle`, `waiting on H<k>`, `STUCK`, `offline` or `done`. A lifeguard puts its verdict in front, for example `drowning, idle — nudged`.

Give every ticket in each lane, in the issue's build order, linked by number and without its title.

**Ticket states:**
- `open`: the lane has no PR yet.
- `in PR`: the lane's PR body has a `Closes` line for it.
- `unfinished`: the lane's PR body names it as not landed.
- `closed`: closed on the tracker, normally by the merge.

**`(c)`** follows the state of any ticket a `CANDIDATE` line names. It shows the lane announced the ticket as done ahead of its PR. On a ticket still `open`, it is the only sign of progress the tracker can't show. On an `unfinished` ticket, it means the lane announced the ticket and the PR still didn't carry it.

**Merge status:**
- `open · checks <green|red|pending> · <mergeable|conflicting>`
- `merged <time>`
- `closed unmerged`

If a stacked lane's PR turns `conflicting` after a sibling merges, add one line under the list naming it. That PR needs `origin/main` merged in, and that's the human's call.
