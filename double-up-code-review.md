# Double-Up Code Review Skill Specification

This document defines a reusable pull request review skill. It is intentionally
machine-agnostic, account-agnostic, repository-agnostic, and implementation-
agnostic.

The skill can be implemented by any capable LLM agent that can inspect a git
repository, call the two blocking review axes, add a third advisory reviewer,
and write or post a combined review.

> **Doctrine version: 2026-08-19.** This file was re-scoped. It used to treat a
> second blocking antagonist as the optional stricter gate. A second blocking
> reviewer is now the standard gate — see "Two Blocking Axes" below and
> `tenth-man.md`. Double-Up is what you add *on top* of the standard gate.

## Contract

- The standard merge gate is two blocking axes: correctness and security. Each
  axis is its own reviewer, its own prompt, and its own verdict.
- Double-Up adds a third reviewer, from a further model family, as an
  **advisory** voice. Advisory findings never decide the merge by themselves.
- All reviewers read the same frozen, redacted evidence. No reviewer reads
  another reviewer's conclusions before producing its own verdict.
- A P0 or P1 on either blocking axis blocks. A missing, malformed, or failed
  blocking reviewer blocks.
- A route excluded by data residency or data policy records a review limitation.
  That skip is non-blocking unless it leaves a required axis unstaffed; then the
  gate blocks pending a human decision. Gate the surface, not the model.
- Fixes are re-reviewed by the same full gate. A prior approval never carries.
- Resolve every route from the host's routing config, never from memory.
- Read-only. No edits, no merges, no deploys from this skill.

## Skill Identity

Name: `double-up-code-review`

Purpose: run a PR review gate that adds a third, independent advisory reviewer
on top of the two blocking axes. Use it when one antagonist and one security
reviewer are not enough confidence for the change in front of you.

Use this skill when the user asks for:

- Double-Up Code Review;
- a double-up review;
- a third-reviewer PR review gate;
- a stricter-than-standard PR or merge gate;
- a re-review after PR fixes;
- a review that should add an extra independent family to the usual gate.

Do not use this skill for standalone plan, architecture, or research dissent
unless the user explicitly wants the PR-style Double-Up gate. The normal
10th Man skill remains unchanged for those workflows.

## Two Blocking Axes

Every PR that reaches this gate has already earned two blocking reviews. This
skill runs them and then adds the advisory third.

- **Correctness axis (blocking)** — the 10th Man antagonist. Prefer a model
  family different from the author's. Scope: real defects, regressions, missing
  validation, deploy risk, and test gaps that hide credible failure.
- **Security axis (blocking)** — a separate reviewer from a third family where
  one exists. Scope: exploitability, trust boundaries, authentication and
  authorization bypasses, injection and deserialization paths, races with a
  security consequence, secret exposure, and unsafe failure states. This axis is
  not a bullet inside the correctness prompt. It is a separate review, because a
  mixed checklist reliably loses the security items to the correctness items.
- **Advisory axis (non-blocking, this skill)** — a further independent family.
  It widens coverage and surfaces angles the first two share blind spots on. Its
  findings are triaged into the blocking axes when they are real; they never
  block on their own authority.

Fallback when the stack is smaller: with two independent routes, run correctness
and security and skip the advisory axis. With one independent route, run the two
axes as two separate invocations with fresh context each time, and state the
independence limitation in the combined review.

## Non-Negotiable Constraints

- Do not depend on any local machine name, username, home directory, repository
  path, private organization name, internal host, or account-specific setup.
- Do not include absolute local paths in prompts, artifacts, comments, docs, or
  examples. Use placeholders such as `<repo>`, `<base_ref>`, and `<head_ref>`.
- Do not use direct LLM API calls unless the host system's existing review route
  already hides that detail behind a local wrapper. Prefer local CLIs.
- The first reviewer is the normal 10th Man antagonist on the correctness axis.
  If the primary coder's model family is known, choose a route from a different
  model family.
- The second reviewer is the security axis, with its own scope and its own
  prompt. It is a required, blocking reviewer.
- The third reviewer is the advisory antagonist that gives this skill its name.
- Every reviewer sees the same frozen evidence. No reviewer may simply critique
  or summarize another reviewer's output.
- A P0 or P1 finding from either blocking axis blocks merge.
- A missing verdict, failed blocking-reviewer invocation, unavailable blocking
  route, or unredacted secret risk blocks merge. A failed blocking route never
  falls back to the author's own family, and never escalates to a reserve model
  that the host keeps for interactive work.
- A blocking route excluded from this material by data-residency or data-policy
  rules records a review limitation and does not block, **provided another
  eligible independent route staffs that axis.** If the exclusion leaves a
  required axis unstaffed, the gate blocks pending a human decision. Decide
  exclusions by where the data goes, not by vendor reputation.
- Resolve every route, model identifier, and effort setting from the host's
  machine-readable routing config where one exists. An unknown or retired
  identifier fails fast.
- Cap reasoning effort at the host's normal high tier unless the user states a
  need for a higher tier in this specific case.
- Once a PR enters this gate, fixes must be re-reviewed through the full
  Double-Up Code Review gate.
- Never discard unrelated user changes.
- Never deploy to production from this skill.
- Redact secrets and local identifiers before writing durable output.

## Timeout and Long-Running Work

Expert review routes can be slow. Large diffs, cold local models, constrained
hardware, remote queues, and high-reasoning settings may take much longer than
ordinary shell commands. Implementations must expose configurable timeouts,
choose generous defaults for expert processes, stream or record progress when
possible, and avoid declaring a reviewer failed solely because it exceeded a
short generic command timeout.

## Required Inputs

The skill should work with these inputs, using defaults when safe:

- `repo`: repository to review. Default: current repository.
- `base_ref`: branch or commit the PR targets. Default: the PR base branch, or
  the repository's configured integration branch.
- `head_ref`: branch or commit being reviewed. Default: current `HEAD`.
- `pr_id`: optional PR number, URL, or platform identifier.
- `primary_coder_family`: optional model family that authored the change, such
  as `model_family_a`, `model_family_b`, or `unknown`.
- `tenth_man_route`: the host's normal antagonistic review command or agent
  route.
- `security_route`: the required security-axis review command or agent route.
  Default: the strongest available read-only route that is not the 10th Man
  route and not the author's family.
- `second_reviewer_route`: the advisory review command or agent route. Default:
  the strongest available read-only route not already used by another axis.
- `focus`: optional risk area. Default: correctness, security, reliability,
  tests, deploy risk, and user-visible regressions.
- `output_target`: where to write the combined review. Default: stdout; if a PR
  platform is available, also post a PR comment or review.

## Required Outputs

Always produce one combined review containing:

- top-level status: `APPROVED` or `BLOCKED`;
- exact reviewed base and head refs;
- review focus and evidence summary;
- 10th Man (correctness axis) verdict and findings;
- security axis verdict and findings, kept in their own list;
- advisory reviewer verdict and findings, marked non-blocking;
- combined blocking findings, de-duplicated when possible;
- required fixes for every P0/P1;
- non-blocking P2/P3 notes;
- explicit instruction that follow-up fixes require Double-Up Code Review
  re-review.

If integrated with a PR platform, post the combined review as one comment or
review. Request changes when blocked. Approve only when both blocking axes ran,
both are free of P0/P1 findings, and triage promoted no advisory finding into
the blocking list.

## Workflow

1. Resolve `repo`, `base_ref`, `head_ref`, and optional `pr_id`.
2. Confirm the working tree does not contain unrelated unstaged changes that
   could pollute the review. If it does, stop and report the risk.
3. Freeze the review evidence.
4. Redact secrets and local identifiers from the frozen evidence.
5. Select the routes for all three axes from the host's routing config. If
   `primary_coder_family` is known, keep the correctness route in a different
   model family.
6. Run the 10th Man antagonist on the frozen evidence.
7. Run the security axis independently on the same frozen evidence.
8. Run the advisory reviewer independently on the same frozen evidence.
9. Parse every verdict. A missing or malformed verdict from a blocking axis is
   blocking; from the advisory axis it is a recorded limitation.
10. Combine findings, keeping each axis in its own list. Preserve disagreement
    instead of smoothing it away.
11. Mark the gate `BLOCKED` if either blocking axis has P0/P1 findings, if a
    blocking axis failed to run, or if triage promoted an advisory P0/P1 into
    the combined blocking list. Otherwise mark it `APPROVED`. A raw `GO` from
    both blocking axes is not sufficient after a promotion — promotion is the
    only thing that gives the third reviewer force, and an approval that ignores
    it turns the advisory axis into a label with no merge effect.
12. Write or post the combined review.
13. On later fixes, rerun this same workflow from step 1.

## Evidence Freeze

Capture at least:

```bash
git -C "<repo>" rev-parse "<base_ref>"
git -C "<repo>" rev-parse "<head_ref>"
git -C "<repo>" status --short
git -C "<repo>" log --decorate --oneline --max-count=20 "<base_ref>..<head_ref>"
git -C "<repo>" diff --stat "<base_ref>...<head_ref>"
git -C "<repo>" diff --name-status "<base_ref>...<head_ref>"
git -C "<repo>" diff --find-renames --find-copies "<base_ref>...<head_ref>"
```

The command shapes above are illustrative. Implementations should use the
host's safest equivalent and correct the revision syntax for their shell and
git version.

If the base or head cannot be resolved, stop and report a blocked review rather
than guessing. If the diff is too large, capture the complete file list and diff
stat, then include a capped diff plus enough changed-file excerpts to review the
highest-risk areas.

Recommended caps:

- diff: about 80,000 to 120,000 characters;
- individual file excerpt: about 12,000 characters;
- changed-file list: complete unless extremely large;
- command output tail: enough to show failures without dumping secrets.

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
  account IDs not necessary for the public review;
- email addresses, phone numbers, messaging identifiers, and other personal identifiers
  unless the user explicitly says they are public and relevant.

Use `[REDACTED_SECRET]` for secrets and `[REDACTED_LOCAL]` for local or
identifying details. If redaction would make the review evidence ambiguous, say
so in the combined review.

## Reviewer Selection

### 10th Man Route

The 10th Man route is whatever the host already uses for normal antagonistic
review. This skill wraps that route; it does not redefine the underlying
10th Man behavior.

Selection rule:

- If the primary coder used model family A, select a 10th Man route from model
  family B when available.
- If the primary coder used model family B, select a 10th Man route from model
  family A when available.
- If the primary coder is unknown, select the strongest available antagonist
  that is plausibly independent from the implementation route.
- If no differential route exists, run the strongest available 10th Man route
  and state the limitation in the combined review.

### Security Route

The security route is required, blocking, and independent. Prefer a family used
by neither the author nor the correctness reviewer. Give it the security prompt
below, not the correctness prompt — the axis is defined by its scope, and a
reviewer given a general prompt will drift back to correctness.

If the material is excluded from a route by data residency or data policy,
select another route **that is still independent of the author**, and prefer one
that is also not the correctness reviewer. Never select the author's family, and
never collapse the security axis back into the correctness reviewer's prompt —
on a sensitive diff those are exactly the shortcuts the exclusion existed to
prevent. If no independent eligible route remains, record the limitation and
report the gate as blocked pending a human decision.

### Advisory (Double-Up) Route

The advisory reviewer must run in read-only or plan mode when the route supports
it. The implementation must prevent this reviewer from editing files during
review.

Example command shape:

```bash
<second_reviewer_command> "<prompt>"
```

Use the equivalent headless read-only invocation for the chosen route. Every
reviewer receives the same redacted frozen evidence, never another reviewer's
conclusions.

## Verdict Rules

Each reviewer must start with exactly one of:

```text
VERDICT: GO
```

or:

```text
VERDICT: NOGO
```

`NOGO` means the reviewer found at least one P0/P1 issue or cannot safely
approve. `GO` means the reviewer found no P0/P1 issues.

Severity definitions:

- `P0`: security breach, data loss, production outage, irreversible corruption,
  or merge would be reckless.
- `P1`: must fix before merge. Real bug, real regression, broken workflow,
  missing required validation, or test gap hiding a credible failure.
- `P2`: should fix soon, but does not block this merge.
- `P3`: nit, polish, wording, or optional cleanup.

Parsing rule:

- `VERDICT: NOGO` on a blocking axis blocks.
- Any P0/P1 finding on a blocking axis blocks.
- A missing verdict on a blocking axis blocks.
- A blocking-reviewer invocation failure blocks.
- `VERDICT: GO` approves only if the body does not contain actual P0/P1
  findings.
- Advisory findings do not block on their own. Triage each one: promote it into
  a blocking axis when it is real, or record why it was not promoted. Silent
  dismissal is not triage.
- A **promoted** advisory finding blocks exactly like a native finding of the
  axis it was promoted into. Re-check the gate status after triage; do not read
  the blocking axes' raw verdicts and stop there.

## Prompt Templates

### 10th Man Prompt

```text
You are the normal 10th Man antagonist for this PR review.

Primary coder model family: <primary_coder_family>
Base ref: <base_ref>
Head ref: <head_ref>
Focus: <focus>

Rules:
- Review only the frozen evidence below unless you explicitly say more evidence
  is required.
- Find real defects, not style preferences.
- Prioritize security, data loss, broken user flows, deploy risk, correctness,
  missing validation, and test gaps that hide real risk.
- Classify findings as P0, P1, P2, or P3.
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- If GO, explicitly state that there are no P0/P1 findings.
- For each finding, include evidence, impact, and required fix.
- Include file and line references when available.
- Do not edit files.

Frozen evidence:
<redacted_frozen_evidence>
```

### Security Axis Prompt

```text
You are the security reviewer in a PR merge gate. You are not the correctness
reviewer. Another reviewer already covers correctness, regressions, and tests.

Base ref: <base_ref>
Head ref: <head_ref>

Scope, and nothing else:
- exploitability of the changed code by an untrusted input or an untrusted actor
- trust boundaries the change moves, widens, or removes
- authentication and authorization bypasses
- injection, deserialization, path traversal, and template evaluation paths
- races and time-of-check-to-time-of-use windows with a security consequence
- secret, token, and credential exposure, including in logs and error text
- unsafe failure states: what this code does when a check errors or times out

Rules:
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- Classify findings as P0, P1, P2, or P3. P0/P1 block the merge.
- For each finding, give the attacker's path: who calls it, with what input, and
  what they get. A finding with no reachable path is at most P2.
- Do not report style, naming, performance, or test-coverage issues. They belong
  to the other axis.
- Do not edit files.

Frozen evidence:
<redacted_frozen_evidence>
```

### Advisory Reviewer Prompt

```text
You are the advisory third reviewer in a Double-Up Code Review PR gate.

You are an independent antagonist. Do not defer to another reviewer.

Primary coder model family: <primary_coder_family>
Base ref: <base_ref>
Head ref: <head_ref>
Focus: <focus>

Rules:
- Review only the frozen evidence below unless you explicitly say more evidence
  is required.
- Find real defects, not style preferences.
- Prioritize issues another reviewer might miss.
- Classify findings as P0, P1, P2, or P3.
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- If GO, explicitly state that there are no P0/P1 findings.
- For each finding, include evidence, impact, and required fix.
- Include file and line references when available.
- Do not edit files.

Frozen evidence:
<redacted_frozen_evidence>
```

## Combined Review Format

```markdown
## Double-Up Code Review

Status: APPROVED | BLOCKED
Base: <base_ref>
Head: <head_ref>
Primary coder family: <primary_coder_family or unknown>

### Evidence

- Changed files: <count and summary>
- Diff scope: <summary>
- Redactions: <none or summary>

### 10th Man Result

VERDICT: <GO or NOGO>

<findings>

### Security Axis Result

VERDICT: <GO or NOGO>

<findings>

### Advisory Reviewer Result (non-blocking)

VERDICT: <GO or NOGO>

<findings, each marked promoted or not promoted, with the reason>

### Blocking Findings

- <P0/P1 finding, owner-neutral required fix>

### Non-Blocking Notes

- <P2/P3 finding>

### Re-Review Rule

Any fixes for this PR must be re-reviewed by the full Double-Up Code Review
gate.
```

## Installation Recipe

An agent installing this skill should:

1. Create a skill or command named `double-up-code-review` in the host's normal
   skill location.
2. Store this document as the skill's primary instructions, or convert the
   sections from "Skill Identity" through "Combined Review Format" into the
   host's required skill file.
3. Configure one variable for the correctness (10th Man) route.
4. Configure one variable for the required security route, with its own prompt.
5. Configure one variable for the advisory reviewer route in read-only headless
   mode.
6. Implement evidence freeze, redaction, three independent reviewer calls,
   verdict parsing, and combined output exactly as described above.
7. Make the two blocking axes the default for every PR wrapper, and make this
   skill the explicit opt-in that adds the advisory axis.
8. Keep standalone 10th Man commands unchanged for non-PR dissent.

## Smoke Test Contract

Use a temporary repository with no private data.

1. Create a branch with a small intentional P1 bug and a small harmless P3 nit.
2. Run the Double-Up Code Review gate against the branch.
3. Confirm that the combined review is `BLOCKED` and includes all three reviewer
   sections, each in its own list.
4. Fix the P1 bug, leave or fix the P3 nit, and rerun the full gate.
5. Confirm that approval requires both blocking axes to return `VERDICT: GO`.
6. Add a defect that is a security problem and not a correctness problem, such as
   an injection path or a secret written to a log. Confirm the security axis
   reports it and the gate blocks.
7. Make the security route fail on purpose. Confirm the gate blocks, and that it
   does not fall back to the author's family or to a reserve model.
8. Make the advisory route fail. Confirm the gate still reaches a verdict and
   records the limitation.
9. Mark the only eligible security route ineligible for the material on
   data-residency grounds. Confirm the combined status is `BLOCKED` pending a
   human decision — not `APPROVED` with a recorded limitation, and not a
   silent re-use of the author's family or the correctness reviewer's prompt.
10. Have triage promote an advisory P1 while both blocking axes returned `GO`.
    Confirm the gate is `BLOCKED`.
11. Confirm no output includes absolute local paths, local usernames, private
    hostnames, credentials, or personal identifiers.

Status vocabulary: a single reviewer returns `VERDICT: GO` or `VERDICT: NOGO`.
The combined gate reports `APPROVED` or `BLOCKED`. The two vocabularies describe
the same outcomes at different levels, and `tenth-man.md` uses the single-
reviewer pair for the same unstaffed-axis case — do not match on one token and
assume the other skill disagrees.

## What Not To Do

- Do not let any reviewer see another reviewer's output before producing its own
  verdict.
- Do not approve when a blocking reviewer fails to run.
- Do not fold the security axis into the correctness prompt as one more bullet.
  That is the failure this structure exists to prevent.
- Do not let an advisory `GO` stand in for a missing blocking axis.
- Do not treat "no blocking issues except..." as approval.
- Do not silently fall back from Double-Up Code Review to standalone 10th Man.
- Do not include private paths or identifying details in an open-source skill
  file, examples, prompts, comments, or screenshots.
- Do not merge on P0/P1 findings just because the other reviewer approved.
