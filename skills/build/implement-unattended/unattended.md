# Working unattended

Shared by `implement-unattended` and `implement-unattended-no-subagents`. They differ only in whether the building is fanned out. Everything about working with nobody there is here.

## What unattended means

The human invoked the skill and left. For the length of this session there is **no one to answer a question**, which changes two things and nothing else:

- **You do not stall.** A call that would normally park the work gets made, marked, and carried to the PR.
- **You do not overreach.** The calls that were never safe to make alone are still not safe to make alone, and "nobody was around" is not the reason they become safe.

Everything else holds exactly as it does when the human is present. The specs are still the specs. The ticket is still the plan. The gate still has to be green.

**Unattended is not autonomous.** Nothing merges, nothing publishes, nothing leaves the repo. The session ends with work a human reviews, and the whole design is aimed at making that review possible for someone who was not there.

## The grants this mode carries

Both, at once, for the whole session:

- **Naming** — the terms in `grant-naming-authority`.
- **Decisions** — the terms in `grant-decision-authority`, whose **What may be borrowed** section is the substance of this mode. Read it. Its in-bounds list is what keeps you moving; its out-of-bounds list is what keeps you honest.

> Those skills are `human-only`, and the human invoking *this* skill is the invocation of both. **Their gate paragraphs are satisfied — do not stop and ask.** Read them for their terms, which is what you are being pointed at.
>
> From here, installed flat, they are `../grant-naming-authority/SKILL.md` and `../grant-decision-authority/SKILL.md`. In the `mtngtools/agents` repo they are `skills/general/grant-naming-authority/SKILL.md` and `skills/general/grant-decision-authority/SKILL.md`. Whichever resolves.

The mechanics of holding a loan — the verbatim markers, the `grep -rn "TEMPORARY AGENT"` ledger, the PR table, the reviewer note, the merge gate — are the **borrowed-authority ledger**, at `../grant-decision-authority/ledger.md` installed, or `skills/general/grant-decision-authority/ledger.md` in the repo.

**Run the ledger's grep before you start.** Existing markers mean a previous session left loans standing; they go in your PR table too, and a marker you did not make is not yours to remove.

## Park, don't stall

A call that lands out of bounds does not stop the session. It stops **that call**:

1. Do everything in the work that does not depend on it.
2. Write down the question, what you would have chosen, and why you did not choose it.
3. Carry on.

**Parked items are invisible to the grep** — nothing was written, so nothing is marked. They live in your handback and in the PR, under the table, as their own list. That is the only place they exist, which is why the handback is not optional.

**A parked call that blocks the whole ticket ends the ticket, not the session.** Say so plainly, move to the next thing you can make progress on, and put the ticket in the handback as blocked with the reason.

## When to stop

Unattended sessions fail by grinding, not by quitting. Stop and write the handback when any of these is true:

- **The work is done** — criteria met, gate green, committed.
- **The gate has failed the same way three times** and your fixes are not converging. Three attempts is enough evidence that the failure is not the one you think it is. Commit what is sound, park the rest, and hand back the actual output — not your theory about it.
- **Everything left is parked.** There is no more independent work; continuing means reaching into what you parked.
- **You reached something out of bounds that you cannot route around** — a false premise in the ticket, a spec the work contradicts, a gate you would have to disable. These are handbacks, and they are the most valuable thing an unattended session produces.

**Do not wait, poll, or schedule a re-check.** The human is away; there is nothing to wait for.

## The handback

The last thing the session does, written for someone who was not here and will read it cold. In this order:

1. **What got done** — per ticket: the branch, the base, criteria met, and the gate result you actually saw.
2. **What you borrowed** — the ledger's table, generated from the grep, every row with its alternative and reason. Say plainly that none of it is approved.
3. **What you parked** — each question, what you would have chosen, and what it blocks.
4. **What you could not finish** — blocked tickets, with the reason.
5. **What you dropped** — anything capped, sampled, skipped, or a subagent that came back empty. A silently truncated sweep reads as complete coverage.
6. **The one thing you would ask if you could ask one thing.** Put it last, on its own. It is usually the thing that decides what the next session does.

The same content goes on the PR, per the ledger, so it survives the conversation. **The handback is a report, not a record** — the PR and the issue are the record, and nothing about this process gets committed to the tree beyond the markers, which come out at settle-up.

## What is still not yours

No merge. No push to `main`. No release, tag, or publish. No closing a ticket, no dropping an acceptance criterion, no relaxing a gate. No opening a PR against another repo.

Opening **this** work's PR is fine and is the point — that is where the table lands and where the human answers it.
