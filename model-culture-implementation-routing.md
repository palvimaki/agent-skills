# Model-Culture Implementation Routing Skill Specification

This document defines a reusable routing skill for assigning work to the model,
agent, or tool route whose working style best fits the current task phase. It is
intentionally provider-agnostic, model-agnostic, account-agnostic,
machine-agnostic, and implementation-agnostic.

The skill can be implemented by any capable LLM agent that can inspect the
host's available model routes, research current model behavior, and create or
update a local skill, prompt, routing guide, or agent instruction file.

> **Doctrine version: 2026-08-19.** Any model name in this repository is a
> worked example with a shelf life. Re-derive the local map from current
> documentation and observed behavior before you install.

## Contract

- Route by observed behavior in this environment, not by brand or benchmark
  rank.
- Research the current route set before writing the map. A remembered map is a
  stale map.
- Fill every role, not only the three creative ones: expansion, coherence,
  execution, correctness review, security review, visual walking, wide-context
  fallback, and research.
- Where a machine-readable routing config exists, it is the source of truth and
  the guide defers to it on conflict.
- Record which routes are ineligible for which data surfaces, and why. Decide by
  hosting jurisdiction and data policy, not by vendor reputation.
- Review roles must be independent of the author role. That constraint outranks
  temperament fit.
- Re-derive the map when any family ships a major version, or when a route's
  observed behavior stops matching its entry.

## Skill Identity

Name: `model-culture-implementation-routing`

Purpose: create a local routing guide that maps task phases to available model
or agent routes by temperament and observed fit, not by benchmark rank alone.

Use this skill when the user asks for:

- model routing by strengths, weaknesses, temperament, or "culture";
- a multi-model implementation workflow;
- guidance for choosing which model should plan, implement, review, explore, or
  execute;
- a model-agnostic version of a model-specific routing rule;
- an update to routing policy after models, plans, tools, or limits change.

Do not use this skill to force every task through multiple models. The goal is
phase-fit routing, not committee behavior.

## Core Principle

Route work by the current model's demonstrated behavior in the host
environment. Do not treat public benchmark rank, brand, price, or model size as
the routing policy by itself.

The installed local skill should answer:

- Which available route is best for opening the search space?
- Which available route is best for coherent direction-setting?
- Which available route is best for concrete execution?
- Which route should provide correctness dissent or review?
- Which route should provide the separate security review?
- Which route can actually drive a user interface and judge what it sees?
- Which route is the read-only fallback when a primary route fails or a very
  large context is needed?
- Which routes are ineligible for which data, and for what stated reason?
- When should the active agent override the default route?

## Non-Negotiable Constraints

- Do not depend on any specific LLM provider, model, account, API, product,
  machine, path, subscription, repository, or organization.
- Do not include private usernames, hostnames, internal project names,
  repository paths, account IDs, API keys, tokens, or local operational details
  in the public or installed skill.
- Do not hardcode stale public claims as if they are current facts.
- Before installing or updating the local skill, research the currently
  available model routes and their strengths, weaknesses, limits, and tool
  behavior.
- Prefer current primary sources when available: model documentation, release
  notes, model cards, provider migration guides, local routing configs,
  evaluation reports, and the host's own task history.
- Treat third-party benchmarks and anecdotes as supporting evidence, not final
  authority.
- Distinguish researched facts from local judgment.
- Keep the final local skill short enough to be used in normal agent context.
- A route's temperament never overrides an independence rule. A reviewer must
  not come from the author's family when an alternative exists, whatever the
  culture map says about fit.
- Cap reasoning effort at the host's normal high tier unless the user states a
  need for a higher tier in that specific case.
- The active agent owns the final routing decision and may override the guide
  based on task context, tool availability, limits, latency, cost, and observed
  performance.

## Timeout and Long-Running Work

Route discovery, model research, and evaluation runs can take longer than
ordinary shell commands, especially when local hardware is slow, model CLIs
queue, or benchmark prompts use high-reasoning settings. Implementations must
expose configurable timeouts, choose generous defaults for research and
evaluation routes, preserve partial findings when possible, and avoid treating
slow but active model work as failed solely because a short generic timeout
elapsed.

## Required Inputs

The skill should work with these inputs, using safe defaults:

- `available_routes`: model or agent routes available in the host environment.
  Default: discover from local config, installed CLIs, tool metadata, or user
  instructions.
- `task_domains`: common work the host expects to route. Default: planning,
  architecture, implementation, debugging, review, research, product strategy,
  naming, UX, operations, and deployment.
- `research_sources`: sources to consult for current model behavior. Default:
  official docs and release notes first, then local task history and reputable
  public evaluations.
- `constraints`: host-specific limits such as budget, rate limits, data
  policies, privacy boundaries, offline mode, latency, or available tools.
- `output_target`: where to install the local routing skill. Default:
  harness-appropriate skill directory, prompt library, routing config, or a
  project instruction file chosen by the implementing agent.

## Required Outputs

Always produce or update one local routing artifact containing:

- a short doctrine for routing by model temperament;
- the local model or route categories, with current names filled in;
- each route's best use cases;
- each route's known failure modes or avoid conditions;
- task-phase routing rules;
- a default workflow pattern;
- explicit override rules;
- a date or note showing when the routing research was last refreshed;
- a statement that the guide is heuristic, not a rigid policy.

If the host supports skill metadata, include a clear trigger description so the
skill activates when choosing routes or model responsibilities.

## Workflow

1. Discover the available model, agent, and tool routes in the local host.
2. Research current strengths, weaknesses, tool behavior, latency/cost limits,
   and failure modes for those routes.
3. Review local evidence if available: past task outcomes, review failures,
   retry patterns, user preferences, and known tool constraints.
4. Group routes by useful working style. Use neutral labels such as
   `expansion`, `coherence`, `execution`, `review`, `specialist`, or equivalent
   names that fit the local environment.
5. Map common task phases to the route group that creates the most leverage.
6. Add override rules for urgency, tool access, safety, privacy, budget,
   latency, degraded availability, and user instruction.
7. Write the local skill or routing artifact.
8. Redact local identifiers and secrets from any public copy.
9. Run the smoke test below.

## Research Guidance

Research should be current at installation time. The installer should prefer:

- official model and product documentation;
- release notes and migration guides;
- model cards, system cards, and safety notes;
- current CLI/tool capability docs;
- local routing configs and previous task logs;
- recent public evaluations when they match the host's task domain.

Avoid overfitting to generic leaderboards. A model that wins a benchmark may
still be the wrong route for a short patch, a messy product decision, a
privacy-constrained task, or a tool-heavy deployment.

When research is not possible, make a conservative local guide using only known
available routes and mark it as `needs refresh`.

## Local Skill Template

The implementing agent should fill in the bracketed sections and remove
irrelevant examples:

```markdown
---
name: model-culture-implementation-routing
description: Use when choosing which available model, agent, or tool route should handle planning, exploration, implementation, review, or strategy phases. Research current local model strengths before changing the routing guide.
---

# Model-Culture Implementation Routing

Last refreshed: <YYYY-MM-DD>

## Purpose

Route tasks by demonstrated model temperament and task-phase fit, not by
benchmark rank alone.

This is a guideline, not a rigid rule. The active agent owns the final routing
decision based on task context, available tools, limits, and observed model
performance.

## Local Model Cultures

### <Expansion Route> = Expansion / Lateral Search

Best when the task needs broad context, alternatives, missed angles, naming,
strategic reframing, edge-case discovery, or unconventional solution paths.

Avoid when the decision must be finalized now or when the route tends to produce
too many options without closure.

### <Coherence Route> = Coherence / Design

Best when the task has architectural consequences, ambiguous requirements, API
boundaries, UX logic, long-term maintainability, product taste, or risk of
building the wrong abstraction.

Avoid for tiny patches where design discussion would slow the work.

### <Execution Route> = Momentum / Execution

Best when the task needs concrete code, fast iteration, repo edits, patches,
tests, wiring, migration, cleanup, or "just make it work."

Avoid for open-ended strategy where fast implementation could lock in the wrong
direction.

### <Review Route> = Dissent / Verification

Best when the task needs antagonistic review, merge gating, correctness
analysis, or independent critique before committing.

Avoid as final authority unless the host explicitly makes this route the
approval gate.

### <Security Review Route> = Adversarial / Exploitability

Best when the question is what an untrusted actor can reach: trust boundaries,
auth bypasses, injection paths, races with a security consequence, secret
exposure, and unsafe failure states.

Keep it separate from `<Review Route>`. A single reviewer holding both scopes
spends its attention on correctness, because correctness is easier to prove.
This is a second review, with its own prompt and its own verdict — not one more
bullet in the correctness prompt.

Avoid for style, performance, or coverage questions. Those belong to the other
axis.

### <Visual Walker Route> = Drive and Judge a Running Surface

Best when the acceptance criterion is what a user sees: walking flows in a
running build, capturing states, and judging them against a spec.

Requires real capability, not just reasoning: the route must be able to drive a
browser or a simulator and read back what it produced. Verify the capability
before assigning the role. Moderate reasoning effort is usually enough.

### <Wide-Context Fallback Route> = Read-Only Breadth

Best for an explicit whole-codebase sweep, and for staffing the **breadth or
whole-tree** role at selection time when the preferred route is ineligible for
that material.

**Never the security axis.** Not at selection time, not as a substitute. Breadth
is the opposite of the narrow, adversarial scope that axis needs, and assigning
it there rebuilds the mixed checklist the two-axis split exists to remove. If no
eligible security route exists, block and wait for a human — as
`tenth-man.md` and `double-up-code-review.md` require.

**Not a post-failure substitute.** When a blocking review axis fails to run, or
returns an unparseable verdict, the gate fails closed — see `tenth-man.md` and
`double-up-code-review.md`. Swapping in this route instead would answer a
security question with a general read of the tree and return a `GO`, which
rebuilds the mixed checklist the two-axis split exists to remove. Substituting a
model is not a remedy for a failed gate; fixing the route or taking the block is.

Constrain it: read-only, explicit request or a documented selection-time
assignment only, and excluded from any data surface its hosting jurisdiction or
data policy makes ineligible.

## Task Routing

| Task phase | Preferred local route |
|------------|-----------------------|
| Review code for correctness | <Review Route>, independent of the author |
| Review code for exploitability | <Security Review Route>, independent again |
| Check a running user-facing build | <Visual Walker Route> |
| Sweep a whole codebase read-only | <Wide-Context Fallback Route> |
| Build feature | <Execution Route> |
| Design feature | <Coherence Route> |
| Find better feature shape | <Expansion Route>, then <Coherence Route> |
| Refactor architecture | <Coherence Route>, then <Execution Route> |
| Patch bug | <Execution Route> |
| Explain bug cause | <Coherence Route> |
| Explore unknown API/tool/library | <Expansion or Research Route>, then <Execution Route> |
| Product strategy | <Coherence Route> + <Expansion Route> |
| Tactical execution plan | <Execution Route> after <Coherence Route> framing |
| "Could there be a smarter way?" | <Expansion Route> |

## Operating Rule

Do not ask every model everything. Pick the route whose default behavior helps
the current phase.

Golden pattern: expansion opens possibility; coherence chooses direction;
execution makes it real.

## Overrides

Override this guide when:

- the user explicitly names a route;
- a route is unavailable, rate-limited, too slow, or lacks required tools;
- hosting jurisdiction, data residency, or provider data policy makes the
  preferred route ineligible for this specific material — gate the surface, not
  the model, and record the skip as a limitation;
- the task is urgent and a capable execution route can complete it now;
- local experience shows a different route is currently performing better;
- a review or safety gate requires independence from the implementation route.
```

## Example Only: One Possible Instantiation

The following example illustrates the principle. It is not a universal routing
policy and must be modified according to current local findings, available
routes, tools, constraints, and observed model performance.

```markdown
Model cultures:

- Codex = Momentum / Execution
  Best when the task needs concrete code, fast iteration, repo edits, patches,
  tests, wiring, migration, cleanup, or "just make it work."
  Use when forward motion matters more than perfect architecture.

- Claude = Coherence / Design
  Best when the task has architectural consequences, ambiguous requirements,
  API boundaries, UX logic, long-term maintainability, product taste, or risk
  of building the wrong abstraction.
  Use when wrong direction is more expensive than slow progress.

- Gemini = Lateral / Expansion
  Best when the task needs broad context, weird alternatives, missed angles,
  naming, strategic reframing, edge-case discovery, or unconventional solution
  paths.
  Use when the obvious solution may not be the best solution.

Golden pattern:
Gemini opens possibility. Claude chooses coherent direction. Codex makes it
real.
```

If the host does not use these models, replace them. If current research shows
their behavior has changed, update the labels. If local tools make a different
route better for execution, review, or exploration, use the local evidence.

The example above is also deliberately incomplete: it shows three creative
cultures and no review, security, visual, or fallback roles. A real installed
map fills all of them. Note too that a modern stack usually has more families
available than roles that need distinct families — that surplus is exactly what
makes an independent security axis affordable.

### Re-derivation cadence

Re-derive the map when any of these happens, not on a calendar:

- a family in the map ships a major version, or renames or retires a tier;
- a route's observed behavior stops matching its entry — this is the signal that
  matters most, and only local task history shows it;
- a new family becomes available, or a subscription ends;
- a data-policy or hosting change moves a route's eligibility.

## Redaction

Before publishing or sharing any installed version, redact:

- local usernames, hostnames, and home-directory paths;
- private organization names and project codenames;
- account IDs, emails, phone numbers, and messaging identifiers;
- API keys, tokens, cookies, passwords, session IDs, OAuth material, SSH keys,
  private key blocks, and credential URLs;
- internal routing commands that reveal private infrastructure.

Use placeholders such as `<Execution Route>`, `<Coherence Route>`, `<repo>`, and
`<local skill directory>` in public documentation.

## Smoke Test

After installation, test the local skill with these prompts:

```text
Choose a model route for a one-line bug patch with tests.
```

Expected: chooses the execution route and avoids multi-model ceremony.

```text
Choose a model route for a new authentication architecture with unclear product requirements.
```

Expected: starts with the coherence/design route, then hands implementation to
the execution route after the plan is clear.

```text
Could there be a smarter shape for this feature before we build it?
```

Expected: uses the expansion/lateral route first, then a coherence route to
collapse options.

```text
The preferred execution route is unavailable. What now?
```

Expected: applies override rules and chooses the next capable local route while
stating the limitation.

## Installation Recipe

1. Copy this specification into the target environment.
2. Have an agent inspect the local model routes and research current behavior.
3. Have the agent fill in the local skill template.
4. Install the filled skill where the host stores reusable skills, prompts,
   slash commands, or routing instructions.
5. Run the smoke test.
6. Record the refresh date and sources used for routing judgments.

The installed artifact should be short and local. This public specification is
the portable recipe, not the final route map for every environment.
