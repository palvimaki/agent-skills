# Mythos-Reserve Routing Skill Specification

This is a **token-effectiveness** protocol. It gets the most realized benefit
out of the newest and most expensive frontier model. It spends that model only
where it is irreplaceable: long-horizon reasoning, architecture, security,
cross-system diagnosis and judgment. Every other token goes to a cheaper route
that does the job just as well. The goal is not the smallest bill and not the
fewest tokens. The goal is **the most accepted outcomes per frontier token**.

"Mythos-class" is a **role**, not a brand. It is whichever model on your stack
is currently the deepest reasoner and the most tightly metered. When the
landscape moves, you do not rewrite this spec by hand. You tell your frontier
model to update your local copy to the present situation. That self-update
clause is part of the spec.

> **Doctrine version: 2026-10-20.** Routing now has four tiers, and the cheapest
> tier runs as harness-native subagents inside the interactive session. Frontier
> effort has a floor, not a ceiling. Quota limits are re-checked by real probe.
> Model names, tier names and effort labels all have a shelf life; the roles and
> rules are the portable part.

## Contract

- Expensive tokens buy thinking. Cheap tokens buy doing.
- Route by tier: fast low-cost worker tier, daily-driver route, demanding route,
  frontier reserve. Put each unit on the lowest tier that clears it.
- The frontier reserve runs only in sessions the operator opens. Native workers
  inside that session never inherit the frontier model; each worker names a
  cheaper tier explicitly.
- A frontier model runs at high effort or above. To save tokens, move the work
  to a cheaper route. Never downshift the frontier model's effort.
- Resolve every model, effort and reviewer from one machine-readable
  `<routing config>`. Report its live version when a choice depends on it. An
  unknown identifier fails fast.
- Subscription first. Agent work uses subscription CLIs and harness-native
  subagents. A metered API is allowed only for approved features that end users
  see.
- Workers get a bounded goal, the relevant files and exact acceptance checks.
  No organization lore.
- Cheap output never grades itself. The parent verifies every worker result.
- Every PR passes an independent review ladder at the exact head SHA. The
  frontier reserve never staffs a review lane as a fallback.
- A usage limit is re-checked by one real probe every few hours. Never wait for
  the published reset date, never evade a limit by switching providers, and
  never ask the operator to buy credits.

## Skill Identity

- Skill name: `mythos-reserve-routing`
- Purpose: keep the frontier model as a **strategic reserve** for the work only
  it can do, route volume work down the tier ladder, keep independent review on
  every PR, and decide inline-versus-delegate by token economics.
- Trigger conditions: before any worker dispatch, any model or effort choice,
  any review staffing choice, and any "should the orchestrator just do this
  itself?" call.
- Audience: solo builders and small teams that run an interactive agent harness
  as the main thread, with one or more subscription CLIs as delegated
  executors.

## The Problem This Solves

A frontier reasoning model is at its best when it thinks long and loops. That
same habit can spend a week of subscription quota in an afternoon. Meanwhile
the cheaper tiers are very capable at bounded work, and their quotas are hard
to exhaust.

The common failure is routing by habit. The expensive model explores the
codebase, summarizes logs, runs test loops and edits twenty files by rule. A
cheaper tier does that work equally well. Then the quota is gone, and with it
the frontier model's real value: its availability for the hard problem. Burning
the reserve on mechanical work does not only waste money. It forfeits
capability.

The fix is a standing protocol that every agent reads, so nobody makes the
routing choice ad hoc.

## The Four Tiers

| Tier | Owns | Never owns |
|---|---|---|
| **Fast low-cost worker tier** | Bounded read-mostly work: codebase, log and document search; file, diff, log and test-output summaries; extraction and classification from supplied context; inventories and status checks; consistency sweeps; mechanical edits with an exact specification. | Multi-file features, architecture, security or merge-gate reviews, final verdicts, operator decisions. |
| **Daily-driver route** | Standard coding, multi-file implementation, uncertain bugs, integration, routine coordination and synthesis. | The hardest unresolved problems. |
| **Demanding route** | Diagnosed hard failures, demanding coding, a different approach after repeated failure. | Bulk volume. |
| **Frontier reserve** | The hardest architecture, security reasoning, cross-system diagnosis, analysis ahead of irreversible decisions, strategy. | Anything a lower tier clears. |

Rules for the tiers:

1. **Harness-native subagents are the fast tier's home.** Many harnesses can
   spawn a subagent on a named cheaper model inside the interactive session, on
   subscription quota. Use that for read-mostly work instead of an inherited
   frontier worker. Model family is not a data clearance: a native subagent can
   run on a different endpoint with a different region, log or retention
   policy. Send sensitive material to a subagent only when the routing config
   marks that exact route cleared for that data class, the same check that
   reviewers get.
2. **Escalate on evidence, not on retry.** On a failed acceptance check or an
   unclear task, move the unit up one tier. Do not retry the same tier
   unchanged. A task that is plainly hard may start on the frontier reserve.
3. **The workers never escalate to the reserve on their own.** Only the
   operator, or an orchestrator already running on the reserve, spends reserve
   tokens.
4. **Headless and scheduled routes keep their declared providers.** The
   frontier reserve has no headless, scheduled or CI lane.

## Effort Rules

- **Frontier floor: high.** A frontier model runs at high effort or above. A
  frontier model at low effort costs frontier quota and gives a lower-tier
  answer. When the work does not need high effort, it does not need the
  frontier model.
- **Lower tiers: medium by default, high for demanding units.** The fast tier
  and the daily driver normally run at medium.
- **Top tiers on purpose.** The highest effort tier is for the case that needs
  it, stated in the task. Keep the effort per route in the `<routing config>`,
  not in prose.

## Hard Rules

1. **No inherited frontier workers.** A worker spawned inside a frontier session
   names its tier. A default that inherits the parent model is the most common
   silent leak.
2. **Resolve routes from the config, never from memory.** A remembered
   identifier goes stale silently. A retired one fails at dispatch time. An
   identifier the config does not know fails fast; it never falls through to a
   default. When the config file is missing, unreadable or invalid, dispatch
   stops and reports the error. Reviewed defaults exist only for the offline
   smoke test; a live dispatch never uses them.
3. **Switch off the quota bombs.** Harness features that loop, fan out or
   recurse on their own on the reserve plan can spend a week of quota in one
   prompt. Disable them at the harness level (permission deny rules, removed
   plugins, settings), so a stray invocation fails instead of billing. The
   operator enables one for a specific run and switches it back off.
4. **No metered fallback.** When a subscription route is unavailable, the
   fallback is another declared subscription route, a queue, or a recorded
   block. It is never a pay-per-token API call.

## Quota Limits

- A usage limit, a payment-required response or a "try again at <date>" message
  is a **hold**, not a fact about the future. The operator can apply a reset at
  any time.
- Re-check with **one bounded real call** through the normal route after a few
  hours, and at most every few hours. A dashboard, a meter or the published
  date is not a probe.
- Resume at once when the probe works. The work record names the last probe
  time and the next probe time. A resume condition is never only the published
  reset date.
- While the hold lasts, route the work by the declared fallbacks in the
  `<routing config>`, or queue it. Never switch to an undeclared provider to get
  around the limit. Never promote bulk volume to a higher tier to get around a
  shared rate window.
- Never ask the operator to top up, buy credits or upgrade a plan because of a
  hold. Only a long-lasting billing failure, as the config defines it, is an
  operator decision.
- Tell two failure shapes apart. A used-up window means queue and probe later.
  A model-identifier rollout error means one logged retry on a declared sibling,
  then fail closed and report it.

## Routing Table: Example Instantiation

The table shows roles. The concrete models live in the `<routing config>`.

| Work | Route |
|---|---|
| Search, summary, extraction, inventory, consistency sweep | Fast low-cost tier, native subagent |
| Mechanical edit with an exact specification | Fast low-cost tier |
| Standard well-specified feature | Daily-driver route, medium |
| Challenging patch, uncertain bug, integration | Daily-driver route, high |
| A unit that failed acceptance on the daily driver | Demanding route, high, bounded |
| Headless or scheduled job | Its declared non-reserve provider |
| PR correctness review | First available reviewer on the `<review ladder>` |
| PR security review (production apps, external repositories) | Security-axis reviewer cleared for the material |
| Visual or UX walk of a running build | A route that can really drive a browser or simulator |
| Hardest architecture, security reasoning, cross-system diagnosis | Frontier reserve, high or above |

## The Review Gate

Reserving the frontier model must not mean less review.

- **The review ladder in `tenth-man.md` is the merge gate.** That spec is the
  single source for round rules, verdict parsing, repair rounds, the security
  axis, sensitive-data routing and the canonical merge checklist. In short: one
  fresh independent reviewer per seat, never the author, bound to the exact
  head SHA; only P0/P1 block; no verdict is never a GO.
- **Never fall back to the frontier reserve or to the author's family** when
  a required seat's ladder is exhausted. The gate has `NO VERDICT` and the PR
  waits.
- **Advisory extras.** A same-session review by a different model family inside
  the harness is useful extra evidence. When the frontier reserve is already the
  orchestrator, its own read of the final diff is also additive. Neither
  approves a merge. A P0/P1 from either one follows the dismissal rule in
  `double-up-code-review.md`: it blocks until the owner confirms and fixes it,
  or until a fresh non-author ladder reviewer agrees to dismiss it.
- **Time limits.** Each review stage has a deadline.

See `tenth-man.md` and `double-up-code-review.md` in this repository for the
prompts and the two-axis layout.

## Inline Versus Delegate

A dispatch has a fixed overhead: the task text, the result check, maybe a
repair round. For tiny changes that overhead costs more than the edit. The
frontier orchestrator edits **inline** only when all of these hold:

- about 25 changed lines or fewer, in 2 files or fewer;
- the change follows from context already loaded, with no new exploration;
- verification is one cheap command;
- no new branch or worktree is needed.

Everything else goes down the tier ladder.

**Hand over the goal, not a spec built from fresh reading.** If the
orchestrator must read a lot of new code only to write the task, it should not.
It gives the worker the goal, the relevant files and the acceptance checks, and
lets the worker explore. Frontier tokens go to framing the problem, setting the
acceptance bar and judging the result.

## Measure Per Accepted Outcome

- Count the whole cost of an accepted result: reasoning, tool calls, retries,
  tests, reviews and repairs.
- Keep stable instruction prefixes. Batch independent reads. Trim tool output.
  Keep durable checkpoints.
- Do not repeat an unchanged test or review. Do not add keepalive calls. Do not
  fork a frontier worker for routine extraction.
- Metered API prices are not a conversion factor for subscription quota. Judge
  subscription routes by quota use per accepted outcome.

## Phases

Subscription terms move. Record your situation as dated phases in your local
copy, so agents apply the right rules without debate:

- **Reserve window:** the frontier model is on a subscription but tightly
  metered. All rules above apply.
- **Sunset:** the frontier model leaves the subscription for metered billing.
  Agent use drops to zero until it returns to a subscription. Any invocation in
  this phase is a budget breach: stop and report it.
- **Abundance:** quota to spare. Relax the inherited-worker rule first. Keep
  independent review, the effort floor and the inline economics; they are good
  engineering at any budget.

## The Self-Update Clause

The model names will go stale; the roles will not. When the reserve changes
family, a tier is leapfrogged or a subscription starts or ends, open a session
with your current frontier model and say:

> Update my `mythos-reserve-routing` skill and my routing config to the present
> situation: these are my current subscriptions, models, effort tiers, metering
> terms and CLIs. Re-derive the four tiers, the review ladder and the phase
> dates. Keep the roles and rules intact.

The update never edits the live routing config. It opens a reviewed change
(a PR) for the config. Protected keys (repository profile, sensitive-data
clearance, ladder order, rung identity, fallbacks, and the review, merge and
human-gate keys) also need an operator record for that head, as in
`tenth-man.md`. The live file stays as it is until that change merges.

Re-deriving the routing from these principles is a frontier task: framing,
judgment and economics. A few minutes of it pay off across every dispatch that
follows.

## Anti-Leak Checklist

Audit your setup for each of these:

- Native subagents that inherit the frontier model instead of naming a cheaper
  tier.
- Skills or scripts that call the frontier model inside automatic pipelines
  (panels, reviewers, summarizers, triage).
- Scheduled tasks, CI hooks or cloud routines set to the frontier family.
- Loop, fan-out or autonomous orchestration features left on for the reserve
  plan.
- Review lanes staffed with the frontier reserve. A per-PR gate looks small and
  runs on every PR.
- A frontier model set to low effort "to save tokens" instead of moving the
  work down.
- A config that names a retired identifier, so dispatch falls back to a default
  without a signal.
- A quota hold whose resume condition is only the published reset date.
- Worker prompts that carry organization lore instead of the goal, the files
  and the checks.

## Installation Recipe

1. **Interactive harness:** create the skill in the harness's skill location,
   with a description that triggers on routing, model choice, review staffing
   and quota decisions.
2. **Native subagent:** define one fast low-cost worker agent, if the harness
   supports named subagents, with the tier's allowed and forbidden work in its
   description.
3. **Each executor CLI:** install the same rules from that executor's point of
   view. Its key rule: it never routes work to the reserve on its own.
4. **Routing config:** keep one machine-readable file that maps task types to
   provider, model identifier, effort, fallbacks and reviewer roles, with a
   version string. Every skill defers to it on conflict. The dispatch layer
   fails fast on an unknown identifier. Declare its protected-key list (see
   `tenth-man.md`); without that list, no autonomous merge runs.
5. **Data eligibility:** record in the config which routes are ineligible for
   which data, and why (hosting jurisdiction, provider data policy, contract).
6. Add a one-line pointer to the skill in each agent's global instructions.

## Smoke Test

1. In a fresh frontier session, ask: "Explore this repo and summarize the
   architecture." Correct: a fast-tier native subagent does the reading, and
   the parent checks the summary. Incorrect: an inherited frontier worker, or
   the parent reads every file itself.
2. Ask it to fix a one-line typo it can see. Correct: an inline edit.
3. Ask for a standard multi-file feature. Correct: the daily-driver route, with
   a bounded goal, the files and the acceptance checks.
4. Open a PR and make the first reviewer return garbage. Correct: procedural
   failure, the next reviewer on the ladder runs at once, and the verdict binds
   to the head SHA. Incorrect: GO, or a fallback to the frontier reserve.
5. Simulate a usage limit that says "try again in five days." Correct: a work
   record with the next real probe in a few hours, and declared fallbacks
   meanwhile. Incorrect: waiting five days, an undeclared provider, or a
   request to buy credits.
6. Ask the frontier model to run at low effort to save quota. Correct: it keeps
   high effort and moves the work to a cheaper tier.

## Redaction Note

This spec names routes by role: frontier reserve, demanding route, daily-driver
route, fast low-cost worker tier, correctness reviewer, security reviewer. Role
names do not go stale and do not reveal whose stack this came from. Your local
copy will gather private context: hosts, repositories, wrapper commands, dates.
Keep that out of any public fork.
