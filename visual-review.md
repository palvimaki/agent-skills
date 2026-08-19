# Visual Review Skill Specification

This document defines a reusable skill: **an autonomous visual and UX gate that
walks a running build, judges what it sees against the spec, and drives fix
iterations until it passes — before any human looks at it.** It is intentionally
provider-agnostic, model-agnostic, machine-agnostic, and framework-agnostic.

Any capable LLM agent that can start a build, dispatch a worker with real
UI-driving capability, and read that worker's report can implement it.

> **Doctrine version: 2026-08-19.** The route names and capability assumptions
> below are worked examples with a shelf life. Verify that your chosen walker
> route can genuinely drive a UI and read back what it produced, before you give
> it this role.

## Contract

- Runs after implementation and code review pass, on any user-facing surface.
- Never against production. A local server, a dev deploy, or a simulator build.
- The walker drives the UI as a user would and screenshots every meaningful
  state. A screenshot it did not take is a state it did not check.
- Every finding cites a screenshot. A finding with no artifact is an opinion.
- Findings are triaged, then batched into **one** fix spec. Never findings →
  code.
- Bounded: at most 3 sweep iterations, then stop and surface what still fails.
- Green means an explicit `VERDICT: PASS`. Absence of findings is not a pass.
- The human enters only after green, and only for what an agent cannot judge.
- Screenshots and reports land on a durable path before the gate reports done.

## Skill Identity

Name: `visual-review`.

Purpose: close the gap between "the tests pass" and "it looks and behaves
right", without spending human attention on defects an agent can see for itself.

Use this skill when:

- an implementation with a user-facing surface has passed code review;
- a release gate needs current evidence that the built thing actually works;
- a fix loop needs to verify a visual or interaction defect is really gone;
- an orchestrator has no UI-driving capability of its own and needs to delegate
  the walk.

Skip it — and say so — for pure backend, infrastructure, or documentation
changes. Announcing the skip is part of the skill; a silently skipped gate looks
identical to a passed one.

## The Problem This Solves

Code review reads a diff. Tests assert what someone thought to assert. Neither
sees a control pushed off-screen at a narrow viewport, text truncated in one
language, an unreadable contrast pair in dark mode, a spinner that never
resolves, or a flow that technically works and feels wrong.

That gap has traditionally been filled by the human, which makes the human a
bottleneck on every iteration — and worse, spends their attention on defects any
careful observer would catch. The human's judgment is scarce and irreplaceable
for *feel*. It is wasted on "the button is behind the header".

So: an agent walks the build first. The human enters last, once, on what is
genuinely theirs.

## Required Inputs

- `spec`: the specification or ticket text with acceptance criteria. Required —
  a walk without acceptance criteria produces taste, not findings.
- `entry_point`: URL, launch command, or built application to drive. Required.
- `artifacts_dir`: durable directory for screenshots and the report.
- `flows`: optional. Named flows to walk. Default: every flow the spec touches.
- `surfaces`: optional. Viewports, themes, and locales to check. Default: the
  primary viewport plus one narrow viewport, and both themes where the product
  has them.
- `walker_route`: the route that drives the UI. Must have real capability.
- `max_iterations`: default 3.

## Required Outputs

- `VERDICT: PASS`, or a findings list.
- For each finding: the screenshot path, what is wrong, and a severity —
  P0 broken, P1 wrong against spec, P2 polish.
- The screenshot trail for every state visited, passing or failing.
- A one-paragraph handoff note for the human, written for someone who has not
  read the diff.

## Workflow

1. **Make it runnable.** Local dev server, dev deploy, or simulator build —
   never production. Capture the exact entry point and apply the strip rule to
   it before storing or quoting it: remove known token parameters — `token`,
   share and invite tokens, session query keys — and keep the rest, so deep-link
   parameters the walk actually needs survive. If the walk needs an account, use
   a seeded test account and pass its credentials through the host's secret
   channel.
2. **Dispatch the walker** with the spec and acceptance criteria, the entry
   point, the artifacts directory, the brief below, and — where the walk needs
   authentication — a handle to the host's secret channel. The secret channel is
   a required dispatch input, not an optional extra: a walker given only a
   de-tokenized URL and no account cannot reach authenticated surfaces, and will
   either skip them silently or report a false visual failure. The secrets
   themselves never appear in the brief, the prompt, the report, the logs, or
   the stored entry point. Give the acceptance criteria verbatim; do not
   paraphrase them into something easier to pass.
3. **Triage the output.** Sweep results are *observations*, not verdicts. One
   route with the whole picture de-duplicates observations that are one defect
   seen from three flows, verifies each against the screenshot it cites,
   discards the ones the artifact does not support, and ranks what remains.
   Without this stage a loop spends an iteration implementing noise, and one
   real P0 drowns in forty P2 opinions.
4. **Re-spec, never patch from findings.** The triaged findings become ONE
   batched fix spec, stating the expected post-fix behavior per finding and
   referencing the screenshots. That spec is what the execution route
   implements. Findings never reach an executor raw.
5. **Re-review and re-walk.** The fix goes through the normal code review gates,
   then the sweep re-runs **delta-scoped** to the affected flows.
6. **Bound it.** At most 3 iterations. Then stop, and surface what passed, what
   still fails, and the evidence for both.
7. **Green = `VERDICT: PASS`.** Only then does the human enter.

## The Walker Brief

```text
Walk every user flow this spec touches, as a user would.

Drive the interface — browser automation or simulator control — and capture a
screenshot of each meaningful state into the artifacts directory. Name each
screenshot for the flow and state it shows.

For each state, judge:
- Does it match the spec and its acceptance criteria?
- Is anything broken, misaligned, truncated, overlapping, unreadable, or
  obscured?
- Is anything confusing: unclear affordance, missing feedback, a dead end, a
  state with no way out?
- Does it look finished, or does it look like scaffolding?

Check both themes and a narrow viewport where the product has them.

Output either:
  VERDICT: PASS
or a findings list, one entry per finding:
  - screenshot path
  - what is wrong, in one sentence
  - severity: P0 broken / P1 wrong against spec / P2 polish

Rules:
- Every finding cites a screenshot you actually captured.
- Report what you observed. Do not report what you expect the code to do.
- If a flow could not be reached, say so and say why. An unreachable flow is a
  finding, not a silence.
- Do not edit files.
```

## Route Roles

- `<Visual Walker Route>` — must genuinely drive a UI and read back what it
  produced. This is a capability requirement, not a reasoning requirement:
  verify it with a trivial page before assigning the role. Moderate reasoning
  effort is usually enough — the work is following a script and judging states.
- `<Triage Route>` — needs the whole picture, so it is one route, never a
  fan-out.
- `<Execution Route>` — implements the batched fix spec.

An orchestrator without UI-driving capability does not attempt the walk itself.
It dispatches the walker and consumes the report.

## Scoped Mid-Run Sweeps

A ticket that completes a user-visible flow or a visual foundation — a
navigation shell, a layout system, a component set others will build on — earns
a sweep of **only its flows**, immediately after it lands and before dependent
work starts. A visual defect in a foundation compounds through everything built
on top of it, and finding it three tickets later means re-doing three tickets.

Same dispatch, same fix-spec loop, same bound, smaller brief. Unblocked
independent work continues in parallel. A scoped pass does not exempt those
flows from the full sweep at the end.

## The Human Handoff

The human enters after the gate is green, and only for what an agent cannot
judge: playability, interaction dynamics, gesture and input feel, look, taste,
and anything the team has explicitly labelled as needing their eye.

Hand them three things: the running build or URL, the screenshot trail, and a
one-paragraph note saying what to feel-test. Batch several items into one
session rather than interrupting repeatedly.

**Loop order matters.** Review gates every code iteration *inside* the agentic
loop, so nothing re-reviews after the human's green light. If it feels wrong,
**they give the spec** — their words are the spec, captured verbatim, never
invented on their behalf — and that spec re-enters the loop from the top, where
the new code is reviewed again. If it feels right, their green light on an
already-reviewed passing build is the final gate.

## Bounds and Failure Surfacing

- Sweep loop: 3 iterations, then surface.
- A walker dispatch is a long-running background job: give it a bounded watcher
  and a deadline sized to roughly three times the expected runtime, so a stall
  alerts a human rather than stranding the pipeline silently.
- Before a large dispatch, check the route's remaining rate headroom. A sweep
  that dies halfway leaves an ambiguous result, which is worse than a queued one.
- Anything unresolved at a bound is surfaced with what passed, what failed, and
  the evidence. Nothing is silently dropped.

## Installation Recipe

1. Create a skill or command named `visual-review` in the host's normal skill
   location.
2. Verify a candidate walker route can drive a UI end to end on a trivial page
   and return real screenshots. Assign the role only after that passes.
3. Configure the walker, triage, and execution routes, plus the artifacts
   directory — durable, not temporary.
4. Wire the gate into the pipeline after code review and before any human
   review, and make the skip path for non-visual changes explicit and announced.
5. Implement the bound, the batched fix spec, the delta-scoped re-walk, and the
   watchdog.

## Smoke Test

1. Run the gate on a trivial page that matches its spec. Confirm a
   `VERDICT: PASS` and a screenshot trail.
2. Break the page visually in a way no test would catch — a control behind a
   header, truncated text at a narrow viewport. Confirm it is reported with a
   screenshot and a severity.
3. Plant a contrived always-failing finding. Confirm the loop stops after 3
   iterations and surfaces the state rather than looping.
4. Feed the triage stage the same defect observed from three flows. Confirm it
   yields one ranked finding, and that an observation its own screenshot
   contradicts is discarded with a reason.
5. Run it on a backend-only change. Confirm it announces the skip rather than
   reporting a silent pass.
6. Confirm the screenshots and the report still exist after the run's temporary
   directories are removed.
7. Grep the durable report, the brief, the logs, and every stored entry point
   for credential and token material. Confirm no hits, including tokens in
   captured URLs.

## Redaction

This skill is unusually leaky by construction: it takes credentials, drives a
real interface, and then writes a durable artifact trail. Three separate
surfaces need care.

**Credentials.** Never place a credential, session cookie, bearer token, or
tokenized preview URL into the walker brief, the prompt, the report, the logs,
or a stored entry point. Pass them through the host's secret channel, which is a
required dispatch input whenever the walk needs authentication.

The strip rule, stated once and used everywhere: before storing or quoting any
URL, remove known token parameters — `token`, share and invite tokens, session
query keys — and keep the other query parameters. Preview and share links
routinely carry tokens this way. Do not strip the whole query string: that also
drops the deep-link parameters the walk needs, and it does not catch a token
hiding under an unexpected parameter name. Maintain the token-key list.

**Screenshots.** They leak more than text does: account names, real user data,
tokens rendered on screen, notification content, and whatever else was visible.
Use seeded test data and test accounts, never real ones.

**The artifact set.** Review it before publishing or attaching it anywhere, and
strip host names, absolute local paths, and organization or product identifiers
from anything that leaves the environment.
