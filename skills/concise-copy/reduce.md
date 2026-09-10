# Reduction method

Shared by the `NN-concise` dials. Each dial sets a **target** — the percentage of the original length to aim for. This file is how you hit it honestly.

## Input

Reduce the text the user gives you: an argument, pasted text, a named file, or the current selection. Length is the character count of that source (whitespace included); aim the output near target × original.

## Meaning is the floor, the target is the ceiling

Reduce toward the target, never below the meaning. Every distinct claim, caveat, and instruction in the source survives — the output says the same thing in less space. If the target can't be reached without dropping meaning, **stop at the smallest faithful version** and report the length you actually reached. The target is a goal, not a licence to cut meaning.

## How to cut

Apply the [concise-copy rules](./SKILL.md) — remove filler, lead with the action or state, active voice, fewest words. At tight targets (10–25%) prose editing alone won't reach it; compress structure too:

- Prose → fragments and lists.
- Merge sentences that share a subject.
- Cut examples and repetition before any claim.
- Keep every unique point; delete every second way of saying it.

Order of sacrifice: filler → phrasing → examples → structure. Meaning is never on the list.

**Approval discussion is off the list too.** A passage recording who approved what, or what was considered and rejected, is not compressed to hit a target — see [concise-copy](./SKILL.md#approval-discussion-is-a-removal-question-not-a-trim). Ask about it if the human is at the keyboard; otherwise leave it whole and list it in your output. A tight target is not a reason to shrink a record until it says less.

## Done when

Every distinct point from the source is present in the output, **and** the output is at or below the target — or, where meaning blocked the target, at the smallest faithful length, with the actual percentage reported to the user.
