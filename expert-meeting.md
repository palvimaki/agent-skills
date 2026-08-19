# Expert Meeting Skill Specification

This document defines a reusable expert meeting skill. It is intentionally provider-agnostic, model-agnostic, account-agnostic, and machine-agnostic.

The skill can be implemented by any capable LLM agent that can call one or more LLMs through whatever local interface is available and write structured artifacts.

It is the companion to the `code-review-panel` skill. The two share the same two-expert conversation pattern. The difference is that `expert-meeting` works on any non-code topic, without a repository, diff, implementation phase, or verification phase.

> **Doctrine version: 2026-08-19.** Any model name, tier, or effort label in
> this repository is a worked example with a shelf life. Re-verify against
> current documentation before you install.

## Contract

- Freeze the context before any expert speaks, and record who or what produced
  the material under discussion.
- Two experts, two genuinely different families where the stack allows, neither
  from the family that authored the material.
- First takes stay independent. No expert reads another's take first.
- Preserve dissent. A meeting that always converges is not a meeting.
- Resolve routes, identifiers, and effort from the host's routing config, never
  from memory. Cap effort at high unless the user states a need.
- Advice only. This skill never acts on its own conclusions.
- Every artifact lands on a durable path before the meeting reports done.

## Skill Identity

Name: `expert-meeting`

Purpose: run a structured two-expert discussion workflow on any non-code topic, producing independent first takes, adversarial critique, alternating discussion rounds, convergence or explicit dissent, a final consensus with next actions, and durable transcript and recap artifacts.

Use this skill when the user asks for:

- an expert meeting;
- a two-expert panel on a decision or topic;
- a structured two-model discussion;
- an adversarial/dissenting view on a strategy, plan, product, hire, policy, or design direction;
- a high-confidence recommendation before committing to a non-code decision.

If the topic involves a repository, branch, commit, PR, patch, or working tree, use `code-review-panel` instead. Use `expert-meeting` for strategy, research, planning, product, writing, hiring, financial, architectural, legal, operational, or any other decision topics where structured multi-model disagreement is useful and no code review or implementation is required.

## Non-Negotiable Constraints

- Do not depend on any specific LLM provider, model, account, API, product, machine, path, or subscription.
- Use the strongest available LLMs for the task. Frontier models are preferred for high-stakes meetings; strong local models are acceptable when that is what is available.
- Prefer two genuinely different models for Expert A and Expert B, ideally from different model families, vendors, training pipelines, or inference stacks. The point is cross-model disagreement: different training data, post-training, tool behavior, and failure modes make the panel more valuable.
- If two genuinely different models are not available, use two meaningfully independent execution contexts. If only one LLM is available, run two fresh independent passes with different roles and state this limitation in the recap.
- Freeze context before asking for expert opinions, and record the origin of the
  material under discussion — who or which route produced it. An expert from the
  same family as the author of a proposal is a weaker critic of it.
- Resolve routes, model identifiers, and effort settings from the host's
  machine-readable routing config where one exists. Never from memory. An unknown
  or retired identifier fails fast.
- Cap reasoning effort at the host's normal high tier unless the user states a
  need for a higher tier in this specific case.
- Exclude a route from the material by hosting jurisdiction and provider data
  policy, not by vendor reputation. Record any skip as a meeting limitation.
- Keep the first expert takes independent. Expert B must not see Expert A's first take before producing its own first take.
- Never act on the meeting's conclusions automatically. This skill produces advice, not actions.
- Redact secrets before writing prompts, responses, transcripts, recaps, or logs.
- Distinguish confirmed evidence from inference.
- Preserve dissent. A two-expert meeting that always converges is suspect.
- Write artifacts to a durable path. A transcript that lives only in a temporary
  directory is a transcript the user will not have tomorrow.

## Timeout and Long-Running Work

Expert meetings can take a long time. Deep reasoning, large context bundles,
remote queues, cold local models, or slower hardware may make an expert pass or
discussion round run far longer than ordinary shell commands. Implementations
must expose configurable timeouts, set generous defaults for expert processes,
record progress or partial output when possible, and avoid treating slow but
active expert work as failed solely because a short generic timeout elapsed.

## Required Inputs

The skill should work with these inputs, using defaults when the user does not provide them:

- `topic`: meeting question or goal. Required. No default.
- `context`: optional background as inline text, file references, URLs, or notes. Default: empty.
- `rounds`: discussion round budget. Default: 3 for a normal meeting, 10 for a high-stakes meeting.
- `outdir`: artifact directory. Default: a dated subdirectory under a project-level expert-panels directory.
- `expert_a`: first LLM execution route. Default: strongest available discussion-capable route.
- `expert_b`: second LLM execution route. Default: strongest independent or dissent-capable route.
- `dissent_expert`: adversarial reviewer role. Default: Expert B, or Expert A in a fresh adversarial context.

An LLM execution route can be any local shell-invokable CLI, local model server wrapper, subscription CLI, self-hosted model interface, or agent facility that accepts a prompt and returns a response. The skill should not care how the route is backed.

## Required Outputs

Always write these artifacts:

- `raw.jsonl`: append-only machine-readable event log.
- `discussion.md`: full readable transcript.
- `recap.md`: concise executive recap.

Optionally also write:

- `discussion.html`: rendered transcript.
- `recap.html`: rendered recap.

Artifacts must contain enough information to understand what was discussed, what context was frozen, what each expert said, where the experts disagreed, whether they converged, what the final recommendation is, what dissent remains, and what the next actions are.

## Workflow

1. Parse the topic and resolve the context.
2. Create the artifact directory.
3. Freeze the context before the first expert turn.
4. Run Expert A independent take.
5. Run Expert B independent take without showing Expert A's first take.
6. Ask one expert to synthesize an initial recommendation.
7. Ask the dissent expert to attack the recommendation adversarially.
8. Alternate Expert A and Expert B discussion turns until both converge or the round budget is exhausted.
9. Ask one expert to produce a final consensus recommendation.
10. Ask the dissent expert for a final skeptical review with an explicit status: `RECOMMEND`, `DO NOT RECOMMEND`, or `NO CONSENSUS`.
11. Write `raw.jsonl`, `discussion.md`, `recap.md`, and optional HTML artifacts.
12. Report status, artifact paths, recommendation, dissent, and next actions.

No code is written. No files outside the artifact directory are modified. No external systems are mutated.

## Context Freeze

Capture at least:

- the exact topic and question being asked;
- any background text the user provided;
- referenced files or URLs, summarized in-line when sufficient and cited when not;
- the user's stated constraints, goals, preferences, and assumptions;
- the decision deadline or urgency if known.

Cap frozen context aggressively so expert prompts remain readable. A useful rule of thumb:

- Inline background: about 40,000 characters.
- Referenced file excerpts: about 12,000 characters each, about 20 files.
- Later transcript context passed between turns: most recent relevant content, about 120,000 characters.

Prefer materials the user explicitly cited. Avoid pulling in tangential information that would dilute the experts' focus on the topic.

## Redaction

Before writing any artifact, redact:

- private key blocks;
- environment variable values whose names include `key`, `token`, `secret`, `password`, or `credential`;
- common API key, access token, session token, SSH key, cloud credential, and OAuth token formats;
- URLs containing embedded credentials;
- personal identifying information beyond what the user explicitly included;
- secrets found in prompts, command output, or model responses.

Use `[REDACTED_SECRET]` or a similarly obvious placeholder. Redaction must happen before durable writes.

## Raw Event Log

Write `raw.jsonl` as newline-delimited JSON. Recommended event kinds:

- `freeze`: topic, context summary, sources cited, assumptions recorded.
- `turn`: speaker, step, round, prompt, response, timestamp, execution route label, return status, duration.
- `summary`: final status, convergence, recommendation, dissent, next actions, artifact paths.

Prompts and responses in `raw.jsonl` must be redacted.

## Expert Roles

Expert A: primary discussant. Strong at structured reasoning, surfacing tradeoffs, and assembling a coherent recommendation.

Expert B: independent discussant. Strong at finding what Expert A missed. Expert B should preferably be a different model from Expert A, not just the same model with a different prompt.

Dissent expert: adversarial discussant. Its job is to attack weak assumptions, missing evidence, cognitive bias, premature consensus, and blind spots.

These are roles, not provider names. Any suitable model or agent can fill them, but the intended setup is two different models so the meeting benefits from different training backgrounds and failure modes.

## Prompt Templates

### Expert A Independent Take

```text
You are Expert A in a two-expert meeting panel. Use your strongest available reasoning.

Topic: <topic>
Context: <frozen context>

Rules:
- Use only the frozen context below unless you explicitly state that more data is needed.
- Give your honest independent take: recommendation, tradeoffs, and risks.
- Distinguish confirmed evidence from inference.
- Prioritize what actually matters for the decision.
- Do not propose implementation steps beyond a concise next-action list.
```

### Expert B Independent Take

```text
You are Expert B in a two-expert meeting panel. Use your strongest available reasoning.
You have not seen Expert A's take.

Topic: <topic>
Context: <frozen context>

Rules:
- Use only the frozen context below unless you explicitly state that more data is needed.
- Give your honest independent take: recommendation, tradeoffs, and risks.
- Distinguish confirmed evidence from inference.
- Prioritize what another discussant might miss.
- Do not propose implementation steps beyond a concise next-action list.
```

### Initial Recommendation Synthesis

```text
Synthesize the independent takes into an initial concrete recommendation.

Topic: <topic>

Include:
- the core recommendation;
- agreed points and disagreements;
- takes or claims that should be rejected as unsupported;
- tradeoffs;
- key risks;
- suggested next actions;
- open questions.

Prior transcript:
<transcript>

Frozen context:
<context>
```

### Adversarial Critique

```text
You are the adversarial reviewer in this meeting.

Topic: <topic>

Critique this initial recommendation before anyone commits to it:

<recommendation>

Prior transcript:
<transcript>

Find hidden assumptions, missing evidence, cognitive bias, premature consensus, and reasons this recommendation should not be adopted.
Be specific. Preserve dissent when evidence is weak.
```

### Discussion Turn

Alternate Expert A and Expert B. Each response must end with exactly one final line:

```text
CONVERGENCE: YES
```

or:

```text
CONVERGENCE: NO
```

Prompt:

```text
You are <speaker> in a two-expert meeting panel.

Topic: <topic>
Discussion round: <round>

Continue the discussion. Challenge weak claims, resolve disagreements, and move toward a defensible recommendation. Do not go beyond a concise next-action list.

Prior transcript:
<transcript>

Frozen context:
<context>

Return a concise but substantive response. State whether you converge, dissent, or need more evidence.
End with exactly one final line: CONVERGENCE: YES or CONVERGENCE: NO.
```

Convergence requires the latest Expert A discussion turn and latest Expert B discussion turn to both end with `CONVERGENCE: YES`.

### Final Consensus

```text
Synthesize the panel transcript into the final consensus recommendation.

Topic: <topic>

Include:
- the recommendation in one paragraph;
- key tradeoffs;
- agreed next actions, ordered;
- what would change the recommendation if new evidence appeared;
- rejected or unsupported claims;
- dissent notes;
- residual uncertainty.

Prior transcript:
<transcript>

Frozen context:
<context>
```

### Final Skeptical Review

```text
You are the final skeptical reviewer. Use skeptical reasoning.

Topic: <topic>

Consensus recommendation:
<consensus>

Prior transcript:
<transcript>

Return exactly one status: RECOMMEND, DO NOT RECOMMEND, or NO CONSENSUS.

If DO NOT RECOMMEND, list the reasons.
If NO CONSENSUS, list the unresolved disagreements and what evidence would resolve them.
If RECOMMEND, list the residual risks and the conditions under which the recommendation still holds.
```

## What This Skill Does Not Do

- It does not touch a repository.
- It does not run shell commands, open browsers, write code, or edit files outside the artifact directory.
- It does not execute the recommendation. Handoff to an implementer (human or agent) is the next action, not this skill's job.
- It does not deploy anything, send any external messages, or mutate any external system.

## Recap Format

`recap.md` should use this structure:

```markdown
# Expert Meeting Recap: <topic>

Status: RECOMMEND / DO NOT RECOMMEND / NO CONSENSUS
Artifacts: <artifact directory>

## Executive Summary
<one paragraph with the recommendation or the reason none was reached>

## Recommendation
<the agreed recommendation, or "no consensus" with unresolved disagreements>

## Tradeoffs
<main tradeoffs the panel weighed>

## Dissent
<preserved dissent, rejected claims, or "none">

## Next Actions
<ordered concrete next actions>

## Residual Uncertainty
<what would change the recommendation if it changed>
```

## Final User Response

The final answer to the user should be concise and lead with the result:

- status: `RECOMMEND`, `DO NOT RECOMMEND`, or `NO CONSENSUS`;
- the one-sentence recommendation;
- the top two or three tradeoffs;
- any preserved dissent;
- the next actions, ordered;
- artifact paths.

Do not paste the full transcript unless the user asks for it.

## Installation Recipe (implementation-agnostic)

An LLM installing this skill on a fresh system should:

1. Create a skill or prompt entry that its harness surfaces when the user asks
   for an expert meeting or multi-expert discussion. Use the equivalent skill,
   prompt, tool, or agent-config location for that harness.
2. Implement a runner that:
   - accepts the inputs above;
   - invokes two independent LLM execution routes via whatever local CLI or wrapper is available (no SDK imports, no direct API calls);
   - enforces the workflow, prompts, convergence rule, redaction, and artifact layout above;
   - writes `raw.jsonl`, `discussion.md`, and `recap.md`, plus optional HTML renders.
3. Pick two genuinely different LLM routes by default and record which routes were used in every run's artifacts. Allow per-invocation overrides.
4. Document how to add more routes, how to pass context inline or by file reference, and how to run a smoke test without spending real model calls.
5. Do not import any LLM provider SDK. Shell out to local CLIs only.

The runner may share code with the `code-review-panel` runner; in fact, doing so is encouraged so the two skills stay consistent.

## Companion Skills

- `expert-code-review-panel.md` — same two-expert pattern over a repository with evidence freeze, optional implementation, and verification.
- `content-presentation.md` — how the rendered `discussion.html` and `recap.html` artifacts should be surfaced when the user asks to see a meeting outcome.

## Smoke Test

The skill should ship a smoke-test mode that substitutes deterministic synthetic responses for each expert, runs the full workflow on a fixture topic, and writes artifacts. The smoke test must not require network access or real model calls, and must fail loudly if any step of the workflow (context freeze, independent takes, synthesis, adversarial critique, discussion, consensus, final skeptical review, artifact writes) is skipped.
