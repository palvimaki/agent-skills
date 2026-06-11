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
aggressively metered. Today, for many builders, that is the frontier Claude
model inside Claude Code, with Codex GPT-5.5 as the workhorse — and that
concrete stack is used as the worked example throughout. If the situation
reverses, or another family pushes to the front, you don't rewrite anything
by hand: **tell your frontier model to update this skill to reflect the
present situation.** It will take care of it. That self-update clause is
part of the spec.

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
  harness (the worked example: Claude Code as the main session, Codex CLI
  via `codex exec` as the delegated executor).

## The Problem This Solves

Frontier reasoning models are at their best when you let them think long,
loop, and experiment — which is exactly what makes them able to consume a
week's subscription quota in an afternoon. Meanwhile the workhorse tier
(in the example: GPT-5.5 on a Codex plan) is a seriously capable
implementation and review model whose low/medium-effort lanes are hard to
exhaust at all — a fact a surprising number of builders overlook.

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

1. **Zero reserve-family subagents, any tier.** Smaller siblings of the
   Mythos-class model usually bill the same plan — in the example, Opus,
   Sonnet, and Haiku spawns all draw from the one Claude subscription, so a
   "cheap" exploration subagent is not cheap. Inside a reserve session,
   delegation means shelling out to the workhorse CLI (`codex exec` in the
   example), full stop. Do not use the harness's native subagent/Task tool
   with reserve-family models.
2. **No headless reserve dispatches** (e.g. `claude -p`) from scripts or
   other agents — they burn the reserve invisibly.
3. **No scheduled or background reserve jobs.** Timed/cron/CI lanes run the
   workhorse.
4. **No multi-agent ceremony in the reserve session.** A Mythos session is
   single-threaded, focused, bounded: no recursive agent trees, no parallel
   reserve fan-outs, no "ultra" workflow patterns. One problem in, one
   artifact out.
5. **The workhorse never escalates to the reserve on its own.** Only the
   human decides to spend reserve tokens.

## Routing Table — Example Instantiation (mid-2026)

The mapping below is the worked example, not the spec. The roles are the
spec; refresh the names via the self-update clause whenever the landscape
moves.

| Work | Route |
|---|---|
| Narrow mechanical patch | Codex GPT-5.3 Spark, low effort |
| Standard well-specced feature | Codex GPT-5.3 Spark, medium effort |
| Challenging well-specced patch | Codex GPT-5.5, low effort |
| Research sweep / codebase exploration | Codex GPT-5.5, medium–high effort |
| PR review / dissent gate | Codex GPT-5.5, xhigh effort (see below) |
| Optional second antagonist on high-risk PRs | A third model family (e.g. Gemini CLI) |
| Architecture, security, hard debugging, strategy | Mythos-class main thread (the reserve) |

Effort-tier names follow the Codex CLI's `model_reasoning_effort` settings.

## The Review Gate — Every PR Still Gets Reviewed

Reserving the expensive model must not mean dropping review. Three lanes:

- **Lane R1 (default, all PRs):** a fresh-context workhorse reviewer at its
  highest reasoning effort, with an antagonist, findings-only prompt. The
  reviewer must be a **separate invocation** from the implementer — the
  session that wrote the code never reviews its own diff. Cross-instance
  review within one family, given a fresh context and an adversarial prompt,
  catches the large majority of what cross-family review catches, at zero
  reserve cost. (See the `tenth-man.md` spec in this repo for a full
  antagonist prompt template.)
- **Lane R2 (high-risk PRs, optional):** add a genuinely different model
  family as a second independent antagonist. Use for auth, payments,
  data-loss surfaces, caching/service-worker behavior, and anything
  production-adjacent. Still zero reserve cost.
- **Lane R3 (Mythos sessions):** when the reserve model is *already engaged*
  as the orchestrator on a hard problem, have it read the final diff and
  issue an in-session verdict **in addition to** Lane R1. The marginal cost
  is small (context already loaded) and it catches spec-level and
  architectural misses that implementation-focused review tends not to flag.
  Never spawn a *separate* reserve process for this.
- **Independence rule:** whoever authored the spec has structural blind
  spots about it. The gate that counts is always the fresh-context
  independent pass (R1); an orchestrator's own verdict is additive, never a
  substitute.

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

## Installation Recipe (worked example: Claude Code + Codex CLI)

Install the same protocol where **both** agents read it:

1. **Claude Code:** create `~/.claude/skills/mythos-reserve-routing/SKILL.md`
   with YAML frontmatter (`name`, and a `description` that triggers on any
   routing, model-selection, review-staffing, or quota decision) followed by
   the rules above, edited to your dates and model versions.
2. **Codex CLI:** create `~/.codex/prompts/mythos-reserve-routing.md` with
   the same content from the workhorse's perspective (its key rule: it never
   routes work to the reserve on its own).
3. Reference the skill from your global instructions file
   (`~/.claude/CLAUDE.md` / `~/.codex/AGENTS.md`) in one line each, so the
   protocol is discoverable even when the skill isn't auto-triggered.
4. If you keep a machine-readable routing config (TOML/JSON mapping task
   types to provider/model/effort), make it the source of truth and have the
   skill defer to it on conflict.

For other harnesses, place the same two artifacts wherever each agent loads
skills, prompts, or standing instructions.

## Smoke Test

1. In a fresh reserve session, ask: "explore this repo and summarize the
   architecture." Correct behavior: it dispatches the workhorse CLI (medium
   effort) rather than spawning a reserve-family subagent or grinding
   through files itself.
2. Ask it to fix a one-line typo it can already see. Correct behavior:
   inline edit, no dispatch.
3. Open a PR and ask for review. Correct behavior: a fresh workhorse
   invocation at top effort, separate from whichever invocation implemented
   the change.
4. During a declared sunset phase, ask for the Mythos-class model. Correct
   behavior: refusal with a pointer to the phase rule, not a silent
   invocation.

## Redaction Note

This spec names current public models and products (Claude Code, Claude
Fable/Opus/Sonnet/Haiku, Codex, GPT-5.5, GPT-5.3 Spark, Gemini CLI) as a
worked example only; the roles and rules are the portable part. Everything
private to the originating setup — hostnames, repo layouts, org names,
ticket systems, internal dates and directives — has been removed. When you
install it, your local copy will accumulate private context of its own:
keep it out of any public fork.
