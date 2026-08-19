# Ultra Prompt Skill Specification

This document defines a reusable skill: **given a task, write a high-quality
prompt for Claude Code's Ultra-Code orchestration feature — its `ultra*code`
effort tier and the dynamic, many-subagent runs it spawns — and pair it with a
tailored recommendation for the configuration that runs it most
token-efficiently.**

The *subject* is deliberately product-specific — it is about one named feature,
Claude Code's Ultra-Code — and that specificity is the point. The *runner* is
agent-agnostic: any capable LLM agent that can write text can follow this spec to
produce the prompt and the configuration advice. The agent doing the writing does
not have to be the model that later runs the prompt.

No LLM API, SDK, or provider key is required. The skill produces text — a prompt
plus advice — that a human pastes into their own session.

> **Doctrine version: 2026-08-19.** The orchestration mechanics below were
> written against one product at one point in its life, and that product moves
> fast. Treat every version number, cap, and keyword as a claim to re-verify
> against current official documentation before you rely on it.

## Contract

- Write the prompt. Do not run the task. The user runs it.
- Deliver the prompt first, as one copy-pasteable block, then the
  token-efficiency configuration. Both halves, every time.
- Every prompt states: one objective, an exact acceptance condition, a fenced
  scope, a named orchestration shape, a plan-before-fan-out requirement, the
  evidence each result must carry, and a stop condition.
- Every prompt carries a short routing block: which route runs which stage, at
  what effort, and when to stop. See "Routing block".
- An orchestration run is armed on the **workhorse** plan, never on a reserve
  plan the operator keeps for interactive thinking work.
- Recommend a plain single-agent prompt whenever the task does not need many
  coordinated agents. Most tasks do not.
- Keep the keyword-safety convention below, and strip the break only in the
  final prompt handed to the user.

## Scope note — one product's mechanics, any runner

The orchestration mechanics documented here belong to one named product feature.
That specificity is deliberate: the job of this skill is to write good prompts
for *that* kind of run. Two things keep it portable:

- **The runner is agent-agnostic.** Any capable agent that can write text can
  follow this spec.
- **The method transfers.** Objective, fenced scope, named shape, plan-first,
  evidence rules, bounded stop condition, and an explicit routing block are what
  make *any* multi-agent or long-horizon run behave. If your host calls its
  feature something else, keep the method and replace the mechanics section
  after checking that host's current documentation.

## Keyword-safety convention (READ FIRST)

This spec is meant to be **read** by a coding agent — Claude Code or any other —
the way it reads any skill or doc pulled from a repo or URL. Read that way (as
file content or tool output) it triggers nothing. It is **not** meant to be
pasted into a live prompt box — but it is written to stay inert even if it is.

Claude Code can arm a background run from text in the **user's prompt**. Exactly
two classes of text matter; everything else is ordinary prose and is left alone.

1. **Hot keywords — fire anywhere in a prompt.** There are only two: the
   orchestration keyword (the Ultra-Code keyword) and the legacy opt-in word that
   meant the same thing before it was renamed. This spec writes each one **only**
   inside an inline-code span with an asterisk inside it — `` `ultra*code` `` and
   `` `work*flow` `` — so the literal string never matches (the asterisk breaks
   it) while the asterisk still renders literally (a code span suppresses Markdown
   emphasis, so two of them can't pair up and swallow the break). In running prose
   the feature is named **Ultra-Code** (hyphenated — a different string, not the
   trigger) and a run is called an **orchestration run**, so the bare keyword
   never appears outside those code spans.
2. **Slash-commands — fire only as the first token of an input line.** Written in
   inline code and never placed at the start of a line, they are inert. (`/g*oal`,
   a cross-tool example, is shown broken for the same reason.)

**Not neutralized, on purpose:** ordinary words that merely share a name with a
command — *model, plan, review, effort, clear, config, status, agents* — are left
exactly as written. They are not triggers, and mangling them would wreck the
prose for zero safety gain. The rule is "neutralize the two hot keywords and keep
slash-commands off line-starts," not "escape every English word."

There is no official escape syntax for this. The nearest *standard* is the Unicode
zero-width space (U+200B), the same trick used to defang `@`-mentions and
`#`-hashtags — but it is avoided here on purpose: it is invisible and a known
source of copy-paste "invisible-character" bugs. A visible asterisk inside a code
span is self-documenting and survives copy-paste.

**One rule when you emit the real prompt for the user:** strip the asterisk from
`ultra*code` so the keyword actually fires in their session. The break belongs
everywhere in this spec; the working keyword belongs only in the finished prompt
the human will paste.

## Skill Identity

Name: `ultra-prompt`.

Triggers when the user asks to generate, write, draft, or improve a prompt for
Ultra-Code (also typed "ultra code" / "ultra-code"), for a dynamic orchestration
run, for a `deep-research` run, or for any multi-subagent fan-out run in Claude
Code.

## What Ultra-Code is (ground truth, with a shelf life)

These mechanics were written against builds in the v2.1.1xx range: the trigger
keyword was renamed from the legacy word to `ultra*code` at v2.1.160. Shipped
builds have moved well past that, and this feature is young and moving fast.
**Re-verify every specific below — the caps, the tier names, the two start
paths, and the keyword itself — against the live build and current official
docs before relying on any edge detail.** Where a detail here disagrees with the
product's own documentation, the documentation wins and this file is stale.

- A **dynamic orchestration run** is a JavaScript script that Claude *writes* to
  coordinate subagents at scale, then executes in the background while the session
  stays responsive. The script holds the loop, the branching, and the
  intermediate results, so only the **final synthesized answer** returns to the
  main context window. Runs are resumable within the same session. Hard limits: up
  to **16 concurrent agents** and **1,000 agents total per run**.
- **Ultra-Code** is a session setting that combines the highest reasoning effort
  tier with **automatic** orchestration: with it on, Claude decides per
  substantive task whether to spawn a run, and one request can become several runs
  in a row (understand → change → verify). It is session-scoped, resets on a new
  session, and is the **most expensive** mode.
- **Two ways to start a run** — choosing the cheaper one is the single biggest
  lever:
  - **Keyword, one task (preferred):** prefix the prompt with the `ultra*code`
    keyword (asterisk stripped at emit time). A plain-language request to run it
    as an orchestration works too. This runs **one task** as an orchestration **without**
    raising the whole session to the top effort tier. Cheaper.
  - **Session mode:** setting the effort tier to `ultra*code` makes *every*
    substantive task in the session a run at top effort. Use only for a sustained
    stretch of genuinely hard tasks.
- Cost reality: a single run can spend **more of a weekly rate limit than a full
  day of normal use**. Reasoning "effort" is a *behavioral signal*, not a hard
  token cap — on current adaptive-reasoning models there is no reliable
  "thinking-ceiling" knob (legacy fixed-thinking-budget settings apply only to
  older model versions with adaptive reasoning explicitly disabled). The real cost
  levers are **scope, effort tier, and per-stage model choice** — not a mythical
  thinking cap.

## When to Use / When Not To

Use it for tasks that genuinely need many coordinated agents or a codified,
re-runnable orchestration: codebase-wide audits, large multi-file migrations,
security sweeps where completeness matters, cross-checked research, or a hard plan
worth drafting from several independent angles.

**Trivial-task escape hatch:** if the objective is a single-file edit, a quick
question, a naming or formatting tweak, a unit test for a known interface, or
anything that needs mid-run human input, say that Ultra-Code is the wrong tool —
the orchestration overhead adds latency and cost without adding quality — and give
the one-line ordinary prompt instead.

## Inputs

- The **task** the user wants the run to perform.
- Optionally: the **target paths / scope**, the **output deliverable** they want,
  any **model or plan constraints**, and whether this is a one-shot task or a
  sustained session of heavy work.

If scope or deliverable is missing, choose sensible defaults and state them; do
not block on the user for detail you can reasonably infer.

## Outputs

Return exactly two clearly separated parts, **prompt first**:

1. A copy-pasteable code block containing the finished prompt. If it should run as
   a single task, begin it with the keyword (the `ultra*code` form with the
   asterisk stripped in the real output you hand the user). Add no commentary
   before the block unless the user asks for explanation.
2. A short **Token-efficiency configuration** section: the tailored
   recommendation. This is half the deliverable — always include it.

Do **not** execute the task yourself. You write the brief; the user runs it.

## Core Method — writing the prompt

A good Ultra-Code prompt is an **orchestration brief**, not a chat message. Build
it from these elements (drop what does not apply; keep the control structure):

1. **Trigger + one-sentence objective.** State the single task plainly. Lead with
   the keyword for a one-shot run.
2. **Tight scope — the #1 cost lever.** Name exact paths / globs / files / a
   bounded question. Never "the whole repo" when a directory will do. Explicitly
   fence off what NOT to touch.
3. **Orchestration pattern.** Name the shape (menu below) so the script Claude
   writes matches intent instead of defaulting to a flat fan-out.
4. **Plan-first.** Require a written phase plan / target list *before* fan-out, and
   prefer reviewing the planned phases (or the raw script) at the approval prompt
   before the run starts.
5. **Quality gate.** Build in an adversarial review / verification phase:
   independent agents check or challenge findings, with false positives filtered
   out, before anything is reported.
6. **Output / synthesis spec.** Say exactly where the final result goes (e.g. a
   named markdown file), its structure, and the acceptance bar. A run returns only
   the synthesis, so specify it precisely.
7. **Model routing hint.** If some phases (extraction, mechanical scans) do not
   need the strongest model, say so — every agent uses the session model unless the
   script routes a stage down.
8. **Bounds & stop conditions.** Note expected scale, when to stop, and that the
   run must not exceed scope.
9. **Routing block.** A short block the run reads back to itself, so the routing
   decision travels with the prompt instead of living in someone's memory.

## Routing block

Embed a compact block in every prompt you write. It states, in the run's own
words, the rules it must hold to. Keep it factual: state only what the host's
routing config actually says, and never invent a model, a reviewer, or a
capability to fill a line.

```md
RUN YOURSELF:
- Resolve model identifiers, effort, and reviewers from the host's routing
  config, never from memory. An unknown identifier fails fast.
- Orchestration and review stages: the strongest workhorse rung, high effort.
- Standard execution stages: the standard rung, medium effort; high when the
  unit is demanding.
- Mechanical stages (extraction, classification, boilerplate): the mechanical
  rung, medium effort.
- Cap effort at high unless the operator stated a need for more in this case.
- Any code this run proposes for merge still passes both review axes:
  correctness and security, each an independent reviewer with its own verdict.
- Stop and report on a true human decision, a policy conflict, or a scope
  conflict. Do not decide it inside the run.
- Long stages get a bounded watcher and a deadline; a stall alerts rather than
  stranding the run.
```

**Reserve-plan rule.** An orchestration run is one of the few things that can
spend a week of quota in a single prompt. Arm it on the workhorse plan. If the
operator keeps a metered reserve model for interactive reasoning, this feature
must be disabled at the harness level on that plan, not merely avoided by
discipline — see `mythos-reserve-routing.md`, hard rule 6. When the user asks
for a run and the only available plan is the reserve, say so plainly and give
the bounded single-agent alternative instead.

## Pattern menu (orchestration shapes)

Pick the pattern that matches the task and name it in the prompt:

- **Fan-out → synthesize** — split a broad task across N parallel agents, then
  aggregate one report. *(codebase audits, multi-file analysis, broad sweeps)*
- **Classify → act (route)** — classify each item, delegate to the right
  specialized agent. *(triage, mixed workloads)*
- **Adversarial verification** — a worker produces; independent verifiers
  challenge and confirm. *(security reviews, correctness-critical work)*
- **Generate → filter** — generate many candidates, then prune or rank down.
  *(idea generation, candidate-bug lists with false-positive pruning)*
- **Tournament (generate → judge)** — several agents attempt the same task; a judge
  picks or merges the best. *(designs, hard plans, competing approaches)*
- **Loop until done** — iterate until a condition is met. *(fix-until-tests-pass,
  refine-until-threshold)*

## Prompt Template

Adapt to the objective; remove irrelevant lines, keep the structure. The keyword
is shown broken (`ultra*code`); strip the asterisk in the version you hand the
user.

```md
ultra*code: {{one-sentence objective}}

Scope (stay strictly inside this):
- Targets: {{exact paths / globs / files / bounded question}}
- Out of scope / do not touch: {{fences}}

Orchestration shape:
- Pattern: {{fan-out→synthesize | classify→act | adversarial-verification | generate→filter | tournament | loop-until-done}}
- Plan first: produce the phase plan and the concrete target list before spawning agents; surface it for approval.

Phases:
1. Analyze: {{understand the targets / build the work list}}
2. Execute: fan out across {{unit, e.g. per-file / per-route / per-source}}; {{what each agent does}}.
3. Verify: independent agents adversarially review the findings; drop unconfirmed / false-positive results.
4. Synthesize: write {{output file and format}} with {{required contents}}.

Model routing:
- Use a smaller model for {{cheap phases}}; reserve the strongest model for {{judgment phases}}.

Output:
- Deliverable: {{e.g. a report at docs/audit.md}}
- Must contain: {{sections, evidence, line-reference citations}}
- Acceptance: {{what "done" looks like}}
```

## Token-efficiency configuration (always include, tailored to the task)

After the prompt, recommend the cheapest setup that still clears the bar. Decide
each line for *this* task rather than dumping defaults:

- **Effort tier — biggest lever.** Default to using the **`ultra*code` keyword on a
  single prompt at a high (not top) effort tier** — you get the orchestration
  without forcing the top tier on every turn. Reserve session-wide Ultra-Code for a
  sustained stretch of genuinely hard tasks. Many runs are fine at high or even
  medium effort; state which you recommend and why, and tell the user to drop back
  to a normal tier the moment the heavy task is done.
- **Scope calibration / slice-first.** Recommend running on one directory or a
  narrow question first to gauge spend before committing to the full target; the
  full run is opt-in only after the slice looks right.
- **Per-stage model routing.** Call out which phases can use a smaller model;
  remind the user to check the active model before a large run if they normally
  switch to a cheaper one for routine work.
- **Plan gate.** Review the planned phases (or view the raw script) at the approval
  prompt before the run — cheaper to fix the plan than to burn a bad run.
- **Live monitoring & stop.** Watch the run dashboard for per-agent token totals;
  stop early without losing completed work; the concurrency and total-agent caps
  bound a runaway script.
- **Run hygiene.** Clear context (or start a fresh session) before a heavy run so
  prior context does not compound. Pre-add the shell / web / tool commands the
  agents need to the allowlist so the run is not stalled by mid-run permission
  prompts.
- **Amortize repeats.** If this is a recurring process, save the run's script as a
  reusable command so future runs reuse the orchestration instead of re-planning.

End with a one-line bottom line: the single recommended effort tier + scope for
this task.

## Private Adversarial Pass

Before returning, revise until these hold:

- Is the scope genuinely tight, or did I leave a "whole-repo" loophole that will
  explode cost?
- Did I pick a *specific* orchestration pattern, or default to a vague fan-out?
- Is there a plan gate and a verification phase, so a wrong turn is caught before
  fan-out and false positives are filtered before reporting?
- Is the output deliverable named and its contents specified (the run returns only
  the synthesis)?
- Did I recommend the **cheapest** effort tier that still clears the bar —
  keyword-on-high before session-wide Ultra-Code — and justify it?
- Is this even worth an orchestration run, or should it be an ordinary prompt?
  (Invoke the trivial-task escape hatch if so.)
- **Keyword safety:** in my reasoning and templates, does the literal keyword stay
  in the broken `ultra*code` form, and does the *final prompt I hand the user*
  strip the asterisk so it actually fires? Never the reverse.

## What NOT to Do

- Do not let the bare, unbroken keyword appear anywhere except the final prompt
  delivered to the user. In your own reasoning keep it as `ultra*code`; a bare
  keyword can arm a run if the text reaches a live input box.
- Do not ship the broken `ultra*code` (with the asterisk) in the *final* prompt —
  it will not fire. Strip the asterisk there, and only there.
- Do not execute the task; this skill writes prompts.
- Do not recommend session-wide top-tier effort when a single keyword-triggered
  prompt at a lower tier will do.
- Do not aim a run at an entire repository when a directory or a bounded question
  does the job.
- Do not invent configuration knobs (e.g. a hard "thinking ceiling") that the
  platform does not expose; the real levers are scope, effort tier, and per-stage
  model.
- Do not use this skill for trivial single-file or single-question tasks.

## Installation Recipe (implementation-agnostic)

An agent installing this skill on a fresh system should:

1. Create the skill, prompt, or command entry at the location its harness uses to
   surface skills, and give it a description that triggers on the phrases above
   ("prompt for ultra-code", "dynamic orchestration prompt", etc.).
2. Preserve the **keyword-safety convention** verbatim — it is the part most likely
   to be silently dropped and the part that keeps the spec inert if its text ever
   reaches a live input box.
3. Keep the **two-part output contract** (prompt first, then the token-efficiency
   configuration) — both halves are required.
4. Before relying on specific mechanics (version numbers, agent caps, the exact
   keyword and effort-tier names, the dashboard and approval flows), have the
   installing agent verify them against the current official Claude Code docs, as
   this feature changes quickly.
5. If the host has organization-wide LLM instruction files, add a one-line pointer
   so agents discover the skill.

## Smoke Test

Deterministic and offline; no network or live run required:

- **Input:** a sample task, e.g. "write a prompt to audit every API route for
  missing auth checks."
- **Assert:** the output has two parts — a fenced prompt block first, then a
  token-efficiency configuration section.
- **Assert (keyword safety):** the bare keyword appears nowhere in the prose or
  reasoning; the literal token shows up only as `ultra*code` inside code, and the
  *delivered prompt* leads with the un-broken keyword so it would fire in a real
  session. Neither rule is violated in the other direction.
- **Assert:** the prompt names a specific scope (not "whole repo"), a specific
  orchestration pattern, a plan-first step, a verification phase, and a named
  output deliverable.
- **Assert (escape hatch):** given a trivial task ("rename one variable"), the
  skill declines the run and returns an ordinary one-line prompt instead.
