# Expert Code Review Panel Skill Specification

This document defines a reusable expert code review panel skill. It is intentionally provider-agnostic, model-agnostic, account-agnostic, and machine-agnostic.

The skill can be implemented by any capable LLM agent that can inspect a git repository, run shell commands on macOS or Linux, call one or more LLMs through whatever local interface is available, and write review artifacts.

## Skill Identity

Name: `code-review-panel`

Purpose: run a structured two-expert code review workflow that freezes repository evidence, gathers independent expert reviews, applies adversarial critique, drives the experts toward consensus, optionally implements agreed fixes, verifies the result, and writes durable transcript and recap artifacts.

Use this skill when the user asks for:

- an expert code review panel;
- a two-model or two-expert review;
- an adversarial/dissenting review;
- a high-confidence review before merge, release, or deploy;
- a structured review of a repository, branch, commit, PR, patch, or working tree.

## Companion Variant: Expert Meeting

A closely related skill can be created from this same specification: `expert-meeting`.

The expert meeting variant runs the same two-expert discussion pattern on any non-code topic the user wants. It keeps the independent first takes, adversarial critique, alternating discussion rounds, convergence requirement, final consensus, dissent notes, transcript, and recap.

The differences are:

- no git repository or code ref is required;
- the input is a topic/question plus optional background context;
- the evidence freeze is replaced by a context freeze: user-provided materials, cited files, URLs, notes, or assumptions gathered before the first expert turn;
- implementation and code verification are omitted unless the user explicitly asks for an action plan to be executed;
- the recap focuses on recommendations, tradeoffs, dissent, uncertainties, and next actions instead of findings, patches, tests, and release risk.

Use the expert meeting variant for strategy, research, planning, product, writing, hiring, financial, architectural, or other decision topics where structured multi-model disagreement is useful.

## Non-Negotiable Constraints

- Do not depend on any specific LLM provider, model, account, API, product, machine, path, or subscription.
- Use the strongest available LLMs for the task. Frontier models are preferred for high-risk reviews; strong local models are acceptable when that is what is available.
- Prefer two genuinely different models for Expert A and Expert B, ideally from different model families, vendors, training pipelines, or inference stacks. The point is cross-model disagreement: different training data, post-training, tool behavior, and failure modes make the panel more valuable.
- If two genuinely different models are not available, use two meaningfully independent execution contexts. If only one LLM is available, run two fresh independent passes with different roles and state this limitation in the recap.
- Freeze the repository evidence before asking for expert opinions.
- Keep the first expert reviews independent. Expert B must not see Expert A's first review before producing its own first review.
- Do not implement changes unless the user explicitly asks for fixes to be applied.
- Never discard or overwrite unrelated user changes.
- Never deploy to production from this skill.
- Redact secrets before writing prompts, responses, transcripts, recaps, or logs.
- Distinguish confirmed evidence from inference.

## Timeout and Long-Running Work

Expert panels can take a long time. Large diffs, cold local models, remote
queues, high-reasoning settings, or slower hardware may make one expert or
discussion round run far longer than ordinary shell commands. Implementations
must expose configurable timeouts, set generous defaults for expert processes,
record progress or partial output when possible, and avoid treating slow but
active expert work as failed solely because a short generic timeout elapsed.

## Required Inputs

The skill should work with these inputs, using defaults when the user does not provide them:

- `topic`: review question or goal. Default: summarize the user's request.
- `repo`: repository path. Default: current working directory.
- `ref`: branch, commit, PR checkout, or revision to review. Default: `HEAD`.
- `focus`: optional path, subsystem, risk area, or review theme. Default: correctness, security, reliability, tests, and release risk.
- `rounds`: discussion round budget. Default: 3 for normal review, 10 for high-risk review.
- `implement`: whether to apply agreed fixes. Default: false.
- `outdir`: artifact directory. Default: a dated directory under `research/ops/expert-panels/`.
- `expert_a`: first LLM execution route. Default: strongest available review-capable route.
- `expert_b`: second LLM execution route. Default: strongest independent or dissent-capable route.
- `dissent_expert`: final skeptical reviewer. Default: Expert B, or Expert A in a fresh adversarial context.
- `implementation_expert`: implementation route if fixes are requested. Default: strongest coding-capable route.

An LLM execution route can be any local shell-invokable CLI, local model server wrapper, subscription CLI, self-hosted model interface, or agent facility that accepts a prompt and returns a response. The skill should not care how the route is backed.

## Required Outputs

Always write these artifacts:

- `raw.jsonl`: append-only machine-readable event log.
- `discussion.md`: full readable transcript.
- `recap.md`: concise executive recap.

Optionally also write:

- `discussion.html`: rendered transcript.
- `recap.html`: rendered recap.

Artifacts must contain enough information to understand what was reviewed, which evidence was frozen, what each expert said, where the experts disagreed, whether they converged, what was implemented, what was verified, and what risk remains.

## Workflow

1. Resolve the repository and review ref.
2. Create the artifact directory.
3. Freeze git evidence before expert review.
4. Run Expert A independent review.
5. Run Expert B independent review without showing Expert A's first review.
6. Ask one expert to synthesize an initial fix plan.
7. Ask the dissent expert to attack the plan adversarially.
8. Alternate Expert A and Expert B discussion turns until both converge or the round budget is exhausted.
9. Ask one expert to produce a final consensus.
10. Ask the dissent expert for a final gate review: `GO`, `NOGO`, or `NO CONSENSUS`.
11. If implementation was requested, run a fresh implementation pass constrained to the final consensus.
12. Run relevant verification commands.
13. Write `raw.jsonl`, `discussion.md`, `recap.md`, and optional HTML artifacts.
14. Report status, artifact paths, findings, verification, changed files, and residual risk.

## Evidence Freeze

Capture at least:

```bash
git -C "$REPO" rev-parse "$REF^{commit}"
git -C "$REPO" status --short
git -C "$REPO" log -5 --decorate --oneline "$REF"
git -C "$REPO" diff --stat "$REF~5..$REF" || git -C "$REPO" diff --stat "$REF^..$REF"
git -C "$REPO" diff "$REF~5..$REF" || git -C "$REPO" diff "$REF^..$REF"
git -C "$REPO" diff --name-only "$REF~5..$REF" || git -C "$REPO" diff --name-only "$REF^..$REF"
```

If those ranges are invalid, fall back to the nearest valid parent, merge base, or current working tree diff. Record which fallback was used.

Include focused file contents when useful. Cap evidence aggressively so expert prompts remain readable:

- Diff: about 80,000 characters.
- Individual file: about 12,000 characters.
- Snapshot files: about 40 files.
- Later transcript context: most recent relevant content, about 120,000 characters.

Prefer files that are changed, in focus, imported by changed files, or test-related. Avoid vendored dependencies, build artifacts, generated lockfile noise, and binary files unless directly relevant.

## Redaction

Before writing any artifact, redact:

- private key blocks;
- environment variable values whose names include `key`, `token`, `secret`, `password`, or `credential`;
- common API key, access token, session token, SSH key, cloud credential, and OAuth token formats;
- URLs containing embedded credentials;
- secrets found in prompts, command output, diffs, or model responses.

Use `[REDACTED_SECRET]` or a similarly obvious placeholder. Redaction must happen before durable writes.

## Raw Event Log

Write `raw.jsonl` as newline-delimited JSON. Recommended event kinds:

- `freeze`: repository, ref, resolved commit, focus, status, changed files, selected files, captured diff summary.
- `turn`: speaker, step, round, prompt, response, timestamp, execution route label, return status, duration.
- `verification`: command, exit code, duration, stdout tail, stderr tail.
- `implementation`: whether implementation ran, files changed, commit/branch/PR if any, implementation notes.
- `summary`: final status, convergence, findings, residual risk, artifact paths.

Prompts and responses in `raw.jsonl` must be redacted.

## Expert Roles

Expert A: primary reviewer. Strong at code correctness, architecture, reliability, and implementation planning.

Expert B: independent reviewer. Strong at finding what Expert A missed. Expert B should preferably be a different model from Expert A, not just the same model with a different prompt.

Dissent expert: adversarial reviewer. Its job is to attack weak assumptions, missing tests, security issues, workflow risk, and premature consensus.

Implementation expert: fresh coding context used only when the user requested applying fixes.

These are roles, not provider names. Any suitable model or agent can fill them, but the intended setup is two different models so the review benefits from different training backgrounds and failure modes.

## Prompt Templates

### Expert A Independent Review

```text
You are Expert A in a two-expert code review panel. Use your strongest available reasoning.

Topic: <topic>
Repository: <repo>
Ref: <ref>
Focus: <focus>

Rules:
- Use only the frozen evidence below unless you explicitly state that more data is needed.
- Identify issues with severity P0/P1/P2/P3.
- For each issue, include evidence, impact, and proposed fix.
- Distinguish confirmed evidence from inference.
- Prioritize security, correctness, reliability, tests, and release risk.
- Do not implement code in this review turn.

Frozen evidence:
<evidence>
```

### Expert B Independent Review

```text
You are Expert B in a two-expert code review panel. Use your strongest available reasoning.
You have not seen Expert A's review.

Topic: <topic>
Repository: <repo>
Ref: <ref>
Focus: <focus>

Rules:
- Use only the frozen evidence below unless you explicitly state that more data is needed.
- Identify issues with severity P0/P1/P2/P3.
- For each issue, include evidence, impact, and proposed fix.
- Distinguish confirmed evidence from inference.
- Prioritize what another reviewer might miss.
- Do not implement code in this review turn.

Frozen evidence:
<evidence>
```

### Initial Plan Synthesis

```text
Synthesize the independent reviews into an initial concrete fix plan.

Topic: <topic>

Include:
- confirmed findings by severity;
- disagreements or weak evidence;
- findings that should be rejected as unsupported;
- ordered fixes;
- verification commands;
- release/deploy guidance;
- residual risk.

Prior transcript:
<transcript>

Frozen evidence:
<evidence>
```

### Adversarial Critique

```text
You are the adversarial reviewer in this panel.

Topic: <topic>

Critique this initial plan before implementation or acceptance:

<plan>

Prior transcript:
<transcript>

Find hidden assumptions, missing tests, security concerns, data-loss risk, workflow risk, release risk, and reasons this plan should not proceed.
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
You are <speaker> in a two-expert code review panel.

Topic: <topic>
Discussion round: <round>

Continue the technical review discussion. Challenge weak claims, resolve disagreements, and move toward an implementable consensus plan. Do not implement code in this discussion turn.

Prior transcript:
<transcript>

Frozen evidence:
<evidence>

Return a concise but technical response. State whether you converge, dissent, or need more evidence.
End with exactly one final line: CONVERGENCE: YES or CONVERGENCE: NO.
```

Convergence requires the latest Expert A discussion turn and latest Expert B discussion turn to both end with `CONVERGENCE: YES`.

### Final Consensus

```text
Synthesize the panel transcript into the final consensus.

Topic: <topic>

Include:
- P0/P1/P2/P3 findings;
- exact evidence;
- rejected or unsupported findings;
- agreed fixes;
- verification commands;
- implementation risk;
- release/deploy guidance;
- dissent notes;
- residual risk.

Prior transcript:
<transcript>

Frozen evidence:
<evidence>
```

### Final Gate Review

```text
You are the final gate reviewer. Use skeptical reasoning.

Topic: <topic>

Consensus plan:
<consensus>

Prior transcript:
<transcript>

Return exactly one status: GO, NOGO, or NO CONSENSUS.

If NOGO, list blockers.
If NO CONSENSUS, list unresolved disagreements and what evidence is needed.
If GO, list required verification and residual risks.
```

## Optional Implementation

Only run implementation if the user explicitly requested fixes to be applied.

Use a fresh implementation context:

```text
You are a fresh implementation agent.

Repository: <repo>
Base/ref under review: <ref>
Topic: <topic>

Consensus plan:
<consensus>

Implement only the agreed fixes from the final consensus.

Rules:
- Do not touch unrelated files.
- Preserve existing user changes.
- Follow the repository's branch, commit, and PR conventions when they are discoverable.
- Never commit directly to protected branches unless the user explicitly instructed it and the repository allows it.
- Run relevant tests.
- Do not deploy production.
- Report changed files, commits or branches created, test results, and anything not implemented.
```

After implementation, capture:

```bash
git -C "$REPO" status --short
git -C "$REPO" diff --stat
git -C "$REPO" diff
```

## Verification

Detect likely verification commands from repository conventions. Examples:

- JavaScript/TypeScript: package manager test, lint, typecheck, build scripts.
- Python: test runner, type checker, linter if configured.
- Rust: test suite.
- Go: package tests.
- Swift: package tests.
- Other ecosystems: use the project's documented or discoverable verification commands.

Run the smallest relevant set first. Broaden when the changed surface is shared, security-sensitive, or release-critical.

Record command, exit code, duration, and short stdout/stderr tails. If no verification command can be detected, say so explicitly in the recap.

## Recap Format

`recap.md` should use this structure:

```markdown
# Code Review Panel Recap: <topic>

Status: GO/NOGO/NO CONSENSUS
Repository: <repo>
Ref: <resolved commit or review target>
Artifacts: <artifact directory>

## Executive Summary
<short result>

## Findings
- P0/P1/P2/P3: <finding, evidence, impact, fix>

## Consensus
<agreed plan or no-consensus explanation>

## Dissent
<dissent, rejected findings, or "None">

## Implementation
<skipped or files changed>

## Verification
- `<command>`: PASS/FAIL/NOT RUN

## Residual Risk
<remaining risk>

## Next Actions
<ordered next steps>
```

## Final User Response

The final answer to the user should be concise and lead with the result:

- status: `GO`, `NOGO`, or `NO CONSENSUS`;
- top findings by severity;
- whether implementation ran;
- verification outcome;
- artifact paths;
- changed files, if any;
- residual risk and next action.

Do not paste the full transcript unless the user asks for it.

## Installation Recipe (implementation-agnostic)

An LLM installing this skill on a fresh system should:

1. Create a skill or prompt entry that its harness surfaces when the user asks
   for an expert code review. Use the equivalent skill, prompt, tool, or
   agent-config location for that harness.
2. Implement a runner that:
   - accepts the inputs above;
   - invokes two independent LLM execution routes via whatever local CLI or wrapper is available (no SDK imports, no direct API calls);
   - enforces the workflow, prompts, convergence rule, redaction, evidence freeze, and artifact layout above;
   - writes `raw.jsonl`, `discussion.md`, and `recap.md`, plus optional HTML renders.
3. Pick two genuinely different LLM routes by default and record which routes were used in every run's artifacts. Allow per-invocation overrides.
4. Document how to add more routes, how to scope focus and rounds, how to enable or disable implementation, and how to run a smoke test without spending real model calls.
5. Do not import any LLM provider SDK. Shell out to local CLIs only.

The runner may share code with the `expert-meeting` runner; in fact, doing so is encouraged so the two skills stay consistent.

## Companion Skills

- `expert-meeting.md` — same two-expert pattern for non-code topics. No repository, no evidence freeze, no implementation.
- `content-presentation.md` — how the rendered `discussion.html` and `recap.html` artifacts should be surfaced when the user asks to see a review.

## Smoke Test

The skill should ship a smoke-test mode that substitutes deterministic synthetic responses for each expert, runs the full workflow on a fixture repository, and writes artifacts. The smoke test must not require network access or real model calls, and must fail loudly if any step of the workflow (evidence freeze, independent reviews, synthesis, adversarial critique, discussion, consensus, gate review, optional implementation, verification, artifact writes) is skipped.
