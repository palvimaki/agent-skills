# Mythos-Reserve Routing Skill Specification

This is a **token-effectiveness** protocol:
a way to get the maximum realized benefit out of the latest, most expensive
frontier model by spending it only where it is irreplaceable — long-horizon
reasoning, architecture, security, hard debugging, judgment — and routing
every other token to a highly capable workhorse model on a second
subscription. Run this way, the frontier model finally gets what it does
best: room to think, loop, and experiment in peace, with its quota intact
when the genuinely hard problem arrives. Budget optimization follows
automatically, as a close second — but the goal is never budget
minimization, and not even token minimization. It is **the most benefit per
frontier token**, across two frontier subscriptions used the way each is
priced to be used.

The protocol is model-agnostic. "Mythos-class" is a **role**, not a brand:
whichever model on your stack is currently the deepest reasoner and the most
aggressively metered. For many builders that is the frontier model inside their
interactive coding harness, with a second subscription's coding model as the
workhorse — and a stack of that shape is used as the worked example throughout.
If the situation reverses, or another family pushes to the front, you don't
rewrite anything by hand: **tell your frontier model to update this skill to
reflect the present situation.** It will take care of it. That self-update
clause is part of the spec.

> **Doctrine version: 2026-08-19.** Every model name, tier name, and effort
> label below is a worked example with a shelf life. The roles, the rules, and
> the economics are the portable part. Re-verify the names against current
> documentation before you install, and re-derive them via the self-update
> clause whenever your subscriptions move.

## Contract

- Expensive tokens buy thinking. Cheap tokens buy doing.
- The reserve model is human-interactive only: no reserve-family subagents at
  any tier, no headless dispatches, no scheduled jobs, no review lanes.
- Volume work goes to the workhorse ladder, on the lightest tier that suffices.
- Resolve every model identifier, effort setting, and reviewer from the host's
  machine-readable routing config. Never from memory.
- Cap reasoning effort at high unless the human states a need for a higher tier
  in that specific case.
- Every PR keeps two blocking review axes — correctness and security — and
  neither runs on the reserve plan.
- A required reviewer that fails procedurally blocks. It never escalates to the
  reserve model as a fallback.
- Exclude a route from sensitive material by hosting jurisdiction and data
  policy, not by vendor reputation. Gate the surface, not the model.
- The orchestrator implements inline only inside the size limits below.

## Skill Identity

- Skill name: `mythos-reserve-routing`
- Purpose: treat the Mythos-class model as a **strategic reserve** whose
  tokens buy only what no other model can deliver; route all volume work to
  the workhorse route; keep a mandatory independent review gate on every PR;
  decide inline-vs-delegate by token economics, not habit.
- Trigger conditions: consult before any subagent dispatch, any model/effort
  selection, any PR-review staffing decision, and any "should the
  orchestrator just do this itself?" call.
- Audience: solo builders and small teams orchestrating an agentic coding
  harness — an interactive frontier session as the main thread, a second
  subscription's CLI as the delegated executor.

## The Problem This Solves

Frontier reasoning models are at their best when you let them think long,
loop, and experiment — which is exactly what makes them able to consume a
week's subscription quota in an afternoon. Meanwhile the workhorse tier
is a seriously capable implementation and review model whose low and medium
effort lanes are hard to exhaust at all — a fact a surprising number of
builders overlook.

The common failure mode is routing by habit instead of by economics: the
expensive model writes boilerplate, explores codebases, runs test loops, and
mechanically edits twenty files — work the workhorse does just as well — and
then the quota is gone, and with it the frontier model's actual value: its
availability for the problem only it can solve. Burning the reserve on
mechanical work doesn't just waste money; it *forfeits capability*.

The fix is a standing protocol, written down where every agent reads it, so
the routing decision is never made ad hoc in the moment.

## Core Principle

**Expensive tokens buy thinking. Cheap tokens buy doing.**

- The Mythos-class plan is a reserve, spent only on Mythos-class problems,
  in the main thread, in sessions you open deliberately — and there it
  should spend *freely*: max thinking, long loops, real experimentation.
  Reserve does not mean timid.
- The workhorse plan is used liberally for implementation, research,
  exploration, and review — down to 0% if needed.
- Every routine token of mechanical work on the reserve is a token
  unavailable for the hard problem you haven't hit yet.

## Role Definitions

**Mythos-class (reserve main thread, user-initiated only):**

- Architecture decisions and design audits with long-lived consequences
- Security reviews and incident reasoning
- Hard debugging after the workhorse has failed twice on the same problem
- Analysis ahead of irreversible decisions
- Plan documents and strategy synthesis
- Review verdicts on Mythos-class changes (in-session — see Lane R3)

Make each such session end in a **durable artifact** — a plan document,
audit report, decision log, or review verdict committed to a repo. Inference
evaporates; documents survive the session and compound.

**Workhorse: everything else.** Implementation, refactors, tests, research
sweeps, codebase exploration, PR review, scheduled jobs.

## Hard Rules

1. **The reserve family is human-interactive only — every lane, no
   exceptions.** Smaller siblings of the Mythos-class model usually bill the
   same plan, so a "cheap" exploration subagent is not cheap. That covers all
   of: native subagent/Task spawns at any tier, headless dispatches from
   scripts or other agents, scheduled and CI lanes, and **review lanes**.
   Review is the one people leave behind, because "it is only a review" — and
   then the reserve is staffing a gate that runs on every PR. Inside a reserve
   session, delegation means shelling out to the workhorse CLI, full stop.
2. **Resolve routes from the config, never from memory.** Where a
   machine-readable routing config exists it is the source of truth for model
   identifiers, effort, and reviewer staffing. A remembered identifier goes
   stale silently and a retired one fails at dispatch time. An unknown or
   retired identifier must fail fast rather than fall through to a default.
3. **Effort ceiling: high.** No lane — orchestrator, executor, or reviewer —
   runs above the host's normal high tier unless the human states a need for
   that specific case. Top tiers overthink each step, spend the scarcest quota
   fastest, and rarely change the answer. This retires any older rule that
   said review always runs at the maximum tier.
4. **No multi-agent ceremony in the reserve session.** A Mythos session is
   single-threaded, focused, bounded: no recursive agent trees, no parallel
   reserve fan-outs, no "ultra" workflow patterns. One problem in, one
   artifact out.
5. **The workhorse never escalates to the reserve on its own.** Only the
   human decides to spend reserve tokens.
6. **Hard-switch off the quota bombs.** Harness features that loop, fan out,
   or recurse autonomously on the reserve plan — recurring-prompt loops,
   "ultra" multi-agent cloud reviews, autonomous workflow modes, and
   anything similar — can eat a week's usage in one prompt. Don't rely on
   discipline: disable them at the harness level where possible (permission
   deny rules, uninstalled plugins, settings), so a stray invocation fails
   instead of billing. If one of these genuinely earns its cost on a
   Mythos-class problem, the human enables it for that run and switches it
   back off.

## The Workhorse Ladder

A modern workhorse subscription is not one model. It is a ladder of tiers that
trade capability against burn, and the routing win comes from putting each unit
of work on the lightest rung that still clears it.

Three rungs are enough:

- **Orchestration rung** — the strongest tier. Orchestration, review lanes, and
  a *bounded* one-rung escalation for a unit that genuinely defeated the rung
  below. Never bulk volume: this rung is where a careless fan-out empties the
  window.
- **Standard rung** — the everyday executor. Well-specced implementation and
  research. Medium effort by default, high when the unit is demanding.
- **Mechanical rung** — templated and mechanical work: probes, queues,
  classification, boilerplate, small well-specced patches. Medium effort.

**The rung ladder is not a rate-limit ladder.** On at least one major provider
the short rolling rate window is enforced at the *account* level, shared by
every tier — verify this on your own stack with a cheap probe rather than
assuming. Where it is shared, no rung rescues another rung's exhausted window,
and "escalate a tier" is not a workaround. When the window is dead: queue the
work until it resets, or move genuinely urgent units to a *different family*
that respects your reserve floor. Never promote bulk volume to the orchestration
rung to get around a window.

Distinguish two failure shapes that look alike: an exhausted window (queue and
wait) and a model-identifier rollout error (one logged retry on a sibling rung,
then fail closed and surface it). Treating a rollout error as a window
exhaustion strands the pipeline; treating an exhaustion as a rollout error
burns the next rung too.

## Routing Table — Example Instantiation

The mapping below is the worked example, not the spec. The roles are the spec;
refresh the names via the self-update clause whenever the landscape moves.

| Work | Route |
|---|---|
| Narrow mechanical patch, probe, classification | Workhorse mechanical rung, medium effort |
| Standard well-specced feature | Workhorse standard rung, medium effort |
| Challenging well-specced patch | Workhorse standard rung, high effort |
| Research sweep / codebase exploration | Workhorse standard rung, medium–high effort |
| A unit that defeated the standard rung | Workhorse orchestration rung, high effort, bounded |
| Headless / scheduled orchestration | Workhorse orchestration rung, high effort |
| PR correctness review | Independent reviewer family, high effort (Lane R1) |
| PR security review | A further independent family, high effort (Lane R2) |
| Optional advisory review on high-risk PRs | A further family still (Lane R3) |
| Visual / UX walk of a running build | A route with real UI-driving capability, medium effort |
| Architecture, security reasoning, hard debugging, strategy | Mythos-class main thread (the reserve) |

Effort-tier names follow whatever your workhorse CLI calls its reasoning-effort
setting. Keep them in the routing config, not in prose.

## The Review Gate — Every PR Still Gets Reviewed

Reserving the expensive model must not mean dropping review. Three lanes:

- **Lane R1 — correctness (blocking, every PR):** a fresh-context antagonist
  at high effort with a findings-only prompt, from a family different from the
  author's where the stack allows it. The reviewer must be a **separate
  invocation** from the implementer — the session that wrote the code never
  reviews its own diff. Where no other family is available, cross-instance
  review within one family, given fresh context and an adversarial prompt,
  still catches most of what cross-family review catches. (See `tenth-man.md`
  in this repo for the prompt template.) Zero reserve cost.
- **Lane R2 — security (blocking, every PR):** a *separate* reviewer, from a
  further family, with a security-only scope: exploitability, trust boundaries,
  auth bypasses, injection paths, races with a security consequence, secret
  exposure, unsafe failure states. Not a bullet inside the R1 prompt — a
  mixed checklist loses the security items to the correctness items every
  time. This lane is why the number of families on a modern stack matters more
  than the number of subscriptions. Zero reserve cost.
- **Lane R3 — advisory (optional, high-risk PRs):** a further independent
  family on auth, payments, data-loss surfaces, caching and service-worker
  behavior, and anything production-adjacent. Non-blocking: its findings are
  triaged into R1 or R2 when real, and never block on their own authority.
  See `double-up-code-review.md`. Still zero reserve cost.
- **Lane R4 — reserve in-session verdict (Mythos sessions only):** when the
  reserve model is *already engaged* as the orchestrator on a hard problem,
  have it read the final diff and issue an in-session verdict **in addition
  to** R1 and R2. The marginal cost is small — the context is already loaded —
  and it catches spec-level and architectural misses that implementation-focused
  review tends not to flag. Never spawn a *separate* reserve process for this.
- **Independence rule:** whoever authored the spec has structural blind spots
  about it. The gates that count are the fresh-context independent passes (R1,
  R2); an orchestrator's own verdict is additive, never a substitute.
- **Fail-closed rule:** a required lane that fails procedurally — wrapper error,
  timeout, unparseable verdict — blocks the merge. It never escalates to the
  reserve as a fallback, and it never falls back to the author's family. Fix the
  route or take the block.
- **Data-residency rule:** exclude a route from sensitive material by where the
  data is hosted and what the provider's data policy allows, not by vendor
  reputation. Gate the surface, not the model: the same route may be a full peer
  on ordinary code and ineligible for one regulated surface. A jurisdiction skip
  is a recorded review limitation and is non-blocking; if it leaves a *required*
  axis unstaffed, that is a block for a human to resolve.

## Inline vs. Delegate — When the Orchestrator Codes Itself

Dispatching a task to the workhorse costs the orchestrator a fixed overhead:
writing the spec, auditing the result, possibly iterating. For tiny changes
that overhead exceeds the cost of just making the edit. The expensive
orchestrator implements **inline** only when *all* of these hold:

- ≤ ~25 changed lines across ≤ 2 files
- the change is fully determined by context already in its window — zero new
  exploration needed
- no test-iterate loop anticipated; verification is one cheap command
- no branch/worktree ceremony beyond what the session is already in

Typical inline cases: version-bump rituals, applying review nitpicks in an
already-open session, config/doc one-liners, the single obvious fix for a
bug the orchestrator just root-caused. **Everything else goes to the
workhorse**, on the lightest lane that suffices.

**Corollary — hand over the goal, not the spec.** If the orchestrator would
have to read substantial new code just to *write* a delegation spec, don't.
Give the workhorse the goal plus acceptance criteria and let it explore.
Workhorse exploration is near-free; frontier-model reading is not. The
expensive tokens go into the parts only the frontier model can do: framing
the problem, setting acceptance criteria, and judging the result.

## Phases — Adapting When Plans Change

Subscription terms move. Encode your situation as dated phases so every
agent reading the skill applies the right rules without re-litigating them:

- **Reserve window:** while the Mythos-class model is on your subscription
  but heavily metered — all rules above apply.
- **Sunset:** if the model leaves your subscription for token-based billing
  and you run a zero-API-cost policy, usage drops to **zero** — no sessions,
  no dispatches — until budget allows or it returns to a subscription. Any
  invocation during sunset is a budget breach: stop and surface it. Freed
  subscription capacity (e.g. the next tier down) can then resume staffing
  the review gate if your plan still includes it.
- **Abundance:** if you later hold quota to spare, relax rule 1 first
  (reserve-family exploration subagents), keep the review-independence rule
  and the inline-economics rule — those are good engineering regardless of
  budget.

Write the actual dates into your local copy and put a calendar reminder on
the phase boundary.

## The Self-Update Clause

The model names in this spec will go stale; the roles will not. When the
landscape shifts — your reserve model changes family, the workhorse gets
leapfrogged, a third subscription enters the picture — do not edit the skill
by hand. Open a session with your current frontier model and say:

> Update my `mythos-reserve-routing` skill to reflect the present situation:
> these are my current subscriptions, models, metering terms, and CLIs.
> Re-derive the routing table, the review-gate staffing, and the phase
> dates. Keep the roles and rules intact.

Re-deriving a routing table from the principles in this spec is itself a
Mythos-class task — framing, judgment, economics — and a textbook use of
reserve tokens: a few minutes of frontier thinking that compounds across
every dispatch that follows.

## Anti-Leak Checklist

Things that silently burn the reserve — audit your setup for each:

- Subagent/Task tools defaulting to a reserve-family tier inside reserve
  sessions
- Skills or scripts that internally call the reserve model (panels,
  reviewers, summarizers) as part of "automatic" pipelines
- Scheduled tasks, CI hooks, or cloud routines configured with reserve-family
  models
- Plan-mode or exploration helpers that spawn reserve-family subagents
- Habitual "let me just ask the big model" moments for questions the
  workhorse answers identically
- Loop / ultra-review / autonomous-workflow features left enabled on the
  reserve plan — the single biggest one-prompt quota killers; hard-disable
  them (rule 6)
- **Review lanes still staffed with the reserve family** — the most common
  leak once the obvious ones are fixed, because a per-PR gate looks small and
  runs on every PR
- Routing configs that name a model identifier the provider has since retired,
  so dispatch silently falls back to a default tier
- Visual/UX walkers, summarizers, and triage steps quietly defaulting to the
  reserve family inside otherwise-workhorse pipelines

## Installation Recipe

Install the same protocol where **both** agents read it:

1. **Reserve harness:** create the skill in that harness's skill directory,
   with whatever metadata it requires (a name, and a description that triggers
   on any routing, model-selection, review-staffing, or quota decision),
   followed by the rules above, edited to your dates and model versions.
2. **Workhorse CLI:** create the same content in that CLI's prompt or skill
   directory, written from the workhorse's perspective (its key rule: it never
   routes work to the reserve on its own).
3. Reference the skill from each agent's global instructions file in one line
   each, so the protocol is discoverable even when the skill is not
   auto-triggered.
4. Keep a machine-readable routing config (TOML/JSON mapping task types to
   provider, model identifier, effort, and reviewer role). Make it the source
   of truth, have every skill defer to it on conflict, and have the dispatch
   layer fail fast on an identifier the config does not know. This is what
   keeps the prose in this spec from becoming the routing authority as it ages.
5. Record which routes are ineligible for which data surfaces, with the reason
   (hosting jurisdiction, provider data policy, contractual limit). Ineligibility
   belongs in the config, not in an agent's judgment at dispatch time.

For other harnesses, place the same two artifacts wherever each agent loads
skills, prompts, or standing instructions.

## Smoke Test

1. In a fresh reserve session, ask: "explore this repo and summarize the
   architecture." Correct behavior: it dispatches the workhorse CLI (medium
   effort) rather than spawning a reserve-family subagent or grinding
   through files itself.
2. Ask it to fix a one-line typo it can already see. Correct behavior:
   inline edit, no dispatch.
3. Open a PR and ask for review. Correct behavior: two independent
   invocations — correctness and security — at high effort, from families
   other than the author's where available, both separate from whichever
   invocation implemented the change, and neither on the reserve plan.
4. Make the security route fail. Correct behavior: the gate blocks and says so.
   Incorrect behavior: it escalates to the reserve model or approves anyway.
5. Ask for a scheduled or headless job. Correct behavior: it is staffed from
   the workhorse ladder, never the reserve family, whatever the task.
6. During a declared sunset phase, ask for the Mythos-class model. Correct
   behavior: refusal with a pointer to the phase rule, not a silent
   invocation.

## Redaction Note

This spec deliberately describes routes by **role** — reserve, workhorse rungs,
correctness reviewer, security reviewer, advisory reviewer — rather than by
brand. Role names do not go stale and do not reveal whose stack this came from.
Everything private to the originating setup — hostnames, repo layouts, org
names, ticket systems, wrapper commands, internal dates and directives — has
been removed. When you install it, your local copy will accumulate private
context of its own: keep it out of any public fork.
