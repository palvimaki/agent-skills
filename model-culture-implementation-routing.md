# Model-Culture Implementation Routing Skill Specification

This document defines a reusable routing skill for assigning work to the model,
agent, or tool route whose working style best fits the current task phase. It is
intentionally provider-agnostic, model-agnostic, account-agnostic,
machine-agnostic, and implementation-agnostic.

The skill can be implemented by any capable LLM agent that can inspect the
host's available model routes, research current model behavior, and create or
update a local skill, prompt, routing guide, or agent instruction file.

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
- Which route should provide dissent or review?
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
- The active agent owns the final routing decision and may override the guide
  based on task context, tool availability, limits, latency, cost, and observed
  performance.

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

Best when the task needs antagonistic review, merge gating, safety analysis,
security review, or independent critique before committing.

Avoid as final authority unless the host explicitly makes this route the
approval gate.

## Task Routing

| Task phase | Preferred local route |
|------------|-----------------------|
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
- privacy or data policy prevents using the preferred route;
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
