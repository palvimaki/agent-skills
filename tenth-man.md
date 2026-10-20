# 10th Man Review Skill Specification

This document defines a reusable antagonistic review skill. It is
provider-agnostic, model-agnostic, harness-agnostic, machine-agnostic and
repository-agnostic.

The skill has three uses. It is one rung of the merge-gate review ladder for
pull requests. It is the standalone dissent for plans, architecture, research
and operations. It is the independent judge for routine process exceptions,
such as an extra repair cycle or a deadline extension.

> **Doctrine version: 2026-10-20.** The merge gate is now an ordered review
> ladder: one fresh independent reviewer per seat and round, a parseable verdict
> decides (on a production profile, after every peer seat finishes), and only
> P0/P1 findings block. Output with no verdict is a
> procedural failure, not a NOGO. The 10th man also adjudicates routine process
> exceptions. Re-check the ladder against your own routing config before you
> install.

## Contract

- One round, one fresh reviewer. The reviewer is never the author and never
  reads the author's conversation. The first output line is `VERDICT: GO` or
  `VERDICT: NOGO`.
- The reviewer comes from the repository's ordered `<review ladder>` in the
  machine-readable `<routing config>`. Never pick a reviewer from memory.
- On a single-review profile, the first parseable verdict decides. On a
  production profile, every peer seat that started must finish, and a P0/P1
  from any seat blocks. A later reviewer never overrides an actual P0/P1.
- Only P0 and P1 block. P2 and P3 become recorded, non-blocking follow-ups.
- Output with no parseable verdict, a timeout, a quota stop or a route error
  is a procedural failure. It is never a GO. Move to the next rung at once.
  Skip a rung that the availability record marks unavailable, without waiting.
- Every verdict binds to the exact frozen head SHA. A new head needs a new
  review. A review of an older head does not apply.
- Production apps and external repositories also get a separate security-axis
  review. An internal tooling repository may use one combined review when its
  repository profile says so.
- Sensitive material goes only to reviewers cleared for sensitive data.
- Each review stage stops at its own declared time limit. A timed-out partial
  is never a GO.
- The reviewer reads and reasons. It does not edit, merge or deploy. Its verdict
  is evidence for the delivery owner, not merge authority.

## Skill Identity

Name: `tenth-man`

Purpose: run a skeptical single-reviewer gate. The reviewer looks for the
strongest reason the proposed work should not proceed yet.

Use this skill when:

- a pull request needs its exact-head review round;
- the operator or an agent asks for a 10th man review, an antagonistic review,
  a dissenting view or a red-team pass;
- a plan, patch, architecture, research claim, incident fix, runbook or release
  plan needs a skeptical gate before commitment;
- a delivery owner needs a routine process exception: an extra repair cycle, an
  agent-set deadline extension or a bounded multi-project window.

Related specs:

- `double-up-code-review.md` adds an optional advisory reviewer on top of this
  gate for high-risk PRs.
- `expert-code-review-panel.md` is a deeper optional two-expert review. It does
  not replace this gate.

## The Review Ladder

The `<routing config>` declares one ordered ladder per repository profile. Each
rung names a route, its effort and its stage time limit. Typical profiles:

- **Internal tooling profile.** One review per round. The single review covers
  correctness and security together. The ladder lists a primary reviewer and
  ordered fallbacks.
- **Production profile.** Apps that real users touch, and external
  repositories. Each round has one frontier-tier correctness review and, when
  available, one secondary review from a different route. A secondary
  procedural failure leaves the frontier verdict in charge. An actual P0/P1
  from either reviewer blocks. A separate security-axis review always runs.
- **Other repositories.** Use the default ladder with the same blocking rule
  until the operator assigns a profile.

Round rules:

1. Freeze and redact the evidence. Record the head SHA.
2. Take the first rung that is available and cleared for this material. On a
   production profile, start every peer seat of the round (frontier, secondary,
   security) as a set. Peer seats are not fallback rungs.
3. Run each seat with a fresh context and the frozen evidence only.
4. Parse the verdict. On a single-review profile, a parseable `GO` or `NOGO`
   ends the round. On a production profile, wait for every peer seat that
   started. A `GO` from one seat never closes the round while a peer is still
   running. An actual P0/P1 from any seat blocks the head.
5. On a procedural failure, record it and go to the next rung of that seat's
   ladder at once.
6. If every rung fails, the round has `NO VERDICT`. The PR waits with a
   recorded resume condition, and the round runs again later. Reviewer
   availability is not an operator decision. A PR never merges without a
   verdict.

Independence rules:

- The reviewer is never the author, and never the session that wrote the
  change.
- Prefer a reviewer from a different model family than the author when the
  ladder offers one. A fresh context of the same family is the minimum.
- A fallback never routes to the author. It never escalates ad hoc to a
  frontier reserve that the ladder does not list.
- Frontier-tier reviewers run at high effort or above. Do not lower a frontier
  reviewer's effort to save quota. If cost matters, move the round to a cheaper
  rung that the ladder declares.

## The Security Axis

On a production or external repository, security is a separate review, not a
bullet in the correctness prompt. On a mixed checklist, correctness findings
crowd out security findings, because they are easier to prove and to write up.

- Scope: exploitability, trust boundaries, authentication and authorization
  bypasses, injection and deserialization, races with a security consequence,
  secret exposure and unsafe failure states.
- The security reviewer reads the same frozen evidence and never reads the
  correctness findings first.
- The two axes report in separate lists. A P0/P1 on either axis blocks.
- The security verdict binds to the same head SHA and keeps identity-bound
  evidence: which route ran, on which head, with which result.
- The security axis has its own ordered ladder. Its procedural failures follow
  the same round rules.

## Sensitive-Data Routing

Some material must not reach some routes: personal or patient data, secrets,
credentials and security internals. Decide by where the data goes, not by
vendor reputation.

- The `<routing config>` marks which reviewers are cleared for sensitive data.
- A sensitive diff skips every rung that is not cleared. The skip is recorded.
- If no cleared rung remains for a required axis, the round has `NO VERDICT`.
  It never passes with a "recorded limitation".

## Blocking Rule and Repair Rounds

- `VERDICT: NOGO` means at least one actual P0 or P1 finding.
- `VERDICT: GO` means no P0/P1 under the stated scope.
- P2/P3 findings never block. The owner records them as follow-ups, with an
  owner, in one PR comment or one issue.
- A repair round checks two things only: are the earlier blocking findings
  fixed, and does the repair delta add a P0 or P1. A new finding on code that
  the repair did not change is a non-blocking follow-up.
- The repair is a new head, so it gets a new round from the top of the ladder.
- The default repair allowance comes from the task. More repair cycles need a
  routine process exception (below).

## Stage Time Limits and Capacity

- Each rung has a positive stage time limit in the `<routing config>`. A rung
  without a valid limit is a route-resolution failure: no process starts, and
  the next rung runs.
- A round budget bounds the whole ladder.
- Set limits from measurement. Slow reasoning on a normal diff is not a
  failure. A limit shorter than the reviewer's normal reasoning time turns the
  ladder into a stream of procedural failures.
- A timed-out reviewer never yields a GO. If its partial output already shows a
  P0/P1 finding, the gate blocks on that finding. Any other timed-out output is
  a procedural failure.
- Declare a payload size limit. A frozen diff above it is blocked before
  dispatch, with the limit named in the record. A reviewer that gives no
  verdict on a large payload is a procedural failure, and the next rung runs.
- Preserve partial output when a route stalls.

## After the Verdict: Merge

The review is evidence. The orchestrator that owns the PR merges it
autonomously only when all of these hold for the current head SHA:

- a fresh `GO` from an independent reviewer on every required axis;
- passing required checks;
- no branch-protection rejection;
- for production and user-facing work, a UX sweep `PASS` whose build identity
  equals the head being merged (see `visual-review.md`); missing, stale, failed
  or incomplete sweep evidence blocks the merge;
- if the diff touches a gate spec or the review, merge or human-gate keys of the
  `<routing config>`, an operator decision record that names this head SHA; a
  missing record blocks the merge.

Rules:

- Merge with a fail-closed head guard, for example
  `gh pr merge <N> --match-head-commit <sha>`. Never use an admin bypass.
- Before the merge, read the target branch's deploy workflows. A merge that
  starts a production deploy or a store release is a publish step. It needs the
  operator's approval.
- A PR that weakens a review, merge or human gate needs its own operator
  decision record. Detect it by path and key, as in the list above, not by the
  author's description. A reviewer `GO` never replaces the record.
- If checks or review are still running, monitor the PR until it merges. Stop
  and record the blocker when a required check fails, a review returns NOGO,
  branch protection rejects the SHA, or the task deadline passes.

## Routine Process Exceptions

A delivery owner does not stop to ask the operator for routine process
exceptions. It asks a fresh independent 10th man instead.

Covered exceptions:

- one additional repair attempt beyond the default allowance;
- a bounded extension of an agent-set deadline, for example up to 60 minutes;
- a bounded multi-project review or execution window, limited to named
  projects and already authorized work;
- technical choices such as another repair attempt versus closing a PR.

Procedure:

1. Write the proposal in the existing `<work record>`: exact next action,
   original task authority, owner, current state, failed attempts, diagnosis,
   new evidence, expected progress, bounds and acceptance checks.
2. Give a fresh 10th man the proposal and the evidence only, never the author's
   implementation conversation. Keep sensitive input on cleared routes.
3. On `GO` for that exact proposal, record the reviewer route, evidence, bounds
   and expiry. Proceed within those bounds. Do not ask the operator to confirm.
4. On `NOGO`, address the finding or change the approach, then resubmit with
   new evidence. A different reviewer cannot erase the finding. Repeating the
   same failed action without new evidence does not qualify.
5. If every declared route is unavailable, keep the checkpoint and park the
   lane with a precise resume condition.

Limits:

- A grant permits one specific next step. It never declares the code
  acceptable. Exact-head review, security review, checks, UX evidence and
  deploy checks still apply.
- Keep the original task identity, deadlines, cumulative repair count and all
  failure evidence. Never reset a counter or rename a retry to hide history.
  A renewal is a new reviewed decision, not an edit of the old expiry.
- A grant never adds production, store release, payment, credential, legal,
  account or destructive-action authority. Explicit operator limits, pauses and
  hard deadlines stay binding.
- Records under a shared local account are audit evidence. They detect
  accidental change. They do not prove reviewer identity against a hostile
  process on the same account.

## Non-Negotiable Constraints

- Depend on no specific provider, model, account, machine, user name, path,
  repository, organization or host.
- Resolve route, model identifier, effort and stage limit from the
  `<routing config>`. An unknown or retired identifier fails fast as a
  procedural failure. It never falls through to a silent default.
- Run agent review on subscription CLIs or harness-native subagents, not on
  metered per-token APIs.
- Review evidence, not impressions. Separate confirmed evidence from inference.
- Lead with blocking findings. A `NOGO` names the remediation needed for `GO`.
- Do not edit files. Never overwrite unrelated changes. Never deploy.
- Redact secrets and identifying local details before any prompt or durable
  write.

## Required Inputs

- `target`: what is reviewed. Required. A PR, branch, commit, diff, plan,
  architecture proposal, research brief, incident report, runbook, release plan
  or exception proposal.
- `head_sha`: the exact head for a code target. Required for a PR round.
- `profile`: the repository profile. Default: from the `<routing config>`.
- `axis`: `correctness`, `security` or `combined`. Default: from the profile.
- `scope`: what to consider and what to leave out. Default: infer from the
  request.
- `author_route`: the model, agent or session that produced the target. Used to
  keep the reviewer independent.
- `sensitivity`: `normal` or `sensitive`. Default: classify from the material.
- `context`: constraints, tests, citations and prior blocking findings for a
  repair round.
- `output_target`: stdout, a file, a PR comment or another review surface.

## Required Outputs

- `VERDICT: GO` or `VERDICT: NOGO` as the first line;
- reviewed target, head SHA, axis and scope;
- severity-ranked findings, each with evidence, impact and, for P0/P1, required
  remediation;
- non-blocking P2/P3 notes, ready to record as follow-ups;
- open questions and residual risk, also when the verdict is `GO`;
- a provenance line: ISO 8601 timestamp, reviewer route, model and effort
  (`effort unknown` when unknown).

The gate record also stores each procedural failure: rung, reason and elapsed
time.

## Severity Model

- `P0`: do not proceed. Security breach, data loss, production outage,
  irreversible migration, legal or compliance hazard, or severe safety risk.
- `P1`: must fix before proceeding. Real bug, real regression, missing required
  validation, unsupported critical assumption, broken workflow, or a test gap
  that hides a credible failure.
- `P2`: should fix soon. A real quality, reliability, evidence or UX issue that
  does not block.
- `P3`: nit, clarity or optional cleanup.

Missing evidence for a critical claim is at least P1 until resolved.

## Evidence Freeze

For code, capture at least:

```bash
git -C "<repo>" rev-parse "<head_ref>^{commit}"
git -C "<repo>" rev-parse "<base_ref>^{commit}"
git -C "<repo>" diff --stat "<base_ref>...<head_ref>"
git -C "<repo>" diff --name-status "<base_ref>...<head_ref>"
git -C "<repo>" diff --find-renames --find-copies "<base_ref>...<head_ref>"
```

For a plan, research brief, runbook, incident report or exception proposal,
freeze the exact text, referenced excerpts, explicit constraints, stated
assumptions, claimed verification and the decision being gated.

Give the reviewer the complete diff in one pass when it fits the payload limit.
Do not split a review into chunks by file group: chunks lose cross-file context
and create their own edge cases. Mark anything left out. Do not invent missing
facts.

## Redaction

Before any prompt or durable write, redact:

- private key blocks, tokens, API keys, cloud credentials, database URLs and
  URLs with embedded credentials;
- environment values whose names contain `key`, `token`, `secret`, `password`,
  `credential`, `cookie` or `session`;
- absolute local paths, user names, host names, private organization and
  project names and account IDs that the review does not need;
- email addresses, phone numbers and other personal identifiers.

Use `[REDACTED_SECRET]` and `[REDACTED_LOCAL]`. If redaction makes the evidence
ambiguous, say so in the review.

## Reviewer Prompt Template

```text
You are the 10th Man: an antagonistic reviewer.

Target: <target>
Head SHA: <head_sha or n/a>
Axis: <correctness | security | combined | exception>
Scope: <scope>
Prior blocking findings (repair round only): <list or none>

Find the strongest reasons this should not proceed yet.

Rules:
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- Use P0/P1/P2/P3. Only P0/P1 block. NOGO requires at least one P0/P1.
- If GO, state that there are no P0/P1 findings under the stated scope.
- In a repair round, check only that the prior blocking findings are fixed and
  that the repair delta adds no P0/P1. Mark other findings as follow-ups.
- For each finding, give evidence, impact and required remediation.
- Separate confirmed evidence from inference.
- Do not praise. Do not summarize before findings. Do not edit files.

Frozen evidence:
<redacted_frozen_evidence>
```

For an exception proposal, add: "GO authorizes only the exact next step and
bounds below. Judge whether the new evidence shows real progress or a changed
approach."

## Output Format

```markdown
VERDICT: GO | NOGO

## Findings

1. [P0/P1/P2/P3] <title>
   Evidence: <file:line, section, command output or quoted claim>
   Impact: <failure mode>
   Required remediation: <fix or proof needed>

## Follow-ups (non-blocking)

- [P2/P3] <title>: <one line>

## Open Questions

- <question or assumption>

## Residual Risk

- <remaining risk>

## Scope

- Reviewed: <target, head SHA, axis>
- Not reviewed: <exclusions>
- Provenance: <ISO 8601 timestamp>, <route>, <model>, <effort>
```

## Review Patterns

- **Code:** security, data loss and migrations, concurrency and state
  corruption, authorization and tenant boundaries, error handling at system
  edges, deploy and rollback hazards, missing tests for changed behavior,
  user-visible regressions.
- **Plan:** unstated assumptions, missing prerequisites, irreversible steps,
  weak sequencing, unclear ownership, weak verification, rollback gaps, hidden
  production impact, decisions disguised as implementation detail.
- **Research:** unsupported or stale claims, cherry-picked evidence, missing
  counterexamples, invalid comparisons, conclusions that the evidence does not
  support.
- **Incident or operations:** unproven root cause, a mitigation that does not
  stop the failure mode, no recurrence guard, observability gaps, blast radius,
  bypassed production or human gates.
- **Exception proposal:** no new evidence, the same failed action repeated,
  bounds that are too wide, an expiry past an explicit operator deadline,
  authority that the grant cannot give.

## Installation Recipe

1. Create a skill, command or agent named `tenth-man` in the harness's normal
   skill location. Store this document as its instructions, or convert the
   sections from "Skill Identity" to "Output Format".
2. Add the `<review ladder>` for each repository profile to the
   `<routing config>`: ordered rungs, effort, stage limit, sensitive-data
   clearance and a separate security ladder where the profile needs one.
3. Keep an availability record per route, so the gate skips an unavailable
   rung at once.
4. Implement evidence freeze, redaction, head-SHA binding, verdict parsing,
   procedural-failure handling, the round budget and the payload limit.
5. Wire the merge step to the head guard and the deploy-workflow check.
6. Add the exception path: proposal record, fresh review, bounded grant with
   expiry, and history that keeps every grant and every NOGO.

## Smoke Test

Use a temporary repository or a synthetic plan with no private data.

1. Review a change with an obvious P1. Confirm `VERDICT: NOGO` with evidence,
   impact and remediation.
2. Fix it. Confirm the repair round checks the old finding and the delta, and
   that a new P3 on unchanged code is a follow-up, not a block.
3. Return only P2/P3 findings. Confirm `VERDICT: GO` and recorded follow-ups.
4. Make the first rung return text with no verdict line. Confirm a procedural
   failure and an immediate move to the next rung, never a GO.
5. Mark the first rung unavailable. Confirm the gate skips it without waiting.
6. Make every rung fail. Confirm `NO VERDICT`, a waiting PR and a resume
   condition, with no operator question.
7. Push a new commit after a GO. Confirm the old GO no longer applies.
8. Time out a reviewer whose partial output has a P1. Confirm a block. Time out
   one with only P3 notes. Confirm a procedural failure.
9. On a production profile, plant an injection defect. Confirm the security
   axis reports it in its own list and blocks.
10. Mark a diff sensitive with no cleared security rung. Confirm `NO VERDICT`,
    not a pass.
11. Submit an exception proposal with no new evidence. Confirm `NOGO`. Submit a
    changed approach. Confirm a bounded GO with an expiry in the record.
12. Confirm no output contains absolute local paths, user names, host names,
    credentials or personal identifiers.

## What Not To Do

- Do not write a balanced pro and con essay. This is a gate.
- Do not treat missing output as GO, and do not count it as a NOGO finding.
- Do not wait on a reviewer that the availability record marks unavailable.
- Do not let a later reviewer erase an actual P0/P1.
- Do not block on P2/P3.
- Do not reuse a review across heads.
- Do not send sensitive material to an uncleared route.
- Do not merge with an admin bypass, and do not merge a publish step without the
  operator.
- Do not ask the operator for a routine process exception.
