# Expert Code Review Panel Skill Specification

This document defines a reusable two-expert code review panel. It is
provider-agnostic, model-agnostic, harness-agnostic, account-agnostic and
machine-agnostic.

The panel is a deeper, optional review. Two independent experts review the same
frozen evidence, a dissent expert attacks their plan, and the experts discuss
until they converge or the round budget ends. The result is evidence for the
delivery owner. It is not the merge gate.

> **Doctrine version: 2026-10-20.** The panel is now explicitly optional and
> sits beside the standard review ladder in `tenth-man.md`; it never replaces
> it. Only P0/P1 findings block; P2/P3 become recorded follow-ups. Frontier
> experts run at high effort or above. Re-check routes against your own routing
> config before you install.

## Contract

- The panel is optional. Run it on an explicit request or for a decision where
  a single reviewer is not enough. It never replaces the repository's
  `<review ladder>`.
- Freeze the evidence and record the head SHA and the author route before any
  expert runs.
- No expert is the author or the authoring session. Prefer two experts from
  two different model families, both different from the author's.
- Keep the first reviews independent. No expert sees another's first take.
- Only P0/P1 findings block. P2/P3 become recorded follow-ups with an owner.
- Keep correctness and security findings in separate lists. On a production or
  external repository, the profile's separate security axis still runs at the
  gate.
- A procedural failure (no verdict, timeout, route error) moves that seat to
  its next declared route. It is never a GO. The runner sets the status in
  this order: NOGO whenever any seat's findings or the consensus contain a
  P0/P1, including a P0/P1 that the consensus rejects (it stays blocking until
  the dismissal rule in Relation to the Merge Gate clears it), whatever the
  dissent text says; otherwise NO CONSENSUS when Expert A, Expert B, the
  dissent expert or the security expert (and the implementation expert when
  `implement` is true) exhausted its routes; otherwise GO or NOGO by the
  consensus rule.
- Sensitive material goes only to routes cleared for sensitive data.
- Read-only by default. Implement fixes only on request. Any fix is a new head
  and goes through the standard ladder before merge.
- Write every artifact to a durable location before the panel reports done.

## Skill Identity

Name: `code-review-panel`

Purpose: run a structured two-expert review that freezes repository evidence,
gathers independent reviews, applies adversarial critique, drives the experts
toward consensus, optionally implements agreed fixes, verifies the result and
writes a durable transcript and recap.

Use this skill when:

- the operator asks for an expert panel or a two-expert review;
- a high-risk change needs a deeper look than one review round gives, for
  example a large refactor, a security-sensitive redesign or a migration;
- the owner needs a structured disagreement before choosing a fix approach.

Do not use it as the default PR gate. Use `tenth-man.md` for the gate and
`double-up-code-review.md` for a single advisory extra on a high-risk PR.

## Relation to the Merge Gate

- The panel's final gate status is advisory input. The PR still needs a fresh
  ladder `GO` on every required axis for its current head SHA.
- A panel P0/P1 that the owner confirms is an actual defect. The owner fixes it
  before the merge. A panel P0/P1 that the owner does not confirm follows the
  dismissal rule in `double-up-code-review.md`: it blocks until a fresh
  non-author ladder reviewer rejects it.
- A panel never authorizes extra repair cycles or deadline extensions. Those
  go through the routine process exception path in `tenth-man.md`.
- The panel does not merge or deploy. Merge stays with the orchestrator under
  the head-guard and deploy-workflow rules in `tenth-man.md`.

## Companion Variant: Expert Meeting

`expert-meeting.md` runs the same two-expert pattern on a non-code topic. It
keeps the independent first takes, adversarial critique, alternating rounds,
convergence rule, consensus, dissent notes, transcript and recap. It replaces
the evidence freeze with a context freeze, and it drops implementation and code
verification unless the operator asks for an executed action plan.

## Non-Negotiable Constraints

- Depend on no specific provider, model, account, product, machine, path or
  subscription.
- Run expert seats on subscription CLIs or harness-native subagents, not on
  metered per-token APIs.
- Resolve every seat's route, model identifier, effort and stage time limit
  from the `<routing config>`. An unknown or retired identifier fails fast.
- Frontier-tier experts run at high effort or above. Do not lower a frontier
  seat's effort to save quota. If the panel is too costly, use cheaper declared
  routes or a smaller round budget.
- Two genuinely different models give the most value, because their training
  and failure modes differ. If only one family is available, use two fresh
  contexts with different roles and state the limitation in the recap.
- If the author route is unknown, record it as unknown. Use only routes that
  are provably not the authoring session, and state the limitation. Do not stop
  the panel to ask the operator.
- Exclude a route from sensitive material by where the data goes, not by vendor
  reputation. Record each skip. A seat with no cleared route counts as
  exhausted, and the Contract status order applies: NOGO on any P0/P1,
  otherwise `NO CONSENSUS`; it does not pass silently.
- Never discard or overwrite unrelated changes. Never deploy.
- Redact secrets before any prompt, response, transcript, recap or log.
- Separate confirmed evidence from inference.

## Stage Time Limits

Each seat has a stage time limit from the `<routing config>`, and the panel has
a total budget. Set limits from measurement: slow reasoning on a normal diff is
not a failure. A timed-out turn never counts as a GO or a `CONVERGENCE: YES`.
Preserve partial output and record the elapsed time.

## Required Inputs

- `topic`: the review question. Default: summarize the request.
- `repo`: the repository. Default: current working directory.
- `ref`: branch, commit or PR head. Default: `HEAD`. Record the resolved SHA.
- `base_ref`: Default: the PR base or the merge base.
- `focus`: optional subsystem or risk area. Default: correctness, reliability,
  tests and release risk.
- `author_route`: the model family, agent or session that wrote the change.
- `sensitivity`: `normal` or `sensitive`.
- `rounds`: discussion round budget. Default: 3, or up to 10 for a high-risk
  review.
- `implement`: apply agreed fixes. Default: false.
- `outdir`: artifact directory. Default: a dated directory in a durable
  documents location outside any repository that can be pushed, never only a
  temp directory. Commit panel artifacts only on an explicit operator request.
  Never commit them when sensitivity is sensitive or when the security seat
  reported an unfixed P0/P1.
- `expert_a`, `expert_b`, `dissent_expert`, `security_expert`,
  `implementation_expert`: seat routes. Default: from the `<routing config>`.

A route can be any shell-invokable CLI, harness-native subagent, local model
server or agent facility that takes a prompt and returns text.

## Required Outputs

Always write:

- `raw.jsonl`: append-only, machine-readable event log;
- `discussion.md`: the full readable transcript;
- `recap.md`: the concise recap.

Optionally write `discussion.html` and `recap.html`.

Artifacts state what was reviewed, the head SHA, what each expert said, where
they disagreed, whether they converged, what was implemented and verified,
which follow-ups were recorded and what risk remains. Label each artifact with
an ISO 8601 timestamp, the routes, the models and the effort (`effort unknown`
when unknown).

## Workflow

1. Resolve the repository, refs and head SHA. Record the author route.
2. Create the artifact directory.
3. Freeze and redact the evidence.
4. Run Expert A's independent review.
5. Run Expert B's independent review without Expert A's output.
6. Run the security expert's independent review on the same evidence, with the
   security scope only.
7. Ask one expert to synthesize an initial fix plan.
8. Ask the dissent expert to attack the plan.
9. Alternate Expert A and Expert B until both converge or the round budget
   ends.
10. Ask one expert for the final consensus.
11. Ask the dissent expert for the final status: `GO`, `NOGO` or
    `NO CONSENSUS`.
12. If implementation was requested, run a fresh implementation pass limited
    to the consensus.
13. Run relevant verification commands.
14. Record P2/P3 findings as follow-ups with an owner.
15. Write the artifacts and report.

## The Security Axis

The panel keeps a separate security seat, because correctness findings crowd
out security findings on a mixed checklist.

- Scope: exploitability, trust boundaries, authentication and authorization
  bypasses, injection and deserialization, races with a security consequence,
  secret exposure and unsafe failure states.
- It reads the same frozen evidence and never reads the panel transcript first.
- Its findings stay in their own list. A P0/P1 here sets the panel status to
  `NOGO`. The convergence rule does not apply to it: the security seat is not
  asked to agree with anyone.
- On a production or external repository, the gate's own security-axis review
  still runs on the final head. The panel's security seat does not replace it.

## Evidence Freeze

Record the author route and the head SHA first. Then capture at least:

```bash
git -C "<repo>" rev-parse "<ref>^{commit}"
git -C "<repo>" rev-parse "<base_ref>^{commit}"
git -C "<repo>" status --short
git -C "<repo>" log --oneline "<base_ref>..<ref>"
git -C "<repo>" diff --stat "<base_ref>...<ref>"
git -C "<repo>" diff --name-status "<base_ref>...<ref>"
git -C "<repo>" diff --find-renames --find-copies "<base_ref>...<ref>"
```

If `base_ref` or the head does not resolve, or `status --short` shows
unrelated changes, stop the panel and report it. Use a fallback only when the
fallback commit is named, and then the recap must say that the reviewed tree
is not the PR head. Never review uncommitted work under a head SHA.

Give each expert the complete diff when it fits the payload limit. Add focused
file contents: changed files, files in focus, files imported by changed files
and related tests. Leave out vendored code, build output, lockfile noise and
binaries unless they matter. Cap later transcript context to the most recent
relevant turns.

## Redaction

Before any durable write or prompt, apply the redaction list in `tenth-man.md`,
including personal identifiers and environment names that contain `cookie` or
`session`. Also redact secrets found in prompts, command output, diffs or model
responses.

Use `[REDACTED_SECRET]` or a similar obvious placeholder.

## Raw Event Log

`raw.jsonl` is newline-delimited JSON. Event kinds:

- `freeze`: repository, ref, head SHA, author route, focus, changed files,
  diff summary;
- `turn`: seat, step, round, prompt, response, timestamp, route, model, effort,
  status, duration;
- `procedural_failure`: seat, route, reason, elapsed time, next route;
- `verification`: command, exit code, duration, output tails;
- `implementation`: whether it ran, files changed, branch or commit;
- `summary`: final status, convergence, findings, follow-ups, residual risk,
  artifact paths.

Prompts and responses in `raw.jsonl` are redacted.

## Expert Roles

- **Expert A:** primary reviewer. Correctness, architecture, reliability and
  implementation planning.
- **Expert B:** independent reviewer. Finds what Expert A missed. Preferably a
  different model family from Expert A.
- **Security expert:** separate seat with the security scope only.
- **Dissent expert:** attacks weak assumptions, missing tests, workflow risk
  and premature consensus. Expert B in a fresh context is acceptable.
- **Implementation expert:** a fresh coding context, used only when fixes were
  requested.

These are roles, not provider names.

## Prompt Templates

### Independent Review (Expert A and Expert B)

```text
You are <Expert A | Expert B> in a two-expert code review panel.
<Expert B only: You have not seen Expert A's review.>

Topic: <topic>
Head SHA: <sha>
Focus: <focus>

Rules:
- Use only the frozen evidence unless you state that more data is needed.
- Classify issues as P0/P1/P2/P3. Only P0/P1 block; P2/P3 are follow-ups.
- For each issue, give evidence, impact and proposed fix.
- Separate confirmed evidence from inference.
- Expert B: prioritize what another reviewer might miss.
- Do not implement code.

Frozen evidence:
<evidence>
```

### Security Review

```text
You are the security reviewer in a code review panel. You have not seen any
other review.

Head SHA: <sha>
Scope: exploitability, trust boundaries, authn/authz bypass, injection,
deserialization, security-relevant races, secret exposure, unsafe failure
states. Nothing else.

Rules:
- Start with exactly one line: VERDICT: GO or VERDICT: NOGO.
- Classify issues as P0/P1/P2/P3. NOGO requires at least one P0/P1.
- For each issue, give file and line, attack path, impact and fix.
- Do not edit files.

Frozen evidence:
<evidence>
```

### Initial Plan Synthesis

```text
Synthesize the independent reviews into an initial fix plan.

Include: blocking findings (P0/P1), follow-ups (P2/P3), disagreements and weak
evidence, findings to reject as unsupported, ordered fixes, verification
commands and residual risk.

Prior transcript:
<transcript>

Frozen evidence:
<evidence>
```

### Adversarial Critique

```text
You are the dissent expert in this panel.

Attack this plan before anyone accepts it:
<plan>

Prior transcript:
<transcript>

Find hidden assumptions, missing tests, data-loss risk, workflow and release
risk, and reasons the plan should not proceed. Be specific. Keep dissent when
evidence is weak.
```

### Discussion Turn

Alternate Expert A and Expert B. Each turn ends with exactly one line:
`CONVERGENCE: YES` or `CONVERGENCE: NO`.

```text
You are <speaker> in a two-expert code review panel. Round: <round>.

Continue the review discussion. Challenge weak claims, resolve disagreements
and move toward an implementable consensus. Do not implement code.

Prior transcript:
<transcript>

Frozen evidence:
<evidence>

State whether you converge, dissent or need more evidence.
End with exactly one line: CONVERGENCE: YES or CONVERGENCE: NO.
```

Convergence needs the latest turn of each expert to end with
`CONVERGENCE: YES`. A missing line counts as `NO`.

### Final Consensus

```text
Synthesize the transcript into the final consensus.

Include: P0/P1 findings with exact evidence, P2/P3 follow-ups, rejected
findings, agreed fixes, verification commands, implementation risk, dissent
notes and residual risk.

Prior transcript:
<transcript>

Frozen evidence:
<evidence>
```

### Final Status

```text
You are the final dissent reviewer.

Consensus plan:
<consensus>

Security findings:
<security_findings>

Prior transcript:
<transcript>

Return exactly one status: GO, NOGO or NO CONSENSUS.
NOGO: list the P0/P1 blockers.
NO CONSENSUS: list unresolved disagreements and the evidence needed.
GO: list required verification and residual risks.
```

## Optional Implementation

Run only when fixes were requested.

```text
You are a fresh implementation agent.

Head SHA under review: <sha>
Consensus plan:
<consensus>

Implement only the agreed fixes.

Rules:
- Do not touch unrelated files. Preserve existing changes.
- Follow the repository's branch, commit and PR conventions.
- Never commit to a protected branch.
- Run relevant tests.
- Do not deploy.
- Report changed files, branch or commits, test results and anything not done.
```

The result is a new head. It goes through the standard ladder in
`tenth-man.md` before any merge. The default repair allowance applies; more
repair cycles need a routine process exception.

## Verification

Detect verification commands from repository conventions: the package
manager's test, lint, type-check and build scripts, or the documented commands
of the ecosystem. Run the smallest relevant set first. Broaden when the change
touches shared, security-sensitive or release-critical code. Record command,
exit code, duration and short output tails. If no command can be found, say so.

For a production patch, also attach current UX sweep evidence bound to the
build, also for backend-only work.

## Recap Format

```markdown
# Code Review Panel Recap: <topic>

Status: GO | NOGO | NO CONSENSUS (advisory; the ladder gate still applies)
Head SHA: <sha>
Provenance: <ISO 8601 timestamp>, <routes>, <models>, <effort>
Artifacts: <artifact directory>

## Result
<one or two sentences>

## Blocking Findings (P0/P1)
- Correctness: <finding, evidence, impact, fix>
- Security: <finding, evidence, impact, fix>

## Follow-ups (P2/P3)
- <finding> -> owner: <owner>, record: <work record>

## Consensus
<agreed plan, or why no consensus>

## Dissent
<dissent and rejected findings, or "None">

## Implementation
<skipped, or files changed and new head SHA>

## Verification
- `<command>`: PASS | FAIL | NOT RUN

## Limitations
<same-family seats, unknown author, sensitive-data skips, procedural failures>

## Residual Risk
<remaining risk>
```

## Final Response to the Operator

Lead with the outcome. Use short active sentences, one idea each:

- the status and the top blocking findings;
- whether implementation ran, and the new head SHA if it did;
- the verification result;
- the artifact location.

Do not paste the transcript unless asked. Close every loose end: fix it,
give it to a named owner with a resume condition, or state one decision that
needs the operator at a true human gate. End with one sentence that says what
happens next and when, or that the work is wrapped up.

## Installation Recipe

1. Create a skill or prompt entry that the harness surfaces when the operator
   asks for an expert code review panel.
2. Implement a runner that:
   - accepts the inputs above;
   - calls each seat through a local CLI or a harness-native subagent, with no
     provider SDK and no metered API;
   - enforces the workflow, prompts, convergence rule, stage limits, redaction,
     evidence freeze and artifact layout;
   - writes `raw.jsonl`, `discussion.md` and `recap.md`.
3. Declare the panel seats in the `<routing config>`, with effort, stage limit
   and sensitive-data clearance. Allow per-run overrides and record the routes
   used in every artifact.
4. Share code with the `expert-meeting` runner, so the two stay consistent.

## Companion Skills

- `tenth-man.md`: the standard merge gate and the routine exception path.
- `double-up-code-review.md`: one advisory extra reviewer for a high-risk PR.
- `expert-meeting.md`: the same two-expert pattern for non-code topics.
- `content-presentation.md`: how to show the rendered artifacts.

## Smoke Test

Ship a smoke-test mode that uses deterministic synthetic responses for each
seat, runs the full workflow on a fixture repository and writes the artifacts.
It needs no network and no real model calls. It must fail loudly if any step is
skipped: freeze, independent reviews, security review, synthesis, critique,
discussion, consensus, final status, optional implementation, verification,
follow-up records or artifact writes. Also check that:

- a P2-only result gives `GO` with recorded follow-ups;
- a security-seat P1, or a consensus P1, gives `NOGO` even when both experts
  converge and the synthetic dissent returns `GO`; an exhausted security seat
  gives NO CONSENSUS; an exhausted security seat together with a consensus P1
  gives NOGO;
- an Expert A P1 that the final consensus rejects gives `NOGO`;
- a seat that returns no verdict or times out moves to its next route and is
  never counted as `GO` or `CONVERGENCE: YES`;
- no artifact contains absolute local paths, user names, host names or
  credentials.
