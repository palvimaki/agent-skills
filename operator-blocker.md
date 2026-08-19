# Operator Blocker Skill Specification

This document defines a reusable skill for handing a human the decisions only
they can make. It is intentionally provider-agnostic, model-agnostic,
harness-agnostic, and machine-agnostic.

Any capable LLM agent that can write text can implement it. No LLM API, SDK, or
provider dependency is required — the whole skill is a presentation protocol.

> **Doctrine version: 2026-08-19.** This spec describes a communication
> protocol, not a toolchain, so it ages slowly. Adapt the wording to your
> operator's vocabulary; keep the sequencing rules exactly.

## Contract

- Announce the **count** first. One line. No detail, no preview, no options.
- Then stop and wait for a go.
- Present **one** blocker per message. Never stack. "And also…" is banned.
- Each blocker is a self-contained nugget: what it is, the decision, 2–3 plain
  options, **recommendation first**, marked as recommended, with one line of why.
- Never assume recall. Name things by what they *do*, never by an identifier.
- Always recommend. A bare "which do you want?" outsources the thinking.
- Do not manufacture blockers. Work you are already cleared to do, you do.
- This is sequencing, not withholding. The operator gets everything, one piece at
  a time.

## Skill Identity

Name: `operator-blocker`.

Purpose: convert a pile of human-gated decisions into a sequence of single,
answerable questions, so the human can decide each one from first principles
without reconstructing the project in their head.

Use this skill whenever you are about to hand the operator one or more
decisions, approvals, or judgments that only they can make:

- promoting anything to production;
- spending money, setting a price, or committing to a contract;
- entering credentials, or granting access;
- accepting a security or privacy trade-off;
- anything irreversible or destructive;
- a design fork where both branches are defensible;
- a legal, account, or identity decision;
- any moment you would otherwise end a message with "want me to X or Y?".

Triggers include reaching a gated decision mid-task, an end-of-task wrap-up that
lists gated items, and the operator saying "let's go through the blockers".

## The Problem This Solves

An operator running several agent sessions at once does not hold any one
codebase in their head. A wall of blockers written in agent vocabulary — PR
numbers, branch names, ticket identifiers, subsystem nicknames — is
unprocessable. They cannot remember which "PR #408" this is, or what "the
tileproxy refactor" was, and the cost of reconstructing that context is higher
than the decision itself. So the queue stalls, and the agent stalls with it.

The fix is not to write less. It is to write **one thing at a time**, in words
that carry their own context.

Define the unit precisely:

> An **operator blocker** is any decision, approval, credential, or judgment
> that only the human can make.

Everything else is your job.

## The Protocol

### Step 1 — Announce the count. Explain nothing.

One line: how many blockers are waiting, and whether they are ready. No detail,
no options, no jargon, no preview of what is coming.

```text
🚦 2 blockers are waiting on you. Ready to go through them one at a time?
```

Then **stop**. Wait for a go.

If the operator already said "walk me through them", skip to Step 2 — but still
give the count, because the count tells them how much they are agreeing to.

If they say "later", hold the queue, carry on with unblocked work, and do not
nag.

### Step 2 — Present ONE blocker. Then stop.

For the single blocker in front of them, write a short, self-contained nugget:

- **What this is** — one or two plain sentences. Assume they remember nothing.
  No bare identifiers: if you must reference one, say what it *does* in human
  words, not what it is called.
- **The decision** — the one thing they are choosing, stated plainly.
- **Options** — two or three plain-language choices, **recommendation first and
  marked as recommended**, with at most one sentence of why.

Nothing else. No second blocker. No appendix. Stop and wait.

### Step 3 — Take the answer, then the next one.

Act on the answer (or record the hold), then present the next blocker as its own
Step 2 nugget. Repeat until the queue is empty.

## Hard Rules

- **One blocker per message.** Stacking defeats the entire protocol.
- **Lead with the decision, not the backstory.** Backstory is one sentence, max.
- **Plain language.** If an intelligent non-programmer would not know a term,
  explain it in the same breath or cut it. No unexplained acronyms, no internal
  nicknames, no configuration syntax quoted at a human.
- **Never assume recall.** Name things by what they do.
- **Always recommend**, with one line of why. A decision is easier to make
  against a default, and the operator can always override it.
- **Count honestly.** One blocker still gets the announcement line.
- **Do not manufacture blockers.** If you are cleared to act — a safe patch,
  opening a pull request without merging, any read-only check — act, and say you
  did. A fake decision costs the operator the same attention as a real one.
- **Presentation, not withholding.** Everything reaches them, sequenced.

## Writing the Nugget

Two failure modes, both common.

**Too much context.** The nugget becomes a status report and the decision is
buried in paragraph three. The operator reads it as work, not as a question.

**Too little translation.** The nugget is short but written in agent vocabulary,
so it is short *and* unanswerable.

The self-check before sending:

> Could the operator answer this from first principles, right now, without
> opening the codebase and without remembering our last session?

If no, it is still jargon. Cut it down further.

### Example — the same blocker, badly and well

Badly:

```text
PR #412 on the tileproxy refactor is blocked on the CDN egress question —
we're at 4.2TB/mo on the current plan and the overage tier kicks in at 5TB.
Also #418 needs your call on the auth scope. Want me to proceed?
```

Well:

```text
🚦 2 blockers are waiting on you. Ready to go through them one at a time?
```

then, after the go:

```text
**What this is.** The map images our app serves to users are getting close to
the traffic limit on our current hosting plan. Above that limit we start paying
per gigabyte.

**The decision.** Move to the bigger plan now, or wait and pay overage if we
cross the line.

- **Move now (recommended)** — the bigger plan costs less per month than two
  months of overage at our current growth.
- **Wait** — cheaper if traffic flattens out, which it might after the seasonal
  peak.
```

## Required Inputs

- `blockers`: the list of gated items gathered during the work.
- `operator_state`: optional. Whether the operator has already agreed to go
  through the queue.

## Required Outputs

- One announcement line with an honest count.
- Then, on a go, one nugget per message, in the shape above.
- A record of each answer, so a later session can see what was decided and why.

## Ordering the Queue

Order by consequence to the operator, not by your convenience:

1. anything that is blocking other work right now;
2. anything time-sensitive, where waiting removes an option;
3. anything irreversible;
4. everything else.

If two blockers are genuinely one decision, merge them into one nugget and say
so. If a blocker turns out to be answerable from a rule the operator already
gave you, it is not a blocker — apply the rule and move on.

## Installation Recipe

An agent installing this skill should:

1. Create a skill, command, or standing instruction named `operator-blocker` in
   the host's normal location.
2. Set its trigger description to fire on gated decisions, end-of-task wrap-ups
   that list gated items, and any phrasing the operator uses for "walk me
   through the blockers".
3. Store the protocol, the hard rules, and the self-check as the instructions.
4. Adapt the vocabulary and the announcement line to the operator's own words.
   Keep the sequencing rules unchanged — they are the skill.
5. Wire it to whatever surface the operator actually reads, and record answers
   somewhere durable.

## Smoke Test

1. Give the agent a task that ends with three gated decisions.
2. Confirm the first message is a single line with the count and nothing else.
3. Confirm nothing further arrives until you answer.
4. Answer, and confirm exactly one nugget arrives.
5. Confirm the nugget names nothing by identifier, states one decision, and
   leads with a marked recommendation and one line of why.
6. Answer it, and confirm the second nugget arrives only then.
7. Add a fourth item the agent is already cleared to do. Confirm it is done
   rather than presented as a decision.
8. Say "later" mid-queue. Confirm the queue is held, unblocked work continues,
   and there is no follow-up nagging.

## Redaction

When installing from or publishing back to a private environment, strip
operator names, personal characteristics, organization and product names,
machine and host names, ticket identifiers, and internal tracker or messaging
integrations. Publish the protocol, not the person.
