# Double-Up Code Review Skill Specification

This document defines an optional extra review for high-risk pull requests. It
is provider-agnostic, model-agnostic, harness-agnostic, machine-agnostic and
repository-agnostic.

Double-Up runs the standard merge gate unchanged, then adds one more
independent reviewer as an **advisory** voice. The extra reviewer widens
coverage. It never approves a merge, and it never blocks a merge on its own
authority.

> **Doctrine version: 2026-10-20.** The standard gate is now the ordered review
> ladder in `tenth-man.md`: one fresh reviewer per seat and round, parseable verdict
> decides, only P0/P1 block. Double-Up is an advisory extra on top of that gate.
> A same-session review by a different model family counts as advisory only.

## Contract

- The merge gate is the repository's `<review ladder>`, as defined in
  `tenth-man.md`. Double-Up does not change it, replace it or add a rung to it.
- Double-Up adds one advisory reviewer from a route that the ladder round did
  not use, preferably from a further model family.
- The advisory reviewer reads the same frozen, redacted evidence and the same
  head SHA. It never reads the gate reviewer's output first.
- Advisory findings never approve and never block by themselves. The delivery
  owner triages each advisory P0/P1 against the code.
- A confirmed advisory P0/P1 is an actual defect. The owner fixes it before the
  merge. The fix is a new head, so it needs a new ladder round.
- An unconfirmed advisory finding becomes a recorded follow-up with the reason.
  Silent dismissal is not triage.
- An advisory failure, timeout or missing verdict is a recorded limitation. It
  never holds the merge past the advisory stage limit.
- Sensitive material goes only to an advisory route cleared for sensitive data.
  If none is cleared, skip the advisory review and record it.
- Read-only. No edits, merges or deploys from this skill.
- Resolve every route, effort and stage limit from the `<routing config>`.

## Skill Identity

Name: `double-up-code-review`

Purpose: get a further independent opinion on a PR whose failure would be
costly, without slowing or weakening the standard gate.

Use this skill when:

- the operator asks for a Double-Up review;
- the delivery owner judges a PR high risk, for example authentication and
  authorization, data migrations, payment code, concurrency, irreversible
  operations, or a change to the review, merge or human gates;
- the standard gate passed, but the owner has a specific doubt that a further
  model family could test.

Do not use it as the default PR gate. Do not use it for plan or research
dissent; use `tenth-man.md` for that. For a full structured discussion between
two experts, use `expert-code-review-panel.md`.

## How It Fits the Gate

| Part | Source | Blocks? |
| --- | --- | --- |
| Correctness or combined review | ladder round (`tenth-man.md`) | Yes, on P0/P1 |
| Security-axis review | security ladder, where the profile has one | Yes, on P0/P1 |
| Advisory review | this skill | No; owner triage decides |

Two kinds of advisory reviewer:

- **Routed advisory reviewer.** A route from the `<routing config>` that is
  marked advisory, run as a fresh read-only process.
- **In-session different-family review.** A harness-native subagent from a
  different model family than the author, run inside the owner's session. It is
  useful and cheap. It is advisory only, because the owner session controls its
  prompt and context. It never counts as the independent gate reviewer.

## Non-Negotiable Constraints

- Depend on no local machine name, user name, home directory, repository path,
  private organization, internal host or account setup. Use placeholders such
  as `<repo>`, `<base_ref>` and `<head_ref>`.
- Run reviews on subscription CLIs or harness-native subagents, not on metered
  per-token APIs.
- The advisory reviewer is never the author and never the gate reviewer of the
  same round.
- The advisory reviewer runs read-only or in plan mode. The implementation
  must stop it from editing files.
- A review binds to one head SHA. A new head makes every earlier review stale,
  advisory included.
- Frontier-tier advisory reviewers run at high effort or above. To save quota,
  pick a cheaper advisory route; do not lower a frontier reviewer's effort.
- Redact secrets and local identifiers before any prompt or durable write.
- Never discard unrelated changes. Never deploy.

## Required Inputs

- `repo`: the repository. Default: current repository.
- `base_ref` and `head_ref`: Default: the PR base and the current head.
- `head_sha`: the exact head the gate reviewed. Required.
- `pr_id`: optional PR number or URL.
- `author_route`: the model family or session that wrote the change.
- `gate_record`: the ladder round result for this head, if it already exists.
- `advisory_route`: Default: the first advisory route in the
  `<routing config>` that is not the author and not the gate reviewer.
- `sensitivity`: `normal` or `sensitive`.
- `focus`: optional risk area. Default: the reason this PR was marked high risk.
- `output_target`: stdout, a file or a PR comment.

## Required Outputs

One combined review:

- gate status: `APPROVED`, `BLOCKED` or `NO VERDICT`, taken from the ladder
  round, with any confirmed advisory P0/P1 applied;
- base, head and head SHA;
- the gate verdict and findings, per axis, each in its own list;
- the advisory verdict and findings, marked non-blocking;
- the triage line for each advisory P0/P1: confirmed (fix required) or not
  confirmed (follow-up, with reason);
- recorded P2/P3 follow-ups;
- limitations: skipped or failed advisory route, sensitive-data skips;
- provenance: ISO 8601 timestamp, routes, models and effort.

## Workflow

1. Resolve the repository, refs and head SHA. Stop if the working tree has
   unrelated changes that would pollute the evidence.
2. Freeze and redact the evidence once. All reviewers get the same bundle.
3. Run the standard ladder round for this head, or reuse its result if it
   already ran on this exact head.
4. Select the advisory route: not the author, not the gate reviewer, cleared
   for the material, available now.
5. Run the advisory review with its own stage time limit. It may run in
   parallel with the ladder round.
6. Parse the advisory verdict. A missing verdict is a recorded limitation.
7. Triage every advisory P0/P1 against the code. Confirm it with evidence, such
   as a failing test, a reproduced path or a cited line, or record why it is
   not real.
8. Set the gate status:
   - `BLOCKED` when the ladder returned NOGO on any axis, or when triage
     confirmed an advisory P0/P1;
   - `NO VERDICT` when the ladder round has no verdict;
   - `APPROVED` only when every required axis has a `GO` and no finding list
     on any axis contains a P0 or P1.
9. Write or post the combined review. Record P2/P3 follow-ups with an owner.
10. After a fix, start again at step 1 with the new head. Run Double-Up again
    only if the risk reason still applies.

Merge stays with the orchestrator under the rules in `tenth-man.md`: fresh GO
on every required axis, passing checks, no branch-protection rejection, a UX
sweep pass bound to the merged head for production and user-facing work, an
operator decision record for any gate-weakening diff, a fail-closed head guard
and a deploy-workflow check.

## Evidence Freeze

```bash
git -C "<repo>" rev-parse "<base_ref>^{commit}"
git -C "<repo>" rev-parse "<head_ref>^{commit}"
git -C "<repo>" log --oneline --max-count=20 "<base_ref>..<head_ref>"
git -C "<repo>" diff --stat "<base_ref>...<head_ref>"
git -C "<repo>" diff --name-status "<base_ref>...<head_ref>"
git -C "<repo>" diff --find-renames --find-copies "<base_ref>...<head_ref>"
```

The command shapes are illustrative. If a ref does not resolve, stop and
report. Give the complete diff in one pass when it fits the payload limit of
the `<routing config>`.

## Redaction

Apply the redaction list in `tenth-man.md` before any prompt or durable write.
Use `[REDACTED_SECRET]` and `[REDACTED_LOCAL]`. If redaction makes the evidence
ambiguous, say so in the combined review.

## Advisory Prompt Template

```text
You are an independent advisory reviewer for a high-risk pull request.
You have not seen any other review. Do not defer to another reviewer.

Head SHA: <head_sha>
Base: <base_ref>
Focus: <focus>

Rules:
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- Use P0/P1/P2/P3. NOGO requires at least one P0/P1.
- Find real defects, not style preferences. Look for what a first reviewer is
  likely to miss.
- For each finding, give file and line, evidence, impact and required fix.
- Separate confirmed evidence from inference.
- Do not edit files.

Frozen evidence:
<redacted_frozen_evidence>
```

## Combined Review Format

```markdown
## Double-Up Code Review

Status: APPROVED | BLOCKED | NO VERDICT
Head SHA: <sha>  Base: <base_ref>
Author route: <family or unknown>
Provenance: <ISO 8601 timestamp>, <routes>, <models>, <effort>

### Gate: correctness (or combined)
VERDICT: <GO | NOGO>  Route: <rung used>  Procedural failures: <list or none>
<findings>

### Gate: security (when the profile has the axis)
VERDICT: <GO | NOGO>  Route: <rung used>
<findings>

### Advisory (non-blocking)
VERDICT: <GO | NOGO | none>  Route: <route>
<findings, each P0/P1 marked CONFIRMED or NOT CONFIRMED with the reason>

### Required fixes
- <every gate P0/P1 and every confirmed advisory P0/P1>

### Follow-ups (non-blocking)
- <P2/P3 and unconfirmed advisory findings, with owner>

### Limitations
- <skipped or failed advisory route, sensitive-data skips>
```

## Installation Recipe

1. Create a skill or command named `double-up-code-review` in the harness's
   normal skill location.
2. Mark one or more advisory routes in the `<routing config>`, with effort,
   stage limit and sensitive-data clearance.
3. Reuse the ladder implementation from `tenth-man.md`. Do not copy it.
4. Implement the advisory call, verdict parsing, triage record and combined
   output.
5. Make Double-Up opt-in: an explicit flag on the PR wrapper, or the owner's
   high-risk judgment. Keep the standard PR path on the ladder alone.

## Smoke Test

Use a temporary repository with no private data.

1. Plant a P1 that the gate reviewer finds. Confirm `BLOCKED`.
2. Plant a P1 that only the advisory reviewer reports, and confirm it with a
   failing test. Confirm `BLOCKED`, and confirm the fix needs a new ladder
   round.
3. Let the advisory reviewer report a P1 that triage cannot confirm. Confirm
   `APPROVED`, with the finding recorded as a follow-up and a reason.
4. Make the advisory route fail or time out. Confirm the gate reaches its
   status without waiting past the advisory limit, and records the limitation.
5. Let the advisory reviewer return GO while the gate round has no verdict.
   Confirm `NO VERDICT`, never `APPROVED`.
6. Mark the diff sensitive with no cleared advisory route. Confirm the advisory
   review is skipped and recorded, and the gate still decides.
7. Push a new commit. Confirm every earlier review is stale.
8. Confirm no output contains absolute local paths, user names, host names,
   credentials or personal identifiers.

## What Not To Do

- Do not let an advisory GO stand in for a ladder verdict.
- Do not let an advisory failure hold the merge.
- Do not dismiss an advisory P0/P1 without a triage record.
- Do not count an in-session review as the independent gate review.
- Do not show one reviewer's output to another before both verdicts exist.
- Do not fold the security axis into the correctness prompt on a production
  profile.
- Do not include private paths or identifying details in prompts, comments or
  artifacts.
