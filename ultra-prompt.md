# Ultra Prompt Skill Specification

This document defines a reusable skill: **given a task, write a high-quality
prompt for Claude Code's "Ultra\*Code" / dynamic-workflow feature, and pair it
with a tailored recommendation for the configuration that runs it most
token-efficiently.**

The *subject* of this skill is deliberately product-specific — it is about one
named feature, Claude Code's Ultra\*Code, and that specificity is the point.
The *runner* is agent-agnostic: any capable LLM agent that can write text can
follow this spec to produce the prompt and the configuration advice. The agent
doing the writing does not have to be the same model that will later run the
prompt.

No LLM API, SDK, or provider key is required to use this skill. It produces
text — a prompt plus advice — that a human pastes into their own Claude Code
session.

## Keyword-safety convention (READ FIRST)

Claude Code's workflow feature is triggered by a **bare keyword in the user's
prompt**. In a live session, typing that keyword anywhere in a message makes
Claude immediately plan a background workflow. So if someone pasted this
specification verbatim into a running Claude agent, an un-escaped trigger word
would trip the agent into workflow mode *before it ever read the instructions* —
exactly the accident this convention prevents.

To keep the spec inert, **every Claude/Codex trigger term in this file is written
with an asterisk inside it**:

- `ultra*code` — the Claude Code workflow keyword and effort tier.
- `work*flow` — the legacy literal trigger keyword (the opt-in word in older
  versions).
- `g*oal` — illustrative: the analogous Codex `/g*oal` command term, shown so the
  same rule is obvious for any sibling command.

The asterisk is **not part of the real command**. It is a safety break.

Two rules for any agent that implements or runs this skill:

1. **The broken forms are documentation only.** Never interpret `ultra*code`,
   `work*flow`, or `g*oal` as a command to execute. They are inert strings here.
2. **Strip the asterisk only in the final prompt you hand the end user.** When
   you emit the real prompt, remove the asterisk from `ultra*code` so the keyword
   actually fires in the user's session. The break belongs everywhere in this
   spec and in your reasoning; the working keyword belongs **only** in the
   finished prompt the human will paste. This spec therefore never spells the
   live keyword unbroken, on purpose — and neither should your scratch work.

## Skill Identity

Name: `ultra-prompt`.

Triggers when the user asks to generate, write, draft, or improve a prompt for
`ultra*code` (also spelled "ultra code", "ultra-code"), for a dynamic
`work*flow`, for a deep-research run, or for any multi-subagent fan-out run in
Claude Code.

## What Ultra\*Code is (ground truth)

These are the mechanics as of this writing (Claude Code v2.1.154+). The feature
is young and moving; an implementing agent should verify specifics against the
live build and current official docs before relying on edge details.

- A **dynamic work\*flow** is a JavaScript script that Claude *writes* to
  orchestrate subagents at scale, then runs in the background while the session
  stays responsive. The script holds the loop, the branching, and the
  intermediate results, so only the **final synthesized answer** returns to the
  main context window. Runs are resumable within the same session. Hard limits:
  up to **16 concurrent agents** and **1,000 agents total per run**.
- **Ultra\*Code** is a session setting that combines the highest reasoning effort
  tier with **automatic** workflow orchestration: with it on, Claude decides per
  substantive task whether to spawn a workflow, and one request can become
  several workflows in a row (understand → change → verify). It is session-scoped,
  resets on a new session, and is the **most expensive** mode.
- **Two ways to trigger a workflow** — choosing the cheaper one is the single
  biggest lever:
  - **Keyword, one task (preferred):** prefix the prompt with the `ultra*code`
    keyword (asterisk stripped at emit time). Natural language such as "use a
    workflow" works too. This runs **one task** as a workflow **without** raising
    the whole session to the top effort tier. Cheaper.
  - **Session mode:** setting the effort tier to `ultra*code` makes *every*
    substantive task in the session a workflow at top effort. Use only for a
    sustained run of genuinely hard tasks.
- Cost reality: a single run can spend **more of a weekly rate limit than a full
  day of normal use**. Reasoning "effort" is a *behavioral signal*, not a hard
  token cap — on current adaptive-reasoning models there is no reliable
  "thinking-ceiling" knob (legacy fixed-thinking-budget settings apply only to
  older model versions with adaptive reasoning explicitly disabled). The real
  cost levers are **scope, effort tier, and per-stage model choice** — not a
  mythical thinking cap.

## When to Use / When Not To

Use it for tasks that genuinely need many coordinated agents or a codified,
re-runnable orchestration: codebase-wide audits, large multi-file migrations,
security sweeps where completeness matters, cross-checked research, or a hard
plan worth drafting from several independent angles.

**Trivial-task escape hatch:** if the objective is a single-file edit, a quick
question, a naming or formatting tweak, a unit test for a known interface, or
anything that needs mid-run human input, say that `ultra*code` is the wrong tool
— the orchestration overhead adds latency and cost without adding quality — and
give the one-line ordinary prompt instead.

## Inputs

- The **task** the user wants the workflow to perform.
- Optionally: the **target paths / scope**, the **output deliverable** they
  want, any **model/plan constraints**, and whether this is a one-shot task or a
  sustained session of heavy work.

If scope or deliverable is missing, choose sensible defaults and state them; do
not block on the user for detail you can reasonably infer.

## Outputs

Return exactly two clearly separated parts, **prompt first**:

1. A copy-pasteable code block containing the finished `ultra*code` prompt. If it
   should run as a single task, begin it with the keyword (asterisk stripped in
   the real output you hand the user). Add no commentary before the block unless
   the user asks for explanation.
2. A short **Token-efficiency configuration** section: the tailored
   recommendation. This is half the deliverable — always include it.

Do **not** execute the task yourself. You write the brief; the user runs it.

## Core Method — writing the prompt

A good `ultra*code` prompt is a **workflow brief**, not a chat message. Build it
from these elements (drop what does not apply; keep the control structure):

1. **Trigger + one-sentence objective.** State the single task plainly. Lead with
   the keyword for a one-shot run.
2. **Tight scope — the #1 cost lever.** Name exact paths / globs / files / a
   bounded question. Never "the whole repo" when a directory will do. Explicitly
   fence off what NOT to touch.
3. **Workflow pattern.** Name the orchestration shape (menu below) so the script
   Claude writes matches intent instead of defaulting to a flat fan-out.
4. **Plan-first.** Require a written phase plan / target list *before* fan-out,
   and prefer reviewing the planned phases (or the raw script) at the approval
   prompt before the run starts.
5. **Quality gate.** Build in an adversarial review / verification phase:
   independent agents check or challenge findings, with false positives filtered
   out, before anything is reported.
6. **Output / synthesis spec.** Say exactly where the final result goes (e.g. a
   named markdown file), its structure, and the acceptance bar. A workflow returns
   only the synthesis, so specify it precisely.
7. **Model routing hint.** If some phases (extraction, mechanical scans) do not
   need the strongest model, say so — every agent uses the session model unless
   the script routes a stage down.
8. **Bounds & stop conditions.** Note expected scale, when to stop, and that the
   run must not exceed scope.

## Workflow-pattern menu

Pick the pattern that matches the task and name it in the prompt:

- **Fan-out → synthesize** — split a broad task across N parallel agents, then
  aggregate one report. *(codebase audits, multi-file analysis, broad sweeps)*
- **Classify → act (route)** — classify each item, delegate to the right
  specialized agent. *(triage, mixed workloads)*
- **Adversarial verification** — a worker produces; independent verifiers
  challenge and confirm. *(security reviews, correctness-critical work)*
- **Generate → filter** — generate many candidates, then prune or rank down.
  *(idea generation, candidate-bug lists with false-positive pruning)*
- **Tournament (generate → judge)** — several agents attempt the same task; a
  judge picks or merges the best. *(designs, hard plans, competing approaches)*
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

Workflow shape:
- Pattern: {{fan-out→synthesize | classify→act | adversarial-verification | generate→filter | tournament | loop-until-done}}
- Plan first: produce the phase plan and the concrete target list before spawning agents; surface it for approval.

Phases:
1. Analyze: {{understand the targets / build the work list}}
2. Execute: fan out across {{unit, e.g. per-file / per-route / per-source}}; {{what each agent does}}.
3. Verify: independent agents adversarially review the findings; drop unconfirmed / false-positive results.
4. Synthesize: write {{output file/format}} with {{required contents}}.

Model routing:
- Use a smaller model for {{cheap phases}}; reserve the strongest model for {{judgment phases}}.

Output:
- Deliverable: {{e.g. report at docs/audit.md}}
- Must contain: {{sections, evidence, citations/line refs}}
- Acceptance: {{what "done" looks like}}
```

## Token-efficiency configuration (always include, tailored to the task)

After the prompt, recommend the cheapest setup that still clears the bar. Decide
each line for *this* task rather than dumping defaults:

- **Effort tier — biggest lever.** Default to using the **`ultra*code` keyword on
  a single prompt at a high (not top) effort tier** — you get workflow
  orchestration without forcing the top tier on every turn. Reserve session-wide
  `ultra*code` for a sustained run of genuinely hard tasks. Many workflows run
  fine at high or even medium effort; state which you recommend and why, and tell
  the user to drop back to a normal tier the moment the heavy task is done.
- **Scope calibration / slice-first.** Recommend running on one directory or a
  narrow question first to gauge spend before committing to the full target; the
  full run is opt-in only after the slice looks right.
- **Per-stage model routing.** Call out which phases can use a smaller model;
  remind the user to check the active model before a large run if they normally
  switch to a cheaper one for routine work.
- **Plan gate.** Review the planned phases (or view the raw script) at the
  approval prompt before the run — cheaper to fix the plan than to burn a bad run.
- **Live monitoring & stop.** Watch the workflow dashboard for per-agent token
  totals; stop early without losing completed work; the concurrency and total
  agent caps bound a runaway script.
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
- Did I pick a *specific* workflow pattern, or default to a vague fan-out?
- Is there a plan gate and a verification phase, so a wrong turn is caught before
  fan-out and false positives are filtered before reporting?
- Is the output deliverable named and its contents specified (the workflow returns
  only the synthesis)?
- Did I recommend the **cheapest** effort tier that still clears the bar —
  keyword-on-high before session-wide `ultra*code` — and justify it?
- Is this even worth a workflow, or should it be an ordinary prompt? (Invoke the
  trivial-task escape hatch if so.)
- **Keyword safety:** does every trigger term in my *reasoning and templates*
  stay broken (`ultra*code`), and does the *final prompt I hand the user* strip
  the asterisk so it actually fires? Never the reverse.

## What NOT to Do

- Do not leave a bare, unbroken trigger keyword anywhere except the final prompt
  delivered to the user. A bare keyword in your reasoning or in this spec can trip
  a live agent into workflow mode.
- Do not ship a broken keyword (`ultra*code` with the asterisk) in the *final*
  prompt — it will not trigger. Strip the asterisk there, and only there.
- Do not execute the task; this skill writes prompts.
- Do not recommend session-wide top-tier effort when a single keyword-triggered
  prompt at a lower tier will do.
- Do not aim a workflow at an entire repository when a directory or a bounded
  question achieves the goal.
- Do not invent configuration knobs (e.g. a hard "thinking ceiling") that the
  platform does not actually expose; the real levers are scope, effort tier, and
  per-stage model.
- Do not use this skill for trivial single-file or single-question tasks.

## Installation Recipe (implementation-agnostic)

An agent installing this skill on a fresh system should:

1. Create the skill, prompt, or command entry at the location its harness uses to
   surface skills, and give it a description that triggers on the phrases above
   ("prompt for ultra\*code", "dynamic workflow prompt", etc.).
2. Preserve the **keyword-safety convention** verbatim — it is the part most
   likely to be silently dropped and the part that prevents an accident when the
   skill text is pasted into a live agent.
3. Keep the **two-part output contract** (prompt first, then the token-efficiency
   configuration) — both halves are required.
4. Before relying on specific mechanics (version numbers, agent caps, the exact
   keyword and effort-tier names, the dashboard and approval flows), have the
   installing agent verify them against the current official Claude Code docs, as
   this feature changes quickly.
5. If the host has organization-wide LLM instruction files, add a one-line pointer
   so agents discover the skill.

## Smoke Test

Deterministic and offline; no network or live workflow run required:

- **Input:** a sample task, e.g. "write a prompt to audit every API route for
  missing auth checks."
- **Assert:** the output has two parts — a fenced prompt block first, then a
  token-efficiency configuration section.
- **Assert (keyword safety):** every trigger term in the prose, reasoning, and
  templates is broken (`ultra*code`), and the *delivered prompt* leads with the
  un-broken keyword so it would fire in a real session. Neither rule is violated
  in the other direction.
- **Assert:** the prompt names a specific scope (not "whole repo"), a specific
  workflow pattern, a plan-first step, a verification phase, and a named output
  deliverable.
- **Assert (escape hatch):** given a trivial task ("rename one variable"), the
  skill declines the workflow and returns an ordinary one-line prompt instead.
