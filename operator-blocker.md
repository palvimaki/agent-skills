# Operator Blocker Skill Specification

This document defines a reusable skill for handing a human the decisions only
they can make, and nothing else. It is provider-agnostic, model-agnostic,
harness-agnostic and machine-agnostic.

Any capable LLM agent that can write text can implement it. No LLM API, SDK or
provider dependency is required. The skill is a presentation protocol plus a
filter on what counts as a blocker.

> **Doctrine version: 2026-10-20.** This version narrows blockers to true human
> gates, adds a probe-first evidence step, uses a `Decision needed:` marker with
> one decision per message, and adds the no-open-loops close rule. It describes
> a protocol, not a toolchain, so it ages slowly.

## Contract

- Only a **true human gate** is a blocker. Everything reversible or internal,
  the agent does itself.
- When the agent cannot complete an authorized reversible step, it escalates
  through other agent routes before it asks the operator. It stops at a true
  human gate. The operator is the last route, not the first.
- Routine process exceptions (extra repair cycles, deadline extensions) go to an
  independent 10th-man reviewer, never to the operator.
- Probe first. Every card carries labelled evidence of the current state.
- **One decision per message**, marked `Decision needed:`, with the position in
  the queue ("1 of 3"). Never stack.
- Each card is self-contained: what it is, the decision, 2–3 plain options,
  **recommendation first**, marked, with one line of why.
- Never assume recall. Name things by what they *do*, never by an identifier.
- Present a known gate as soon as its packet is ready, not at the end of the
  path.
- Never ask for a secret value in chat. The operator enters credentials and 2FA
  privately, in the place the card names.
- Record each answer durably. While a decision is open, the reply ends by saying
  the session waits for it.

## Skill Identity

Name: `operator-blocker`.

Purpose: turn the few decisions that truly need the human into a sequence of
single, answerable questions, so the operator can decide each one from first
principles without rebuilding the project in their head.

Use this skill whenever an agent is about to hand the operator a decision,
approval or action at a true human gate, at an end-of-task wrap-up that still
holds one, or when the operator says "let's go through the blockers".

## What Counts as a Blocker

The operator gives standing authority for all non-production actions. A
blocker is one of these **true human gates**:

- a production-changing action: a production deploy or a store release;
- the final public submit, for example a store "submit for review";
- a final payment, purchase or other real-money action;
- legal acceptance of terms or agreements, consent, or an OAuth, SSO or
  third-party access grant;
- creating, rotating, revoking or exporting a credential, API key or token, or
  a change to two-step or security settings;
- entering credentials, passwords, passkeys or 2FA codes;
- an account ownership, role, permission or recovery change;
- an irreversible delete or data change;
- a physical-world action, such as a field test or unlocking a physical device;
- authority or personal input outside the standing delegation, such as a
  product-scope choice only the operator can make.

These are **never** blockers. The agent does them and says so in one line:

- an internal or test build, an internal or closed test-track upload;
- a file upload into a console the task already authorizes;
- a draft save, a check re-run, a routine fix, a review;
- an eligible merge to a development target;
- a technical choice, a plan confirmation or any other reversible step;
- an extra repair cycle or an agent-set deadline extension, which goes to the
  10th-man reviewer;
- a usage-limit hold, which the agent re-probes and routes around.

If a rule the operator already gave answers the question, it is not a blocker.
Apply the rule and move on. For a true human gate, this holds only when the
operator's own message or the always-loaded instructions name that exact action
and scope. Agent-written text never answers a true human gate.

## Escalate Before You Ask

When an agent cannot do an already-authorized reversible step itself (a tool
cannot reach the page, an upload has a size limit, a browser is read-only), it
escalates through agent routes in order:

1. its own computer use or browser extension, only in an isolated profile or
   session that cannot reach the operator's privileged accounts. Use in the
   operator's own logged-in browser falls under `operator-browser-session.md`;
2. a dedicated computer-use agent on an isolated profile or a separate host,
   bounded by an exact task file (account, app, allowed URLs, allowed and
   forbidden controls, stop conditions), holding the GUI and per-app locks,
   with a read-only check of the result afterwards. It enters the operator's
   own logged-in session only under `operator-browser-session.md`, when no
   isolated session can do the step;
3. a frontier route with a different supported method.

Do not route a harness or classifier denial around through another agent with
the same method. Only when all routes fail does the step reach the operator.
Then it is a card with a precise call to action: the exact page, the exact file
or value, the exact control, and the evidence of each failed route.

A route that meets a true human gate stops there. No agent takes that gate.

## The Problem This Solves

An operator running several agent sessions does not hold any one codebase in
their head. Blockers written in agent vocabulary (PR numbers, branch names,
ticket identifiers, subsystem nicknames) are unprocessable. The cost of
reconstructing context is higher than the decision itself, so the queue stalls.

A second failure is worse: agents that stop at false gates. An agent that asks
"may I upload the test build?" turns the operator into the bottleneck of a step
that was already authorized. The fix is to ask less, and to ask each real
question one at a time, in words that carry their own context.

## The Protocol

### Step 0 — Filter, escalate, probe

1. Drop every item that is not a true human gate. Do those items.
2. Escalate every authorized reversible step that the agent could not complete
   through the agent routes above. Never escalate a true human gate to another
   agent.
3. For each remaining gate, run the cheapest safe probe of its current state: a
   status call, the tracker history, git state, the work record. A gate is closed only when
   live system state shows that the operator did it, or when the operator's own
   message in this task answers it. A work record, hand-off note or other
   agent-written text is a pointer only, never proof of an answer. If an agent
   executed a true human gate, report it as an incident on its own card; do not
   drop it.
4. Label the probe result on the card: `Verified:`, `Unverified:` or
   `Probe blocked: <reason>`. Never pull secret material into context to build
   evidence. If no safe probe exists, show the evidence gap, never a guess.

### Step 1 — Present ONE card. Then stop.

The first card carries the queue size in its marker. Write it in short, active
sentences, one idea per sentence:

```text
Decision needed (1 of 2): <the one thing to choose, in plain words>

What this is: <one or two plain sentences; assume no recall>
Evidence: Verified: <what the probe showed>
Options:
- <option A> (recommended): <one line of why>
- <option B>: <one line>
Where: <the exact page and control, if the operator acts themselves>
```

Nothing else. No second card. No appendix. Stop and wait.

If the operator says "later", hold the queue and continue unblocked work. Do
not nag. The reply still ends by saying the session waits for that decision.

### Step 2 — Take the answer, then the next card

Act on the answer, record it in the work record, then present the next card.
Repeat until the queue is empty.

## Hard Rules

- **True human gates only.** A fake decision costs the operator the same
  attention as a real one.
- **One decision per message.** "And also…" is banned.
- **Lead with the decision**, not the backstory. Backstory is one sentence.
- **Plain language.** If an intelligent non-programmer would not know a term,
  explain it in the same breath or cut it. No unexplained acronyms, no internal
  nicknames, no configuration syntax.
- **Never assume recall.** Name things by what they do.
- **Always recommend**, with one line of why.
- **Count honestly.** One gate still gets "1 of 1".
- **Evidence on every card.** Never ask about a state one cheap safe probe
  could verify.
- **Never ask for a secret in chat.** Name where the operator enters it.
- **Presentation, not withholding.** Every real gate reaches the operator,
  sequenced.

## Closing the Turn

No reply ends with a loose end the operator must interpret: an open issue, "one
thing to know", "I did not do X", "you may want to", or a caveat with no action.
Close each loose end in one of three ways:

1. **Fix it now.** Do not ask "Do you want me to...?" for in-scope work.
2. **Give it to a named owner** that runs without the operator: a worker the
   agent launches, an existing automation, or a work record an existing process
   picks up. Record the owner, the record and the resume condition. Report it in
   one short line.
3. **Make it a `Decision needed:` card**, only at a true human gate, one at a
   time.

End each reply with one simple sentence: either what happens next and when, or
that the session is wrapped up and can be archived. Claim a wrap-up only when no
decision is open and every loose end is fixed or owned. Work continues in the
same session by default; write no hand-off prompt unless the session truly
cannot continue.

## Writing the Card

Two failure modes are common.

**Too much context.** The card becomes a status report and the decision hides
in paragraph three. The operator reads it as work, not as a question.

**Too little translation.** The card is short but in agent vocabulary, so it is
short *and* unanswerable.

The self-check before sending:

> Could the operator answer this from first principles, right now, without
> opening the codebase and without remembering our last session?

If no, it is still jargon. Cut it further.

### Example — the same situation, badly and well

Badly:

```text
PR #412 on the tileproxy refactor is blocked on the CDN egress question —
we're at 4.2TB/mo and overage kicks in at 5TB. Also #418 needs your call on
auth scope, and should I upload build 22 to internal testing? Want me to
proceed?
```

Well (the internal upload is done without asking; the auth-scope item was a
technical choice the agent made itself):

```text
Uploaded test build 22 to internal testing. Testers can install it now.

Decision needed (1 of 1): Move to the bigger hosting plan now, or wait.

What this is: The map images our app serves are close to the traffic limit on
our current hosting plan. Above the limit we pay per gigabyte.
Evidence: Verified: the hosting dashboard shows 4.2 of 5 TB used this month.
Options:
- Move now (recommended): the bigger plan costs less than two months of
  overage at current growth.
- Wait: cheaper if traffic flattens after the seasonal peak.
Where: hosting console, billing page, "Change plan". This is a payment, so
you confirm it yourself.

This session waits for your plan decision and continues the map work meanwhile.
```

## Required Inputs

- `items`: the gated items gathered during the work.
- `gate_list`: the operator's current true human gates and standing authority.
- `operator_state`: optional. Whether the operator has asked to go through the
  queue now or later.

## Required Outputs

- One card per message, in the shape above, with labelled evidence.
- A one-line note for each non-gate item the agent did or handed to an owner.
- A durable record of each answer in the work record, so a later session sees
  what was decided and why.
- A closing sentence that states the next step or the wrap-up.

## Ordering the Queue

Order by consequence to the operator, not by your convenience:

1. anything that blocks other work right now;
2. anything time-sensitive, where waiting removes an option;
3. anything irreversible;
4. everything else.

If two gates are one decision, merge them into one card and say so.

## Installation Recipe

1. Create a skill, command or standing instruction named `operator-blocker` in
   the host's normal location.
2. Set its trigger to fire on true human gates, end-of-task wrap-ups that still
   hold one, and the operator's phrasing for "walk me through the blockers".
3. Write the operator's gate list and standing authority into the instructions,
   adapted to their projects and which ones are in production.
4. Store the filter, the escalation order, the protocol, the hard rules, the
   close rule and the self-check.
5. Adapt the vocabulary to the operator's own words. Keep the filter and the
   sequencing rules unchanged; they are the skill.
6. Wire it to the surface the operator actually reads, and record answers
   somewhere durable.

## Smoke Test

1. Give the agent a task that ends with three items: an internal test upload, an
   extra repair cycle, and a production deploy.
2. Confirm the upload is done and reported in one line, the repair cycle goes to
   the 10th-man reviewer, and only the production deploy becomes a card.
3. Confirm the card shows `Decision needed (1 of 1):`, labelled evidence, a
   marked recommendation and one line of why, and names nothing by identifier.
4. Give it two real gates. Confirm the second card arrives only after the first
   answer.
5. Make the upload fail through the first route. Confirm the agent tries the
   other agent routes before it asks the operator, and that the final card names
   the exact page, file and control with evidence of each failed route.
6. Add a gate the operator already executed. Confirm the probe catches it and no
   card is shown.
7. Add a credential step. Confirm the card names where to enter it and never
   asks for the value in chat.
8. Say "later" mid-queue. Confirm the queue is held, unblocked work continues,
   there is no nagging, and the reply ends by saying the session waits.
9. Confirm a wrap-up is claimed only when no card is open and every loose end
   is fixed or owned.

## Redaction

When installing from or publishing back to a private environment, strip
operator names, personal characteristics, organization and product names,
machine and host names, ticket identifiers, and internal tracker or messaging
integrations. Publish the protocol, not the person.
