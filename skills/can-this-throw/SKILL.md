---
name: can-this-throw
description: "Answer whether code can throw in a way nothing catches. Traces every throw to its catch and continuance, or to where it fails the app, and gives one verdict: (a) caught with continuance, (b) can fail the app, (c) other. Use when asked whether something can throw or crash, and before putting any code decision to a human."
argument-hint: "The code: a PR, a diff range, a file:line or method, or a borrowed case's site"
metadata:
  type: command
  invocation: model-discoverable
  applies-to: [code, exceptions, review, decisions, approvals]
---

# can-this-throw

A human deciding about code keeps needing one answer: **can this throw in a way nothing catches?** This skill answers it so the human never has to ask. The answer is a **verdict**, one of three, backed by a trace of every throw.

**The verdict has to come from a trace.** "It's inside a try/catch somewhere up the stack" is not evidence. A verdict of (a) holds only when every throw source has been followed through **every** call site to a catch.

## 1. Fix the scope

Decide which code the verdict covers: a PR, a diff range, one method, or a borrowed case's site plus the code its call produced. State it in one line.

**Done when:** the scope is one stated set of code.

## 2. List the throw sources

List everything in scope that can raise:

- an explicit `throw`;
- a call whose contract throws, such as parsing, IO, casts, collection access, argument checks or disposal;
- an awaited task that can fault;
- a callback or handler the code registers, which will run on someone else's stack;
- a constructor or static initialiser.

**Done when:** every source has its own line.

## 3. Trace each one to a catch or a boundary

Walk outward through every caller, not just the one you expect. Stop at the first catch that handles the throw, or at a **boundary**, where none of our code is left above it:

- the process entry (`Main`, or the top-level script);
- a thread's entry, or a thread-pool work item;
- a framework callback or event handler, such as the UI loop, a timer or a message consumer;
- a fire-and-forget task: `async void`, a discarded or unawaited `Task`, a `Task.Run` nobody observes, or an unhandled promise;
- a static initialiser, which surfaces later as a type-initialisation failure, or a finalizer.

**Two traps.** A synchronous throw before the first `await` in a fire-and-forget method escapes straight to its caller, past any guard wrapped around the awaited work. And a catch that logs and rethrows isn't a catch.

**At a catch,** name the **continuance**: what the program does next and what state it is in, whether that's a retry, a fallback, a default or a degraded mode. A catch that swallows the exception and leaves state half-changed has no continuance.

**At a boundary,** name what reaching it does: the process exits, a thread the app needs dies, the exception is dropped unobserved, or the host decides.

**Done when:** every source ends either at a named catch with its continuance, or at a named boundary with its effect.

## 4. Give the verdict

The verdict is one of three:

- **(a) caught on every path, each with a continuance.** Every source ended at a catch that says what happens next.
- **(b) can fail the app.** At least one path reaches a boundary that ends the process or kills a thread the app needs. Name that path.
- **(c) other.** State what it is. For example: a catch whose continuance leaves a stuck or half-changed state; an exception dropped unobserved; behaviour that depends on host configuration; or a question the code can't settle, such as reflection or a foreign callback, in which case say what is unknown.

**Mixed results take the worst.** Any (b) path makes the verdict (b). Otherwise, any (c) makes it (c).

```
Throws: (a) caught on every path, with continuance | (b) can fail the app | (c) other — <state>
- <source> (<file:line>) → caught at <file:line> → continues by <…>
- <source> (<file:line>) → escapes through <path> → <what it does to the app>
Not traced: <code outside the scope that a source depends on, or "nothing">
```

**Done when:** the verdict line comes first, every source from step 2 has its own line, and **Not traced** is filled in.

## Where it is used

`/settle-borrowed-authority` puts this verdict at the top of every code case before asking it. Whenever else a code decision goes to a human, such as a review item or a question about behaviour, give the verdict first, so the question never has to be asked. Naming decisions don't need it.
