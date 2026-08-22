---
name: decisions-to-specs
description: Settle a wayfinder map's decisions into durable spec files in the repo — updating existing specs or writing new ones, placed by locality, as ADRs where the decision earns it.
argument-hint: "The wayfinder map (issue URL or number)"
disable-model-invocation: true
---

# Decisions → Specs

`/wayfinder` charts decisions as GitHub issues — the conversation, captured at that moment. Those issues are the **record**; they are not where a decision *lives*. This skill takes a wayfinder map and settles its resolved decisions into **durable spec files in the repo**, so the next step — `/specs-to-tickets` — has something concrete to slice.

Read GitHub for the decisions; write the repo for the specs. The two surfaces stay split on purpose (see [Split surfaces](#split-surfaces)). This skill is the bridge, and only the bridge — it never creates issues, never moves specs onto the tracker.

**Settling a decision supersedes its ticket.** The issue is a snapshot of one conversation; from there the decision lives in the spec — and only the spec moves as later decisions, review, and implementation refine it. A decision ticket read months later is therefore likely stale, so step 6 stamps each one with a **superseded** banner pointing at its spec. The issues stay closed as the rationale trail; the specs are what `/specs-to-tickets` slices into implementation tickets.

## Process

### 1. Load the map and its decisions

Resolve the map from the argument (issue URL or number). If none was given, find the open issue labelled `wayfinder:map`. Read its body — **Destination** and **Decisions so far**. For every decision listed, fetch its child ticket and read the answer `/wayfinder` recorded on close — its resolution comment.

**Done when:** every resolved decision in Decisions-so-far is in hand with its answer. A decision with no durable consequence (a dead end, an explicit "no change") is dropped with a note — not forced into a spec.

### 2. Map the existing spec landscape

Before writing anything, learn where specs already live, so decisions **update rather than duplicate**:

- package/assembly `spec/` folders (and any spec sitting closer to the code)
- root `spec/` — cross-cutting specs
- `spec/adr/` — existing ADRs (the ADR folder is a subfolder of the spec folder, at whichever `spec/` scope owns the decision)

Read `CONTEXT.md` and any ADRs touching the areas these decisions affect. Use the glossary's vocabulary in everything you write; if a decision contradicts an ADR, surface it rather than silently overriding.

**Done when:** for each decision you can name the spec file it belongs in, or state that none exists yet.

### 3. Route each decision — confirm before writing

Apply the [Placement rules](#placement-rules-locality-ladder) to give each decision a home: an existing spec to update, a new spec to create, or an ADR. "Closer to code if it makes sense" is a judgment call — present the full routing table (decision → target file → new/update) and get the user's confirmation **before writing a single file**.

**Done when:** the user has approved a target for every decision.

### 4. Write and update

- **Update an existing spec:** weave the decision into the relevant section. Don't restate the conversation — the issue holds that; the spec holds the settled decision.
- **New spec:** use the [New-spec shape](#new-spec-shape) so `/to-tickets` consumes it the same way.
- **ADR:** use the [ADR shape](#adr-shape).

Every decision links back to its source GitHub issue — the spec is the *what*, the issue is the *why it was decided*.

**Done when:** every routed decision from step 3 is reflected in a file on disk.

### 5. Tighten with `/concise-copy`

Run `/concise-copy` over every file you touched. Specs are read repeatedly — earn their length.

### 6. Stamp each decision ticket as superseded

Go back to every decision issue that fed a spec and prepend this banner to its **body** — replacing an existing one if you're re-running:

```markdown
> **Superseded by spec — do not build from this ticket.**
> This decision now lives in `src/<Package>/spec/<feature>.md`, which is the source of truth and has almost certainly moved on since this was written. Below is *why* the decision was made; build from the spec.
```

Name every spec the decision fed. A decision dropped in step 1 gets no banner — nothing superseded it.

**Done when:** every decision issue routed in step 3 carries the banner, naming its spec file(s).

### 7. Update the map, then report

Write an overview of what you did back onto the **map body**, under a `## Specs settled` heading (see [contract](#the-maps-specs-settled-section)). This is the durable guide the next step — `/specs-to-tickets` — reads; the map stays the single place that shows where the effort is.

Then report to the user: what landed where, which decision issues were stamped, and that `/specs-to-tickets <map>` is the next step.

**Done when:** the map has a `## Specs settled` section listing every file written in step 4.

## The map's Specs-settled section

The handoff to `/specs-to-tickets`. Write it onto the map body (create the heading if absent; refresh it if re-run). One line per spec file — path, disposition, and the decision issues that fed it:

```markdown
## Specs settled
<!-- written by /decisions-to-specs -->
- `src/<Package>/spec/<feature>.md` — new — decisions: #12, #15
- `spec/<cross-cutting>.md` — updated — decisions: #13
- `src/<Package>/spec/adr/0003-<slug>.md` — ADR — decision: #14
```

Disposition is `new`, `updated`, or `ADR`. ADR lines are recorded but flagged so the next step knows **not** to slice them.

## Placement rules (locality ladder)

Prefer the **tightest scope that fully contains the decision**:

1. **Package/assembly `spec/`** — the decision affects one package. It may sit even closer to the code (beside the module) when that reads better.
2. **Root `spec/`** — cross-cutting: touches multiple packages, or the system's shape.
3. **`spec/adr/` (ADR)** — when the decision is **hard to reverse**, **surprising without context**, and **the result of a real trade-off** (all three, per Matt's ADR criteria). Fewer than three → not an ADR. The ADR folder nests under the `spec/` at the same scope as the decision: a package-scoped ADR goes in `<package>/spec/adr/`, a cross-cutting one in the root `spec/adr/`.

**Spec vs ADR:** a spec describes *what to build*; an ADR records *a decision and its rationale*. A decision can produce both — an ADR for the "why", a spec for the "what" — but split them only when each carries weight alone.

## New-spec shape

Mirror `/to-spec`'s template so `/to-tickets` reads it the same way. Trim any section a decisions-derived spec doesn't need:

- **Problem Statement** — from the user's perspective
- **Solution** — from the user's perspective
- **User Stories** — `As an <actor>, I want <feature>, so that <benefit>`
- **Implementation Decisions** — modules, interfaces, contracts, schema (no file paths / code)
- **Testing Decisions** — seams, what a good test covers
- **Out of Scope**

## ADR shape

Per the suite's ADR format: numbered `spec/adr/NNNN-slug.md` at the owning scope (scan that folder for the highest number, increment). Title + 1–3 sentences: context, decision, why. Add `Status` / `Consequences` only when they earn it. An ADR can be one paragraph — its value is recording *that* a decision was made and *why*.

## Split surfaces

GitHub holds the conversation (wayfinder maps + decision issues) and, later, the implementation tickets. The repo holds specs. Keeping them apart is the point: the record doesn't bloat the repo, and the specs aren't buried in an issue tracker.

## Where this sits in the flow

`/wayfinder` → GitHub **decision** issues (closed, rationale, stamped superseded) → **`/decisions-to-specs`** → repo **spec files** (+ `## Specs settled` on the map) → `/specs-to-tickets` → `/to-tickets` → GitHub **implementation** tickets (fresh, distinct from the decision issues) → `/implement`.

Optional: wire the terminal step into a wayfinder map's `## Notes` — *"when the route is clear, run `/decisions-to-specs`"* — or run it by hand whenever decisions have piled up enough to settle.
