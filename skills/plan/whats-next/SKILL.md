---
name: whats-next
description: Work out the next move on the current wayfinder map and hand back a short, copy-pasteable prompt for the session that does it.
argument-hint: "Optional: the map (number or URL), if the session doesn't make it obvious"
metadata:
  type: command
  invocation: skill-callable
  applies-to: [planning, wayfinder, sessions, prompts]
---

# What's Next

Ends a session by handing over the start of the next one: a prompt short enough to read at a glance and copy without editing.

**Stay on the map.** Unless the current map is finished, the next move is on that same map. Do not offer a different effort, a tempting side quest, or a cleanup that occurred to you along the way — one map at a time is the whole discipline of `/wayfinder`, and this skill is where it gets broken if anywhere.

**The output is a prompt, not advice.** Everything the human needs to act is inside a copyable block. Reasoning that doesn't fit in the block goes in the one-line label above it, or nowhere.

## Process

### 1. Fix the map and the repo

From the argument, the session, or the tracker — in that order, and ask rather than search if none of them settles it. Ask the human which map rather than surveying the repo for one; this skill is for the map you were just on.

**Done when:** you have one map number and one `owner/name`.

### 2. Read the map for its state

Call `/read-the-map`. It owns the checklist, the verdicts, and the verdict → next-move mapping; this skill turns its answer into something pasteable and adds nothing to the judgment.

If it comes back **Done** — everything closed, sliced, and merged — that is the one case where the next move leaves this map. Say so, and let the human pick the next effort.

**Done when:** you have its verdict and the one or two doors it names.

### 3. Cut the choices down

**Two, three at the very most.** A move earns its place by being **blocking** — the most other work waits on it — or **quick** — done in one short session and off the board. Everything else is not next; leave it out entirely rather than listing it as a third-best option.

**Done when:** every move left standing is there for one of those two reasons.

### 4. Hand over the prompt

One block per move, in the [shape below](#prompt-shape). Nothing after the blocks except a single line if something genuinely can't be said inside one.

**Done when:** the human can copy a block and paste it into a fresh session unchanged.

## Prompt shape

```
/wayfinder <map#> <next thing> in repo <owner/name>
```

`<next thing>` is the door `/read-the-map` named — `take #<n> — <title>`, `/decisions-to-specs`, `/specs-to-tickets`, `/prototype #<n>`, `/implement #<n>`. Keep the whole prompt to a line where it can be — the next session reads the map for itself, so a prompt that explains the work is padding.

**If the move writes to the repo** — `/decisions-to-specs`, `/implement`, `/prototype` — end the prompt with the worktree reminder, so it travels with the paste:

```
/wayfinder 157 /decisions-to-specs in repo mtngtools/mtng-dotnet-mono
Work in a worktree.
```

A move that only touches the tracker (`take`, `/specs-to-tickets`) gets no reminder — there's nothing to isolate.

## Labels

One block, no label needed. More than one, each gets a **one-line label above its block** saying why it's on the list:

**Unblocks the other three tickets**
```
/wayfinder 157 take #163 — presentation timing base unit in repo mtngtools/mtng-dotnet-mono
```

**Quick — clears the last decision**
```
/wayfinder 157 /decisions-to-specs in repo mtngtools/mtng-dotnet-mono
Work in a worktree.
```

Labels are for choosing between blocks. They never carry instructions the pasted prompt needs — anything the next session must know goes inside the block.

## Where this sits in the flow

At the seam between sessions. `/read-the-map` says where this map stands; **`/whats-next`** says what to type to act on it. `/check-wayfinder-maps` is the one to reach for only when the question is *which map at all* — a different question from this one.

Pairs with `/are-decisions-from-this-session-saved`: that one makes sure this session left nothing behind, this one makes sure the next one starts without re-deciding where it is.
