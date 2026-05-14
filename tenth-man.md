# 10th Man Review Skill Specification

This document defines a reusable antagonistic review skill. It is intentionally
provider-agnostic, model-agnostic, account-agnostic, machine-agnostic, and
repository-agnostic.

The skill can be implemented by any capable LLM agent that can inspect the
review target, ask one independent reviewer to challenge it, and return a
clear `GO` or `NOGO` verdict with evidence.

## Skill Identity

Name: `tenth-man`

Purpose: run a skeptical single-reviewer gate for code, plans, architecture,
research, operations, or release decisions. The reviewer looks for the strongest
reason the proposed work should not proceed yet.

Use this skill when the user asks for:

- a 10th Man review;
- an antagonistic review;
- a dissenting view;
- a red-team pass;
- a skeptical gate before implementation, merge, release, deploy, publication,
  or strategic commitment;
- a review of a plan, patch, PR, research claim, architecture, incident fix, or
  operational change.

For PR workflows that explicitly require both normal 10th Man and a second
independent reviewer, use the separate `double-up-code-review` skill instead.
This skill remains the standalone antagonist.

## Non-Negotiable Constraints

- Do not depend on any specific LLM provider, model, account, API, SDK, machine,
  username, path, repository, organization, or host.
- Prefer a reviewer from a different model family or execution route than the
  primary author when available. If that is not available, use the strongest
  available independent context and state the limitation.
- Review evidence, not vibes. Separate confirmed evidence from inference.
- Lead with blocking findings before summary.
- A `NOGO` verdict must include concrete remediation required for `GO`.
- A `GO` verdict means no P0/P1 blockers remain under the stated scope.
- Do not edit files unless the user explicitly asks for implementation after
  the review.
- Never overwrite unrelated user changes.
- Never deploy to production from this skill.
- Redact secrets and identifying local details before durable writes.

## Timeout and Long-Running Work

Antagonistic review may take longer than ordinary shell checks, especially with
large evidence bundles, high-reasoning routes, remote queues, or constrained
local hardware. Implementations must expose configurable timeouts, choose
generous defaults for reviewer invocations, preserve partial output when a
route stalls, and avoid killing a healthy reviewer just because it exceeds a
short generic command timeout.

## Required Inputs

The skill should work with these inputs:

- `target`: what is being reviewed. Required. Examples: plan text, code diff,
  PR, branch, commit, file list, research brief, architecture proposal, incident
  report, runbook, release plan, or deploy plan.
- `scope`: what the reviewer should and should not consider. Default: infer
  from the user request.
- `risk_level`: optional. Values such as `low`, `normal`, `high`, or
  `critical`. Default: `normal`.
- `author_route`: optional description of the model, agent, team, or process
  that produced the target. Used only to select an independent reviewer route.
- `reviewer_route`: optional command, agent, model, or local wrapper to run the
  antagonist. Default: strongest independent review route available.
- `context`: optional background, constraints, files, commands, tests, or
  citations.
- `output_target`: stdout, a file, a PR comment, an issue comment, or another
  review surface. Default: stdout.

## Required Outputs

Always produce:

- `VERDICT: GO` or `VERDICT: NOGO` as the first line;
- reviewed target and scope;
- severity-ranked findings;
- evidence for each finding;
- impact and failure mode for each finding;
- required remediation for each P0/P1;
- non-blocking P2/P3 notes;
- open questions or assumptions;
- residual risk even when verdict is `GO`.

For code review, findings should include file and line references when possible.
For plans or research, findings should cite sections, claims, assumptions,
missing evidence, or decision points.

## Severity Model

Use this severity scale:

- `P0`: do not proceed. Security breach, data loss, production outage,
  irreversible migration, legal/compliance hazard, or severe safety risk.
- `P1`: must fix before proceeding. Real bug, real regression, missing required
  validation, unsupported critical assumption, broken workflow, or test gap that
  hides credible failure.
- `P2`: should fix soon. Meaningful quality, reliability, maintainability,
  evidence, or UX issue, but not a blocker.
- `P3`: nit, clarity improvement, polish, or optional cleanup.

Blocking rule:

- Any P0 or P1 means `VERDICT: NOGO`.
- P2/P3 only means `VERDICT: GO`, unless the volume of non-blocking issues
  creates a credible release or decision risk.
- Missing evidence for a critical claim is at least P1 until resolved.

## Workflow

1. Parse the target, scope, and risk level.
2. Freeze the evidence before review.
3. Redact secrets and local identifying information.
4. Select an independent reviewer route when available.
5. Ask the reviewer to attack the target within the stated scope.
6. Require the reviewer to produce `VERDICT: GO` or `VERDICT: NOGO`.
7. If the verdict is malformed, treat the review as `NOGO` and ask for a
   corrected verdict.
8. Return findings first, ordered by severity.
9. If the target is revised, rerun the review on the revised target instead of
   treating the previous review as approval.

## Evidence Freeze

For code or repository review, capture at least:

```bash
git -C "<repo>" rev-parse "<ref>"
git -C "<repo>" status --short
git -C "<repo>" diff --stat "<base_ref>...<head_ref>"
git -C "<repo>" diff --name-only "<base_ref>...<head_ref>"
git -C "<repo>" diff --find-renames --find-copies "<base_ref>...<head_ref>"
```

If reviewing a plan, architecture, research brief, runbook, or incident report,
freeze:

- the exact text under review;
- any referenced files, excerpts, or links;
- explicit user constraints;
- stated assumptions;
- claimed verification commands or evidence;
- current decision being gated.

Cap evidence so the review stays focused:

- diff or document excerpt: about 80,000 to 120,000 characters;
- individual file excerpt: about 12,000 characters;
- changed-file list or source list: complete when feasible;
- command output: enough to show pass/fail and relevant errors.

If evidence is incomplete, say so. Do not invent missing facts.

## Redaction

Before any durable write or model prompt, redact:

- private key blocks;
- environment variable values whose names include `key`, `token`, `secret`,
  `password`, `credential`, `cookie`, or `session`;
- common API keys, OAuth tokens, cloud credentials, SSH keys, database URLs, and
  bearer tokens;
- URLs containing embedded credentials;
- absolute local paths, especially home-directory paths;
- local usernames, hostnames, private organization names, project codenames, and
  account IDs not necessary for the review;
- email addresses, phone numbers, messaging identifiers, and other personal
  identifiers unless the user explicitly says they are public and relevant.

Use `[REDACTED_SECRET]` for secrets and `[REDACTED_LOCAL]` for local or
identifying details.

## Reviewer Prompt Template

```text
You are the 10th Man: an antagonistic reviewer.

Target: <target>
Scope: <scope>
Risk level: <risk_level>
Author route: <author_route or unknown>

Your job is to find reasons this should not proceed yet.

Rules:
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- Use P0/P1/P2/P3 severities.
- P0/P1 findings block and require NOGO.
- If GO, explicitly state that there are no P0/P1 findings under the stated
  scope.
- Lead with findings, ordered by severity.
- For each finding, include evidence, impact, and required remediation.
- Distinguish confirmed evidence from inference.
- Preserve uncertainty. If evidence is missing for a critical claim, treat that
  as a finding.
- Do not praise the work. Do not summarize before findings.
- Do not edit files.

Frozen evidence:
<redacted_frozen_evidence>
```

## Output Format

```markdown
VERDICT: GO | NOGO

## Findings

1. [P0/P1/P2/P3] <title>
   Evidence: <file:line, section, command output, or quoted claim>
   Impact: <failure mode>
   Required remediation: <specific fix or proof needed>

## Open Questions

- <question or assumption>

## Residual Risk

- <remaining risk after remediation or reason risk is acceptable>

## Scope Notes

- Reviewed: <what was included>
- Not reviewed: <known exclusions>
```

If there are no findings, write:

```markdown
VERDICT: GO

## Findings

No P0/P1 findings under the stated scope.

## Residual Risk

- <remaining uncertainty, test gap, or scope limit>
```

## Review Patterns

### Code Review

Focus on:

- security vulnerabilities;
- data loss or migration risk;
- concurrency, race conditions, and state corruption;
- auth, permissions, and tenant boundaries;
- error handling at system boundaries;
- deploy and rollback hazards;
- missing tests for changed behavior;
- user-visible regressions;
- compatibility and performance risks.

### Plan Review

Focus on:

- unstated assumptions;
- missing prerequisites;
- irreversible steps;
- weak sequencing;
- unclear ownership;
- insufficient verification;
- rollback gaps;
- hidden production or customer impact;
- decisions disguised as implementation details.

### Research Review

Focus on:

- unsupported claims;
- stale sources;
- cherry-picked evidence;
- missing counterexamples;
- invalid comparisons;
- overconfident conclusions;
- recommendations that do not follow from the evidence.

### Incident or Operations Review

Focus on:

- whether the root cause is proven;
- whether the mitigation actually stops the failure mode;
- whether recurrence is prevented;
- observability and verification gaps;
- rollback and blast-radius control;
- whether production approval or safety gates are being bypassed.

## Installation Recipe

An agent installing this skill should:

1. Create a skill or command named `tenth-man` in the host's normal skill
   location.
2. Store this document as the skill's primary instructions, or convert the
   sections from "Skill Identity" through "Output Format" into the host's
   required skill file.
3. Configure the reviewer route to the strongest independent model or agent
   available.
4. Add an optional parameter for `author_route` so the implementation can choose
   a different model family where possible.
5. Implement evidence freeze, redaction, reviewer invocation, verdict parsing,
   and fail-closed behavior.
6. Make revised targets rerun the full review instead of carrying forward a
   prior approval.

## Smoke Test Contract

Use a temporary repository or a synthetic plan with no private data.

1. Review a target with an obvious P1 issue.
2. Confirm the output starts with `VERDICT: NOGO`.
3. Confirm the P1 includes evidence, impact, and required remediation.
4. Fix the target or revise the plan.
5. Rerun the review.
6. Confirm `VERDICT: GO` appears only when no P0/P1 remains.
7. Confirm output contains no absolute local paths, local usernames, private
   hostnames, credentials, or personal identifiers.

## What Not To Do

- Do not make the review a balanced pro/con essay. It is a gate.
- Do not bury blockers under summary text.
- Do not approve with unresolved P0/P1 issues.
- Do not treat a prior review as approval for a revised target.
- Do not edit files while acting as reviewer.
- Do not include local paths, private names, account identifiers, or secrets in
  the public skill file or in durable review artifacts.
