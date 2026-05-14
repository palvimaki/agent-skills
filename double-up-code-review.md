# Double-Up Code Review Skill Specification

This document defines a reusable pull request review skill. It is intentionally
machine-agnostic, account-agnostic, repository-agnostic, and implementation-
agnostic.

The skill can be implemented by any capable LLM agent that can inspect a git
repository, call a normal antagonistic "10th Man" review route, call a second
independent reviewer route in read-only mode, and write or post a combined
review.

## Skill Identity

Name: `double-up-code-review`

Purpose: run a PR review gate that combines the normal 10th Man antagonist with
an independent second antagonist. Either reviewer can block the merge. Fixes
are re-reviewed by the full Double-Up Code Review gate, not by standalone
10th Man.

Use this skill when the user asks for:

- Double-Up Code Review;
- a double-up review;
- a second-reviewer PR review gate;
- a stricter-than-10th-Man PR or merge gate;
- a re-review after PR fixes;
- a review that should include both the usual antagonist and a second
  independent reviewer.

Do not use this skill for standalone plan, architecture, or research dissent
unless the user explicitly wants the PR-style Double-Up gate. The normal
10th Man skill remains unchanged for those workflows.

## Non-Negotiable Constraints

- Do not depend on any local machine name, username, home directory, repository
  path, private organization name, internal host, or account-specific setup.
- Do not include absolute local paths in prompts, artifacts, comments, docs, or
  examples. Use placeholders such as `<repo>`, `<base_ref>`, and `<head_ref>`.
- Do not use direct LLM API calls unless the host system's existing review route
  already hides that detail behind a local wrapper. Prefer local CLIs.
- The first reviewer is the normal 10th Man antagonist. If the primary coder's
  model family is known, choose a 10th Man route from a different model family.
- The second reviewer is an independent antagonist route.
- The two reviewers should see the same frozen evidence. The second reviewer
  must not simply critique or summarize the 10th Man review.
- A P0 or P1 finding from either reviewer blocks merge.
- A missing verdict, failed reviewer invocation, unavailable reviewer route, or
  unredacted secret risk blocks merge.
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
- `second_reviewer_route`: independent review command or agent route. Default:
  the strongest available read-only route that is not the 10th Man route.
- `focus`: optional risk area. Default: correctness, security, reliability,
  tests, deploy risk, and user-visible regressions.
- `output_target`: where to write the combined review. Default: stdout; if a PR
  platform is available, also post a PR comment or review.

## Required Outputs

Always produce one combined review containing:

- top-level status: `APPROVED` or `BLOCKED`;
- exact reviewed base and head refs;
- review focus and evidence summary;
- 10th Man verdict and findings;
- second reviewer verdict and findings;
- combined blocking findings, de-duplicated when possible;
- required fixes for every P0/P1;
- non-blocking P2/P3 notes;
- explicit instruction that follow-up fixes require Double-Up Code Review
  re-review.

If integrated with a PR platform, post the combined review as one comment or
review. Request changes when blocked. Approve only when both reviewers are free
of P0/P1 findings.

## Workflow

1. Resolve `repo`, `base_ref`, `head_ref`, and optional `pr_id`.
2. Confirm the working tree does not contain unrelated unstaged changes that
   could pollute the review. If it does, stop and report the risk.
3. Freeze the review evidence.
4. Redact secrets and local identifiers from the frozen evidence.
5. Select the normal 10th Man route. If `primary_coder_family` is known, choose
   a different model family for this reviewer.
6. Run the 10th Man antagonist on the frozen evidence.
7. Run the second reviewer independently on the same frozen evidence.
8. Parse both verdicts. Missing or malformed verdicts are blocking.
9. Combine findings. Preserve disagreement instead of smoothing it away.
10. Mark the gate `BLOCKED` if either reviewer has P0/P1 findings or invocation
    failed. Otherwise mark it `APPROVED`.
11. Write or post the combined review.
12. On later fixes, rerun this same workflow from step 1.

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

### Second Reviewer Route

The second reviewer must run in read-only or plan mode when the route supports
it. The implementation must prevent this reviewer from editing files during
review.

Example command shape:

```bash
<second_reviewer_command> "<prompt>"
```

Use the equivalent headless read-only invocation for the chosen route. The
second reviewer should receive the same redacted frozen evidence as 10th Man,
not the 10th Man's conclusions.

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

- `VERDICT: NOGO` blocks.
- Any P0/P1 finding blocks.
- Missing verdict blocks.
- Reviewer invocation failure blocks.
- `VERDICT: GO` approves only if the body does not contain actual P0/P1
  findings.

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

### Second Reviewer Prompt

```text
You are the second independent reviewer in a Double-Up Code Review PR gate.

You are a second independent antagonist. Do not defer to another reviewer.

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

### Second Reviewer Result

VERDICT: <GO or NOGO>

<findings>

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
3. Configure one variable for the normal 10th Man route.
4. Configure one variable for the second independent reviewer route in read-only
   headless mode.
5. Implement evidence freeze, redaction, two independent reviewer calls, verdict
   parsing, and combined output exactly as described above.
6. Make PR creation and PR re-review wrappers call this skill instead of calling
   standalone 10th Man.
7. Keep standalone 10th Man commands unchanged for non-PR dissent.

## Smoke Test Contract

Use a temporary repository with no private data.

1. Create a branch with a small intentional P1 bug and a small harmless P3 nit.
2. Run the Double-Up Code Review gate against the branch.
3. Confirm that the combined review is `BLOCKED` and includes both reviewer
   sections.
4. Fix the P1 bug, leave or fix the P3 nit, and rerun the full Double-Up Code
   Review gate.
5. Confirm that approval requires both reviewers to return `VERDICT: GO`.
6. Confirm no output includes absolute local paths, local usernames, private
   hostnames, credentials, or personal identifiers.

## What Not To Do

- Do not let the second reviewer see the 10th Man review before producing its
  own verdict.
- Do not approve when one reviewer fails to run.
- Do not treat "no blocking issues except..." as approval.
- Do not silently fall back from Double-Up Code Review to standalone 10th Man.
- Do not include private paths or identifying details in an open-source skill
  file, examples, prompts, comments, or screenshots.
- Do not merge on P0/P1 findings just because the other reviewer approved.
