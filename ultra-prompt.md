# Ultra Prompt Skill Specification

This document defines a reusable skill: **given a task, write a compact,
self-complete prompt for Claude Code's Ultra-Code orchestration feature — its
`ultra*code` effort tier and the dynamic, many-subagent runs it spawns — or for
a comparable bounded deep-research or multi-worker run.**

The *subject* is deliberately product-specific: it is about one named feature,
Claude Code's Ultra-Code. The *runner* is agent-agnostic. Any capable LLM agent
that can write text can follow this spec. The agent that writes the prompt does
not have to be the model that later runs it.

No LLM API, SDK or provider key is required. The skill produces text that a
human pastes into their own session.

> **Doctrine version: 2026-10-20.** The output is now one self-complete prompt
> block plus its verified character count, with a default size limit. The
> routing block now follows tier routing, the frontier effort floor, the review
> ladder and the quota re-check rule. Product mechanics move fast: re-verify
> every version number, cap and keyword against current official documentation.

## Contract

- Write the prompt. Do not run the task unless the user asks for execution.
- Deliver one copy-pasteable Markdown code block with every run instruction
  inside it. Add no text before the block. After it, report only the verified
  character count.
- Keep the prompt under about 3,500 characters by default. Measure it (for
  example with `wc -c`) before you return it.
- Every prompt states one objective, an exact acceptance condition, a fenced
  scope, one named work shape, a plan and target list before work starts, the
  evidence each result must carry, a deliverable, and a stop condition.
- Every prompt carries a short routing block that the run reads back to itself.
  It states only what the host's `<routing config>` confirms.
- An orchestration run is armed on a non-reserve plan, never on a plan the
  operator keeps for interactive frontier thinking.
- Recommend a plain single-agent prompt when the task does not need many
  coordinated agents. Most tasks do not.
- The run stops and reports at a true human decision, a policy conflict or a
  scope conflict. It does not decide those inside the run.
- Keep the keyword-safety convention below, and strip the break only in the
  final prompt handed to the user.

## Scope note — one product's mechanics, any runner

The orchestration mechanics here belong to one named product feature. That
specificity is deliberate: the job of this skill is to write good prompts for
*that* kind of run. Two things keep it portable:

- **The runner is agent-agnostic.** Any capable agent that can write text can
  follow this spec.
- **The method transfers.** Objective, fenced scope, one work shape, plan
  first, evidence rules, a bounded stop condition and an explicit routing block
  make *any* multi-agent or long-horizon run behave. If your host calls its
  feature something else, keep the method and replace the mechanics section
  after you check that host's current documentation.

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
run, for a `deep-research` run, or for any multi-worker fan-out run.

## What Ultra-Code is (ground truth, with a shelf life)

These mechanics were written against builds in the v2.1.1xx range, where the
trigger keyword was renamed from the legacy word to `ultra*code`. Shipped
builds have moved on. **Re-verify every specific below against the live build
and current official documentation.** Where this file disagrees with the
product's documentation, the documentation wins and this file is stale.

- A **dynamic orchestration run** is a script that the model *writes* to
  coordinate subagents at scale, then runs in the background while the session
  stays responsive. The script holds the loop, the branching and the
  intermediate results, so only the **final synthesized answer** returns to the
  main context. The documented limits at the time of writing were 16 concurrent
  agents and 1,000 agents per run.
- **Ultra-Code** is a session setting that combines the highest effort tier
  with **automatic** orchestration. With it on, the model decides per task
  whether to spawn a run, and one request can become several runs in a row. It
  is session-scoped and it is the **most expensive** mode.
- **Two ways to start a run.** The cheaper one is the biggest lever:
  - **Keyword, one task (preferred):** lead the prompt with the `ultra*code`
    keyword (asterisk stripped at emit time). This runs one task as an
    orchestration without raising the whole session to the top tier.
  - **Session mode:** set the effort tier to `ultra*code`. Every substantive
    task in the session then becomes a run at top effort. Use it only for a
    sustained stretch of genuinely hard tasks.
- **Cost reality.** One run can spend more of a weekly limit than a full day of
  normal use. Effort is a behavioral signal, not a hard token cap. The real
  levers are scope, effort tier and per-stage model choice.

## When to Use / When Not To

Use it for work that needs many coordinated agents: codebase-wide audits,
large multi-file migrations, security sweeps where completeness matters,
cross-checked research, or a hard plan drafted from several independent angles.

**Trivial-task escape hatch.** Use a normal single-agent prompt when the task
is small, touches one known file, needs human decisions during the run, or has
no useful independent work. Say so, and give that prompt instead.

## Inputs

- The **task** the run must perform.
- Optionally: target paths or scope, the deliverable, model or plan limits, and
  whether this is one task or a sustained stretch of heavy work.

If scope or deliverable is missing, choose sensible defaults and state them in
the prompt. Do not block on detail you can reasonably infer.

## Outputs

1. One copy-pasteable code block that holds the finished prompt and every run
   instruction. For a one-task run, it leads with the keyword (the `ultra*code`
   form with the asterisk stripped in the real output).
2. After the block, one line: the verified character count. Nothing else,
   unless the user asks for explanation or run settings.

## Build the prompt

Track these steps:

```
- [ ] Step 1: Objective and acceptance condition
- [ ] Step 2: Tight scope and excluded work
- [ ] Step 3: One work shape
- [ ] Step 4: Plan, target list and stop condition
- [ ] Step 5: Evidence for each result
- [ ] Step 6: Deliverable format and sections
- [ ] Step 7: Routing block
- [ ] Step 8: Size and final check
```

1. **Objective and acceptance.** One task, stated plainly, with an exact
   acceptance condition.
2. **Tight scope — the main cost lever.** Name exact paths, data, questions
   and the output file. Fence off excluded work. Never "the whole repo" when a
   directory will do. For a large target, scope a first slice and make the full
   run a follow-up.
3. **One work shape** from the menu below, named in the prompt, so the script
   matches intent instead of a flat fan-out.
4. **Plan first.** A phase plan and a concrete target list before any fan-out,
   surfaced at the approval step where the host has one. A stop condition for
   true human decisions, policy conflicts and scope conflicts.
5. **Evidence.** What each result must carry: file and line references, test
   output, sources. When the acceptance surface is visible in a browser or a
   simulator, require evidence from that surface. Build in a verification phase
   where independent agents challenge the findings and drop false positives.
6. **Deliverable.** Where the result goes, its format and its required
   sections. A run returns only the synthesis, so specify it precisely.
7. **Routing block.** See below.
8. **Size and final check.** Under the size limit, every control present.

## Routing block

Embed a short `RUN YOURSELF` block in every prompt. Keep it factual. Do not
invent a model, a reviewer or a capability to fill a line. Adapt this shape to
what the host's routing config says:

```md
RUN YOURSELF:
- Resolve models, effort and reviewers from the routing config, never from
  memory. An unknown identifier fails fast.
- Bounded read-mostly stages (search, extraction, summaries, inventories,
  exact mechanical edits): the fast low-cost worker tier, as native subagents.
  The parent verifies each result; cheap output never grades itself.
- Standard execution and coordination: the daily-driver route, medium effort;
  high for demanding units.
- A unit that failed its acceptance check: one tier up, not a retry.
- No stage inherits the frontier model. A frontier stage, if the config allows
  one, runs at high effort or above.
- Subscription routes only. No metered API calls.
- Each worker gets its goal, its files and its acceptance checks, and nothing
  else.
- Code proposed for merge passes the review ladder at the exact head SHA:
  an independent correctness reviewer, plus a separate security reviewer where
  the repository requires one.
- On a usage limit: use the declared fallback, record the next real probe in a
  few hours, and continue when it works. Never switch to an undeclared
  provider.
- Stop and report a true human decision, a policy conflict or a scope conflict.
- Long stages get a bounded progress check with a deadline and a stop
  condition; it removes itself when the run ends.
```

**Reserve-plan rule.** An orchestration run can spend a week of quota in one
prompt. Arm it on a non-reserve plan. If the operator keeps a tightly metered
frontier model for interactive reasoning, disable this feature on that plan at
the harness level, not by discipline alone (see `mythos-reserve-routing.md`,
"Switch off the quota bombs"). When the only available plan is the reserve, say
so plainly and give the bounded single-agent prompt instead.

## Work shapes

Pick one and name it in the prompt:

- **Fan-out and synthesize** — split a broad task across parallel agents, then
  aggregate one report. *(audits, multi-file analysis, broad sweeps)*
- **Classify and route** — classify each item and send it to the right
  specialist. *(triage, mixed loads)*
- **Adversarial verification** — a worker produces; independent verifiers
  challenge and confirm. *(security reviews, correctness-critical work)*
- **Generate and filter** — generate many candidates, then prune or rank.
  *(ideas, candidate-bug lists with false-positive pruning)*
- **Compare alternatives** — several agents attempt the same task; a judge picks
  or merges the best. *(designs, hard plans)*
- **Iterate to a measurable condition** — repeat until a check passes.
  *(fix until tests pass, refine to a threshold)*

## Prompt Template

Adapt to the objective; remove lines that do not apply, keep the structure. The
keyword is shown broken (`ultra*code`); strip the asterisk in the version you
hand the user.

```md
ultra*code: {{one-sentence objective}}

RUN YOURSELF:
- Resolve models, effort and reviewers from the routing config; never from memory.
- {{routing choices for this task}}
- {{bounded progress check, if this is a long run}}

Scope:
- Targets: {{exact paths, records, or questions}}
- Excluded: {{clear boundary}}

Plan:
- Make a target list and phase plan before execution; surface it for approval.
- Stop and report a true human decision, a policy conflict, or a scope conflict.

Work shape: {{selected work shape}}

Phases:
1. Analyze: {{bounded analysis; build the work list}}
2. Execute: {{independent work units and what each agent does}}
3. Verify: {{independent checks; drop unconfirmed results}}
4. Deliver: {{file, format, and required sections}}

Acceptance:
- {{measurable completion condition}}
- {{evidence each result carries; browser-visible evidence if applicable}}
```

## Cost levers (apply while writing; advise only on request)

Build the cheapest run that still clears the bar into the prompt itself:

- **Keyword on one task, not session mode.** Lead with the keyword at the
  session's normal high tier. Session-wide Ultra-Code is for a sustained
  stretch of genuinely hard tasks only.
- **Slice first.** Run on one directory or one narrow question to gauge spend;
  the full run is a follow-up.
- **Per-stage routing.** Put every read-mostly stage on the fast worker tier in
  the routing block.
- **Plan gate.** A wrong plan is cheaper to fix at the approval step than after
  a bad run.

When the user asks how to run it, also give: check the active model and effort
before a large run; clear context or start a fresh session first; pre-allow the
commands the agents need so the run does not stall on permission prompts; watch
per-agent token totals and stop early if spend runs away; save a recurring
run's script as a reusable command.

## Final check

Before returning, confirm:

- The scope is tight, with no whole-repo loophole.
- One specific work shape is named.
- There is a plan gate, a verification phase and a stop condition.
- The deliverable is named and its contents are specified.
- The routing block is present and states only what the routing config
  confirms.
- The block is under the size limit, measured, and the count is reported.
- The task is worth an orchestration run at all; if not, the escape hatch
  applies.
- **Keyword safety:** in reasoning and templates the keyword stays in the
  broken `ultra*code` form, and the *final prompt handed to the user* strips the
  asterisk so it fires. Never the reverse.

## What NOT to Do

- Do not let the bare, unbroken keyword appear anywhere except the final prompt
  delivered to the user. In your own reasoning keep it as `ultra*code`; a bare
  keyword can arm a run if the text reaches a live input box.
- Do not ship the broken `ultra*code` (with the asterisk) in the *final* prompt —
  it will not fire. Strip the asterisk there, and only there.
- Do not execute the task unless the user asks for execution.
- Do not put text before the prompt block, or commentary after it beyond the
  character count, unless the user asks.
- Do not recommend session-wide top-tier effort when one keyword-led prompt
  will do.
- Do not aim a run at an entire repository when a directory or a bounded
  question does the job.
- Do not invent configuration knobs (for example a hard "thinking ceiling")
  that the platform does not expose.
- Do not put routing claims in the block that the routing config does not
  confirm.

## Installation Recipe (implementation-agnostic)

An agent installing this skill on a fresh system should:

1. Create the skill, prompt or command entry where its harness surfaces skills,
   with a description that triggers on the phrases above.
2. Preserve the **keyword-safety convention** verbatim. It is the part most
   likely to be dropped, and the part that keeps the spec inert if its text
   reaches a live input box.
3. Keep the **output contract**: one self-complete block, then the verified
   character count.
4. Fill the routing block from the host's own routing config: its tiers, its
   review ladder, its fallbacks. Leave out any line the config does not
   support.
5. Verify the mechanics (version numbers, agent caps, keyword and effort-tier
   names, approval and dashboard flows) against current official
   documentation, because this feature changes quickly.
6. If the host has organization-wide instruction files, add a one-line pointer.

## Smoke Test

Deterministic and offline; no live run required:

- **Input:** "Write a prompt to audit every API route for missing auth checks."
- **Assert:** the output is one fenced block followed only by a character count,
  and the count matches the block and is under the size limit.
- **Assert (keyword safety):** the bare keyword appears nowhere in prose or
  reasoning; it shows up only as `ultra*code` inside code, and the *delivered
  prompt* leads with the unbroken keyword. Neither rule is broken the other way.
- **Assert:** the prompt names a specific scope, one work shape, a plan-first
  step, a verification phase, an evidence rule, a named deliverable, a stop
  condition and a routing block.
- **Assert (routing):** read-mostly stages go to the fast worker tier, no stage
  inherits the frontier model, and merge-bound code passes the review ladder at
  the exact head SHA.
- **Assert (escape hatch):** given "rename one variable", the skill declines
  the run and returns an ordinary one-line prompt.
