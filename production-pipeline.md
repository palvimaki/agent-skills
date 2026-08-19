# Production Pipeline Specification — Spec Front → Two Loops → Ship

This document defines a complete agentic development pipeline: how a human
operator and a set of LLM agents take a feature from idea to shipped code with
exactly one human thinking phase up front and one human judgment axis at the
end. It is intentionally provider-agnostic, model-agnostic, account-agnostic,
machine-agnostic, and repository-agnostic.

An installing agent should be able to read this spec and set the pipeline up
in its operator's environment: install or adapt the constituent skills, wire
the issue tracker, create the label vocabulary, and fill the route placeholders
with locally researched model choices.

> **Doctrine version: 2026-08-19.** Every model name, tier, and effort label in
> this repository is a worked example with a shelf life. Re-verify against
> current documentation before you install.

## Contract

- One human thinking phase up front. One human feel phase at the end. Agents
  own everything between them.
- Every finding becomes a written spec before it becomes code. No exceptions,
  from any review, sweep, or feel pass.
- Review lives inside the agentic loop, on two blocking axes — correctness and
  security. Nothing re-reviews after the operator's green light.
- Sweep observations are raw material, not verdicts. One triage stage verifies,
  de-duplicates, and ranks them before they become a spec.
- Every loop is bounded: 3 sweep iterations, 2 review fix cycles, then surface.
- Every deliverable lands on a durable path before the run reports done.
- The pipeline ships to a development target. Production promotion is the
  operator's own action.
- Resolve routes from the host's routing config, never from memory.

## Skill Identity

Name: `production-pipeline`

Purpose: run feature development as a spec-first pipeline with two nested
iteration loops — an **inner agentic loop** where agents iterate to
correctness (implement → code review → visual sweep → fix spec, repeat), and
an **outer human loop** where the operator iterates on *feel* only (feel pass →
give the spec, repeat). Every finding becomes a spec before it becomes code.
No review follows the operator's green light.

Use this skill when the user asks to:

- set up "the production pipeline", "the spec-front pipeline", or "the
  two-loop pipeline" on a repository or environment;
- establish an end-to-end agentic development workflow where agents run
  autonomously between human touchpoints;
- stop being asked for mid-run code approvals and instead review working
  software.

## Attribution

The **front half** of this pipeline — per-repo setup for tracker, labels, and
domain docs; the relentless-interview ("grill") pattern; conversation-to-spec;
adversarial spec review; ticket decomposition into tracer bullets; TDD at
pre-agreed seams; and two-axis code review — is adapted from **Matt Pocock's
public engineering skills**: <https://github.com/mattpocock/skills>. Treat that
repository as the upstream reference for those stages. This spec states the
pipeline-level contract each stage must satisfy, plus the modifications this
pipeline layers on.

The **endgame** — the nested two-loop structure, the autonomous visual sweep
gate, the integrated re-spec phase, the run-without-stopping rule, the label
ladder (`scoped-sweep`, `host-verify`, `operator-feel`), and the operator
feel-pass handoff — is original doctrine from this pipeline's authors,
published here.

## The Contract in One Line

Two nested loops. Agents iterate to correctness inside — implement, review,
sweep, fix spec, repeat. The operator iterates on feel outside — feel pass,
give the spec, repeat. Findings always become a spec before they become code,
and no review follows the operator's green light.

## Non-Negotiable Constraints

- Do not depend on any specific LLM provider, model, account, API, SDK,
  machine, username, path, repository, organization, or host.
- The operator does exactly two things: the thinking up front (the grill and
  the decisions it surfaces) and the feel/spec cycle at the end. Agents never
  pause between tickets for human code approval; the operator never reads
  diffs as a gate.
- Findings — from any review, sweep, or feel pass — become a written spec
  before any code is written from them. Never findings → code directly.
- Review lives **inside** the agentic loop: every code iteration is reviewed
  before it can reach the operator, and nothing re-reviews after the
  operator's green light. The green light on an already-reviewed passing build
  is the final gate.
- Review is **two blocking axes**, not one pass: correctness and security, each
  its own reviewer with its own scope and its own verdict. A security concern
  demoted to one bullet inside a correctness prompt is a security concern that
  does not get found.
- Work produced by a run is not done until it is **durable**. Worktrees,
  temporary directories, and scratch paths are removed later; a deliverable
  that lives only there is a deliverable the operator will not have tomorrow.
- All iteration is bounded. The sweep loop runs at most 3 iterations before
  stopping and surfacing what still fails. Review fix cycles run at most 2
  iterations before returning to spec or architecture. Nothing loops
  indefinitely, and nothing fails silently.
- The pipeline ships to a development target. Production promotion is the
  operator's own action, outside this pipeline.
- Redact secrets and identifying local details from anything durably written.

## Pipeline Overview

```
SPEC FRONT — the grill is the operator's; the rest is agents
  grill ──▶ /to-spec ──▶ spec review ──▶ /to-tickets
  (research fan-out runs in parallel)        │ ticket set, ready-for-agent
                                             ▼
┌── OUTER HUMAN LOOP — operator iterates here, on feel only ──────────────┐
│                                                                         │
│  ┌── INNER AGENTIC LOOP — no stops, no operator; review lives here ──┐  │
│  │                                                                   │  │
│  │   /implement ─▶ review ×2 axes ─▶ sweep ─▶ triage ─▶ fix spec ┐   │  │
│  │        ▲                                                      │   │  │
│  │        └──────────────────────────────────────────────────────┘   │  │
│  │   until sweep passes · max 3 sweeps · review = 2 axes             │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│         │ PASS                                                          │
│         ▼                                                               │
│   operator feel pass ── feels wrong ──▶ operator gives the spec ──▶     │
│         │ feels right                    (re-enter inner loop)          │
└─────────┼───────────────────────────────────────────────────────────────┘
          ▼
   merge ──▶ dev deploy ──▶ ship
```

## Stage Contracts

### Stage 0 — Repo setup (once per repository)

Before first use, the repository needs three things configured (the upstream
skills include a setup skill for exactly this):

- **Issue tracker** — where specs and tickets live: the platform's native
  issues, or local markdown files for repos without a remote. Recorded in a
  docs file agents can read.
- **Triage labels** — at minimum `ready-for-agent`, plus the three pipeline
  labels defined below.
- **Domain docs** — a domain glossary (e.g. `CONTEXT.md`) and architecture
  decision records (`docs/adr/`), which every later stage reads and respects.

### Stage 1 — The grill

A relentless interview of the operator about the feature: map the design as a
tree of decisions, work the frontier of currently-askable questions **one
question per message**, always leading with a recommended answer the operator
can accept in a word. Facts are the agent's job (dispatch sub-agents to look
things up; never ask the operator for anything discoverable); decisions are
the operator's. The session ends when the frontier is empty — nothing left
silently assumed. Output: shared understanding, plus any glossary entries and
ADRs the decisions produced.

This is the only thinking the operator does up front.

### Stage 2 — Research fan-out (parallel)

For any non-trivial feature, launch parallel research workers with explicit
web-search mandates before the spec is written — a spec built only from
training data is stale by default. Standard bench, one worker each:

1. **Prior art** — how do the best current implementations solve this?
2. **Libraries** — candidates with license, maintenance health, last release;
   adopt/avoid recommendation.
3. **Papers and innovations** — anything newer than the models' training data.
4. **Pitfalls and postmortems** — what did teams that built this regret?

Each brief demands primary sources with URLs and publication dates and forbids
answering from memory. Findings land as markdown files in the repo, and the
spec must cite them. The grill proceeds in parallel; the spec waits for both.

### Stage 3 — /to-spec

Synthesize the conversation (no re-interviewing) into a spec published to the
tracker: problem statement, solution, an extensive numbered list of user
stories, implementation decisions, testing decisions, and out-of-scope. Sketch
the **seams** the feature will be tested at — as high and as few as the
codebase allows — and confirm them with the operator. No file paths or code
snippets (they go stale), except prototype-derived snippets that encode a
decision more precisely than prose.

### Stage 4 — Spec review

The last gate before ticketing. Every downstream step verifies *against* the
spec, so the spec is the one axiom nothing later challenges — this stage
challenges it once, while an error still costs one edit instead of many
implemented tickets.

A **cold reader** — a fresh context with none of the design conversation,
preferably a different model family via the review route — reads only the
spec, the codebase, the glossary, and the ADRs, and judges seven axes:
standalone implementability, internal consistency, codebase truth (claims
checked against actual code), seam placement, testability, ticketability
(decomposes into context-window-sized vertical slices), and scope. Every
finding is a **pasteable edit** with severity `blocking` or `recommended`.
Verdict: GO / GO-with-edits / NO-GO. The operator dispositions findings one at
a time; accepted edits are applied; one round only, no automatic re-review.

### Stage 5 — /to-tickets

Break the spec into **tracer-bullet vertical slices**: each ticket cuts a
narrow but complete path through every layer, is demoable on its own, and is
sized for a single fresh context window. Each ticket declares its **blocking
edges** — the tickets that must complete first. Wide mechanical refactors are
the exception: sequence them as expand → migrate-in-batches → contract.

Quiz the operator on granularity, edges, and merge/split — one question at a
time — then publish in dependency order with `ready-for-agent` applied, using
native blocking links where the tracker has them.

**Ticketing is also where the label ladder is applied** (see below): gates are
declared here, as sequencing decisions, not improvised mid-run.

### The label ladder

Four rungs, from no human involvement to full human judgment:

- **unlabeled** — verified by agents alone; covered by the end-of-feature
  visual sweep only.
- **`scoped-sweep`** — apply when BOTH hold: the ticket completes a
  user-visible flow or visual foundation (navigation shell, layout system,
  design tokens, canvas/map rendering), AND other tickets are blocked on it
  building further on that surface — so a visual defect would compound. The
  ticket gets a scoped visual sweep of just its flows immediately after
  landing, before dependents start; parallel unblocked work continues.
- **`host-verify`** — apply when acceptance needs real hardware a sandboxed
  builder cannot touch: GPU rendering, frame-rate benchmarks, physical input
  devices, audio, simulators. The rule is **sandbox builds, host verifies**:
  the builder ships the build plus a verification-request artifact (an
  executable run script, the acceptance criteria, and the evidence to capture)
  and keeps working the sandbox-verifiable frontier; a host-side agent runs it
  and returns the evidence. Never marked green on build alone; never stalls
  the run. Functional device I/O is `host-verify`; how the device *feels* is
  `operator-feel`.
- **`operator-feel`** — apply only when acceptance can be judged solely by
  human experiential testing: playability, interaction dynamics, gesture feel,
  look, taste. These are the ONLY tickets that pull the operator in
  mid-pipeline, and they are batched into one session wherever the dependency
  graph allows. When in doubt, do not label — the autonomous visual gate
  catches ordinary UX defects.

### Inner agentic loop — /implement

Implement ticket to ticket **without stopping**. The operator does not read
code, and there is nothing for them to judge until the UX exists — so no
pauses between tickets for human review. Use TDD at the pre-agreed seams:
red before green, one vertical slice per cycle, tests at public interfaces
only. Run typechecking regularly, single test files regularly, and the full
suite once at the end. The loop always executes **whichever spec is newest** —
the original, a sweep's fix spec, or the operator's feel spec.

Mid-run gates fire per the label ladder: a `scoped-sweep` ticket landing
triggers its scoped sweep before dependents start; a `host-verify` ticket
queues its verification request; `operator-feel` tickets are surfaced batched
while unblocked work continues. The implementer may *add* an unplanned scoped
sweep when a ticket turns out riskier than specced; it may never skip a
declared one.

### Inner agentic loop — /code-review

Every code iteration is reviewed — including fix-spec iterations — and never
after the operator. Two axes run as parallel fresh-context sub-agents so they
cannot mask each other:

- **Standards** — does the diff follow the repo's documented standards (plus a
  fixed code-smell baseline, repo docs overriding)?
- **Spec** — does the diff faithfully implement the newest spec: anything
  missing, anything not asked for, anything implemented wrong?

Where the environment has an independent review route (ideally a different
model family), it runs as a dissent gate on the same iteration: severity-ranked
findings with a concrete suggested fix each, treated as hypotheses to
cheap-probe, batched into ONE fix spec per iteration, delta-scoped re-review,
at most two fix iterations before returning to spec or architecture.

**The security axis runs alongside, not inside.** A separate reviewer, from a
further family where the stack allows it, reads the same diff with a
security-only scope: exploitability, trust boundaries, auth bypasses, injection
and deserialization paths, races with a security consequence, secret exposure,
and unsafe failure states. It produces its own severity-ranked findings and its
own verdict, kept in their own list. A P0 or P1 on either axis blocks the
iteration. A required reviewer that fails procedurally blocks too — it never
falls back to the author's route, and never escalates to a reserve model the
operator keeps for interactive work. Where a route is ineligible for the
material on data-residency or data-policy grounds, record the skip as a review
limitation; if that leaves a required axis unstaffed, surface it as a block.
See `tenth-man.md` and `double-up-code-review.md` for the prompts.

### Inner agentic loop — the visual sweep

After implement and code review pass on work with any user-facing surface
(web, app, TUI with visual layout — skip for pure backend/infra/docs and say
so):

1. **Make it runnable** — local dev server, dev deploy, or simulator build;
   never production. Capture the entry point.
2. **Dispatch the visual walker route** with the spec and acceptance criteria,
   the entry point, and an artifacts directory. The brief: walk every user
   flow this spec touches, as a user would; drive the UI (browser automation
   or simulator control) and screenshot each meaningful state; judge each
   state against the spec — broken, misaligned, truncated, overlapping,
   unreadable, confusing, or ugly all count; check light/dark and a mobile
   viewport where relevant. Output `VERDICT: PASS` or findings with screenshot
   path, what is wrong, and severity (P0 broken / P1 wrong / P2 polish).
3. **Triage before spec** — sweep output is a pile of *observations*, not a
   verdict, and this is doubly true when the sweep is fanned out across
   parallel segments or flows. One triage stage — a single route with the whole
   picture — de-duplicates observations that are one defect seen from three
   flows, verifies each against the artifact it cites, discards the ones the
   screenshot does not support, and ranks what remains. Skipping triage is how
   a pipeline spends an iteration implementing noise, and how one real P0
   drowns in forty P2 opinions.
4. **Iterate autonomously** — triaged findings become ONE batched fix spec, the
   execution route implements it, both review axes review it, the sweep re-runs
   delta-scoped to the affected flows. **Max 3 iterations**, then stop and
   surface what still fails.
5. **Green = `VERDICT: PASS`.** Only then does the operator enter.

**The re-spec phase is integrated here and is explicit:** findings never go to
an executor raw. The **triage stage** writes the one batched fix spec from the
ranked, verified findings — post-patch expected behavior per finding,
referencing the screenshots — and that spec is what implement executes. The
sweep worker does not write the spec; it produced the observations the triage
stage just filtered, so it is the wrong party to judge which of them survived.

### Outer human loop — the feel pass

The operator enters only after the inner loop is green, and only for what an
agent cannot judge: playability, interaction dynamics, gesture feel, look,
taste — plus any `operator-feel` tickets, batched. The handoff is: the running
build or URL, the screenshot trail, and a one-paragraph "what to feel-test"
note.

- **Feels wrong** → the operator gives the spec. Their words *are* the spec —
  capture them verbatim, sharpen via /to-spec if they want, never invent on
  their behalf — and that spec re-enters the inner loop through /implement,
  where the new code is re-reviewed by the full agentic loop.
- **Feels right** → merge. The green light on an already-reviewed passing
  build is the last gate: merge → dev deploy → ship. Nothing re-reviews after
  the operator.

## Route Roles

The pipeline needs these routes. Discover what is available locally and fill
them after researching current strengths, weaknesses, limits, privacy
constraints, rate limits, cost, latency, and observed performance:

- `<Execution Route>` — implements tickets and fix specs. A capable workhorse;
  volume work, so cost and rate limits matter.
- `<Review Route>` — the independent correctness dissent gate. Strongest
  available independent context; a different model family from the execution
  route when possible.
- `<Security Review Route>` — the blocking security axis. A further family
  where one exists, with a security-only prompt. Required on every iteration
  that produces code, not only on security-flavored tickets.
- `<Triage Route>` — verifies, de-duplicates, and ranks sweep observations
  before they become a fix spec. Needs the whole picture, so it is one route,
  not a fan-out.
- `<Cold Reader Route>` — the spec reviewer. Fresh context is mandatory;
  different family preferred.
- `<Visual Walker Route>` — the sweep worker. Must be able to drive a UI
  (browser automation or simulator control), capture screenshots, and judge
  them against acceptance criteria. Moderate reasoning effort is typically
  sufficient — it follows a script and judges states.
- `<Research Routes>` — cheap, search-capable workers for the fan-out; run
  concurrently.

```text
The following model mapping is an example only. It illustrates the principle and
must be replaced or edited according to current local findings, available tools,
privacy constraints, rate limits, cost, latency, and observed model
performance.
```

Example only: the source environment ran execution on mid-tier models of a
second frontier family via its CLI, the visual walker on that family's top
tier at medium effort, spec and code review on a third family for genuine
cross-family dissent, and research on cheap search-native workers — with the
operator's own interactive sessions on the first family reserved for the grill
and the feel loop.

## Bounds, Watchdogs, and Failure Surfacing

- Sweep loop: max 3 iterations, then surface.
- Review fix cycles: max 2 iterations, then back to spec or architecture.
- Every stage that produces a file the operator is expected to open later —
  spec, review artifact, screenshot trail, report — writes it to a durable path
  before the stage reports done. A path inside a temporary directory or a
  worktree is not durable.
- Long-running dispatches (sweeps, big implement runs) get a bounded watcher
  and a deadline alarm sized to roughly 3× expected runtime — a stalled
  background job must alert the operator, never strand the pipeline silently.
- Anything unresolved at a bound is surfaced explicitly to the operator with
  what passed, what still fails, and the evidence. Nothing is silently
  dropped, and nothing loops past its bound.

## Installation Recipe

For the installing agent, on the operator's environment:

1. **Discover the harness** — where this system stores skills, slash commands,
   prompts, or tool definitions, and what sub-agent and background-dispatch
   facilities exist.
2. **Install the front-half skills** — from the upstream repository
   (<https://github.com/mattpocock/skills>) where usable directly, otherwise
   write local adaptations satisfying the stage contracts above: setup, grill,
   to-spec, spec review, to-tickets, TDD, implement, code review.
3. **Write the endgame skills** — the visual sweep skill and the implement
   augmentations (no-stops rule, label-ladder gates, newest-spec rule) per
   this spec. These are not in the upstream repo.
4. **Run repo setup** on the target repository: tracker choice, triage labels,
   and the three pipeline labels (`scoped-sweep`, `host-verify`,
   `operator-feel`), plus domain-doc scaffolding.
5. **Research and fill the route placeholders** — official docs, release
   notes, model cards, local routing config, local task history, and current
   evaluations. Record the mapping in a local routing doc the skills can read.
   Treat this spec's example mapping as an example only.
6. **Verify capability prerequisites** — the visual walker route genuinely can
   drive a browser or simulator and capture screenshots in this environment;
   the tracker CLI works; sub-agents can run in parallel.
7. **Run the smoke tests** below.

## Smoke-Test Contract

A correct installation passes all of these on a toy repository:

1. Repo setup produces the tracker doc, the triage labels, and the three
   pipeline labels.
2. A short grill session asks exactly one question per message and leads each
   with a recommendation.
3. /to-spec publishes a spec containing all required sections and citing at
   least one research finding (research fan-out may be stubbed for the toy).
4. Spec review runs in a context that verifiably lacks the design
   conversation and returns a verdict with pasteable edits.
5. /to-tickets publishes ≥ 2 tickets with correct blocking edges and applies
   labels per the ladder rules.
6. /implement completes two toy tickets without pausing for approval between
   them.
7. /code-review emits separate Standards and Spec reports from parallel
   sub-agents.
8. The visual walker produces screenshots and a `VERDICT:` line for a trivial
   web page, and the loop demonstrably stops after 3 iterations on a
   contrived always-failing finding.
9. The security axis runs on a code iteration, reports separately from the
   correctness axis, and blocks the loop on a planted injection or
   secret-exposure defect. Making it fail blocks the loop rather than falling
   back to another route.
10. Feeding the triage stage the same defect observed from three flows yields
    one ranked finding, not three, and an observation contradicted by its own
    screenshot is discarded with a reason.
11. Every artifact named in a stage's completion report still exists after the
    run's temporary directories are removed.
12. No secrets or identifying local details appear in any published artifact.

## Redaction Rule

When adapting this pipeline from (or back into) a private environment, strip
organization names, machine and host names, private project names, internal
directive references, and provider subscriptions from anything published.
Publish role names, not vendor names; keep concrete model mappings in local
routing docs, not in the pipeline spec.
