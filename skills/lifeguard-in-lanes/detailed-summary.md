# Detailed lane summary

`lifeguard-in-lanes` prints this at a **milestone** (a lane opening its PR, or a lane's PR merging) and whenever the human asks for a summary. It replaces that tick's table or line.

On top of what the tick already has, gather two things. From the issue body, take each lane's tickets. For each lane that has posted `DONE`, fetch its PR:

```
gh pr view <pr> --repo <owner/name> --json url,state,mergedAt,mergeable,statusCheckRollup,body
```

## The format

**Lanes on map [#<map>](<map url>) — <time> — <milestone, or "on request">**

| Lane | State | PR | Merge status |
|---|---|---|---|
| 1 — stack | swimming, busy | — | — |
| 2 — image and config | done | [#712](<url>) | open · checks green · mergeable |
| 3 — director | waiting on H1 | — | — |
| 4 — host | done | [#715](<url>) | merged 16:02 |

- **Lane 1 — stack:** [#676](<url>) open · [#677](<url>) open · [#678](<url>) open · [#680](<url>) open
- **Lane 2 — image and config:** [#667](<url>) in PR · [#673](<url>) in PR · [#685](<url>) unfinished

Give every ticket in each lane, in the issue's build order, linked by number and without its title.

**Ticket states:**
- `open`: the lane has no PR yet.
- `in PR`: the lane's PR body has a `Closes` line for it.
- `unfinished`: the lane's PR body names it as not landed.
- `closed`: closed on the tracker, normally by the merge.

**Merge status:**
- `open · checks <green|red|pending> · <mergeable|conflicting>`
- `merged <time>`
- `closed unmerged`

If a stacked lane's PR turns `conflicting` after a sibling merges, add one line under the list naming it. That PR needs `origin/main` merged in, and that's the human's call.
