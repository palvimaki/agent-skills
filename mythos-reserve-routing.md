# Mythos-Reserve Routing Skill Specification

A budget-aware agent routing protocol for builders running **Claude Code and
Codex CLI side by side on consumer subscriptions** — specifically for anyone
who has watched a single ambitious prompt eat a meaningful slice of their
weekly Claude limit.

Unlike the other specs in this repo, this one **names models on purpose**. It
exists for one concrete situation: you hold one Claude subscription (Pro/Max)
and one ChatGPT/Codex subscription, the Claude plan meters frontier-model
usage aggressively, and the Codex plan is generous at low/medium reasoning
effort. If that is you, brand-agnostic advice is less useful than a working
default you can install today and edit as the model landscape moves.

## Skill Identity

- Skill name: `mythos-reserve-routing`
- Purpose: treat your most expensive frontier model (Claude Fable 5 / Opus —
  "Mythos-class") as a **strategic reserve**, route all volume work to Codex
  GPT-5.5, and still keep a mandatory independent review gate on every PR.
- Trigger conditions: consult before any subagent dispatch, any model/effort
  selection, any PR-review staffing decision, and any "should the orchestrator
  just do this itself?" call.
- Audience: solo builders and small teams orchestrating Claude Code as the
  main session with Codex CLI (`codex exec`) as the delegated executor.

## The Problem This Solves

The frontier Claude models are at their best when you let them think long,
loop, and experiment in peace — which is exactly what makes them capable of
consuming a week's subscription quota in an afternoon. Meanwhile GPT-5.5 on a
Codex plan is a highly capable implementation and review model whose
low/medium-effort tiers are hard to exhaust at all.

The common failure mode is routing by habit instead of by economics: the
expensive model writes boilerplate, explores codebases, runs test loops, and
mechanically edits twenty files — work a cheaper model does just as well —
and then the quota is gone when a genuinely hard problem shows up.

The fix is a standing protocol, written down where both agents read it, so
the routing decision is never made ad hoc in the moment.

## Core Principle

**Expensive tokens buy thinking. Cheap tokens buy doing.**

- The Claude plan is a reserve, spent only on Mythos-class problems, in the
  main thread, in sessions you open deliberately.
- The Codex plan is the workhorse, used freely for implementation, research,
  exploration, and review — down to 0% if needed.
- Every routine token of mechanical work on the expensive plan is a token
  unavailable for the hard problem you haven't hit yet.

## Tier Definitions

**Mythos-class (Claude main thread, user-initiated only):**

- Architecture decisions and design audits with long-lived consequences
- Security reviews and incident reasoning
- Hard debugging after Codex has failed twice on the same problem
- Analysis ahead of irreversible decisions
- Plan documents and strategy synthesis
- Review verdicts on Mythos-class changes (in-session — see Lane R3)

Make each such session end in a **durable artifact** — a plan document, audit
report, decision log, or review verdict committed to a repo. Inference
evaporates; documents survive the session and compound.

**Everything else: Codex.** Implementation, refactors, tests, research
sweeps, codebase exploration, PR review, scheduled jobs.

## Hard Rules

1. **Zero Claude subagents, any tier.** Opus, Sonnet, and Haiku spawns all
   bill the same Claude plan — a "cheap" Sonnet exploration subagent is not
   cheap on a metered subscription. Inside a Claude Code session, delegation
   means shelling out to `codex exec`, full stop. Do not use the native
   subagent/Task tool with Claude-tier models.
2. **No headless Claude dispatches** (`claude -p`) from scripts or other
   agents — they burn the reserve invisibly.
3. **No scheduled or background Claude jobs.** Timed/cron/CI lanes run Codex.
4. **No multi-agent ceremony in Claude Code.** A Mythos session is
   single-threaded, focused, bounded: no recursive agent trees, no parallel
   Claude fan-outs, no "ultra" workflow patterns. One problem in, one
   artifact out.
5. **Codex never escalates to Claude on its own.** Only the human decides to
   spend reserve tokens.

## Routing Table

| Work | Route |
|---|---|
| Narrow mechanical patch | Codex GPT-5.3 Spark, low effort |
| Standard well-specced feature | Codex GPT-5.3 Spark, medium effort |
| Challenging well-specced patch | Codex GPT-5.5, low effort |
| Research sweep / codebase exploration | Codex GPT-5.5, medium–high effort |
| PR review / dissent gate | Codex GPT-5.5, xhigh effort (see below) |
| Optional second antagonist on high-risk PRs | Gemini CLI |
| Architecture, security, hard debugging, strategy | Claude main thread (the reserve) |

Effort-tier names follow the Codex CLI's `model_reasoning_effort` settings.
Adjust model IDs as new versions ship; the shape of the table is the point.

## The Review Gate — Every PR Still Gets Reviewed

Rationing the expensive model must not mean dropping review. Three lanes:

- **Lane R1 (default, all PRs):** a fresh-context Codex GPT-5.5 xhigh
  reviewer with an antagonist, findings-only prompt. The reviewer must be a
  **separate `codex exec` invocation** from the implementer — the session
  that wrote the code never reviews its own diff. Cross-instance review
  within one family, given a fresh context and an adversarial prompt, catches
  the large majority of what cross-family review catches, at zero Claude
  cost. (See the `tenth-man.md` spec in this repo for a full antagonist
  prompt template.)
- **Lane R2 (high-risk PRs, optional):** add a genuinely different model
  family — e.g. Gemini CLI — as a second independent antagonist. Use for
  auth, payments, data-loss surfaces, caching/service-worker behavior, and
  anything production-adjacent. Still zero Claude cost.
- **Lane R3 (Mythos sessions):** when Claude is *already engaged* as the
  orchestrator on a hard problem, have it read the final diff and issue an
  in-session verdict **in addition to** Lane R1. The marginal cost is small
  (context already loaded) and it catches spec-level and architectural misses
  that implementation-focused review tends not to flag. Never spawn a
  *separate* Claude process for this.
- **Independence rule:** whoever authored the spec has structural blind spots
  about it. The gate that counts is always the fresh-context independent pass
  (R1); an orchestrator's own verdict is additive, never a substitute.

## Inline vs. Delegate — When the Orchestrator Codes Itself

Dispatching a task to Codex costs the orchestrator a fixed overhead: writing
the spec, auditing the result, possibly iterating. For tiny changes that
overhead exceeds the cost of just making the edit. The expensive orchestrator
implements **inline** only when *all* of these hold:

- ≤ ~25 changed lines across ≤ 2 files
- the change is fully determined by context already in its window — zero new
  exploration needed
- no test-iterate loop anticipated; verification is one cheap command
- no branch/worktree ceremony beyond what the session is already in

Typical inline cases: version-bump rituals, applying review nitpicks in an
already-open session, config/doc one-liners, the single obvious fix for a bug
the orchestrator just root-caused. **Everything else goes to Codex**, on the
lightest lane that suffices.

**Corollary — hand over the goal, not the spec.** If the orchestrator would
have to read substantial new code just to *write* a delegation spec, don't.
Give Codex the goal plus acceptance criteria and let it explore. Codex
exploration is near-free; frontier-model reading is not. The expensive tokens
go into the parts only the frontier model can do: framing the problem,
setting acceptance criteria, and judging the result.

## Phases — Adapting When Plans Change

Subscription terms move. Encode your situation as dated phases so every agent
reading the skill applies the right rules without re-litigating them:

- **Reserve window:** while the Mythos-class model is on your subscription
  but heavily metered — all rules above apply.
- **Sunset:** if the model leaves your subscription for token-based billing
  and you run a zero-API-cost policy, usage drops to **zero** — no sessions,
  no dispatches — until budget allows or it returns to a subscription. Any
  invocation during sunset is a budget breach: stop and surface it. Freed
  subscription capacity (e.g. Opus) can then resume staffing the review gate
  if your plan still includes it.
- **Abundance:** if you later hold quota to spare, relax rule 1 first
  (Claude-tier exploration subagents), keep the review-independence rule and
  the inline-economics rule — those are good engineering regardless of
  budget.

Write the actual dates into your local copy and put a calendar reminder on
the phase boundary.

## Anti-Leak Checklist

Things that silently burn the reserve — audit your setup for each:

- Subagent/Task tools defaulting to a Claude tier inside Claude Code sessions
- Skills or scripts that internally call Claude (panels, reviewers,
  summarizers) as part of "automatic" pipelines
- Scheduled tasks, CI hooks, or cloud routines configured with Claude models
- Plan-mode or exploration helpers that spawn Claude subagents
- Habitual "let me just ask the big model" moments for questions GPT-5.5
  answers identically

## Installation Recipe

Install the same protocol where **both** agents read it:

1. **Claude Code:** create `~/.claude/skills/mythos-reserve-routing/SKILL.md`
   with YAML frontmatter (`name`, and a `description` that triggers on any
   routing, model-selection, review-staffing, or quota decision) followed by
   the rules above, edited to your dates and model versions.
2. **Codex CLI:** create `~/.codex/prompts/mythos-reserve-routing.md` with
   the same content from the Codex perspective (its key rule: it never routes
   work to Claude on its own).
3. Reference the skill from your global instructions file
   (`~/.claude/CLAUDE.md` / `~/.codex/AGENTS.md`) in one line each, so the
   protocol is discoverable even when the skill isn't auto-triggered.
4. If you keep a machine-readable routing config (TOML/JSON mapping task
   types to provider/model/effort), make it the source of truth and have the
   skill defer to it on conflict.

## Smoke Test

1. In a fresh Claude Code session, ask: "explore this repo and summarize the
   architecture." Correct behavior: it dispatches `codex exec` (medium
   effort) rather than spawning a Claude-tier subagent or grinding through
   files itself.
2. Ask it to fix a one-line typo it can already see. Correct behavior: inline
   edit, no dispatch.
3. Open a PR and ask for review. Correct behavior: a fresh `codex exec` xhigh
   reviewer, separate from whichever invocation implemented the change.
4. During a declared sunset phase, ask for the Mythos-class model. Correct
   behavior: refusal with a pointer to the phase rule, not a silent
   invocation.

## Redaction Note

This spec deliberately keeps public model and product names (Claude Code,
Claude Fable/Opus/Sonnet/Haiku, Codex, GPT-5.5, GPT-5.3 Spark, Gemini CLI)
because the skill targets builders in exactly this stack. Everything private
to the originating setup — hostnames, repo layouts, org names, ticket
systems, internal dates and directives — has been removed. When you install
it, your local copy will accumulate private context of its own: keep it out
of any public fork.
