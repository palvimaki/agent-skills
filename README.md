# agent-skills

Portable specifications for the skills behind a multi-agent coding setup: tiered
model routing, an independent review ladder, autonomous merge with true human
gates only, visual QA, operator communication, and session wrap-up — plus a few
skills for everyday agent work. Any capable agent can install each one on its own
harness.

Eighteen specs. Each file is self-contained. It opens with a dated **doctrine
version** line and a short **Contract** block: the normative core in about ten
lines. You can judge freshness at a glance and act on the rules without reading
the whole file. Model names, tier names and product versions are worked examples
with a shelf life. The roles and the rules are the portable part.

Current doctrine version: **2026-10-20**.

## The setup in one paragraph

An orchestrator session owns one outcome and one work record. It routes work by
cost tier: a fast, low-cost tier as harness-native subagents for search,
summaries and mechanical edits; a daily-driver route for standard coding; a
demanding route; and a frontier reserve kept for the hardest reasoning. Agent work
runs on subscriptions, and one machine-readable routing config is the source of
truth. Every PR gets one fresh, independent reviewer per round from an ordered
ladder, bound to the exact head SHA; only P0/P1 findings block, and production
code also gets a separate security axis. The orchestrator then merges its own PR
behind a fail-closed guard on both base and head SHAs. The operator is asked only at true human gates — the
canonical list in `session-convergence.md` (production releases, the final
public submit, payments, credentials, legal and consent grants, account
changes, irreversible deletes, physical-world actions) — and every session ends with
nothing left for the operator to interpret.

## Routing

- **[mythos-reserve-routing.md](mythos-reserve-routing.md)** — A token-effectiveness routing protocol. It keeps the frontier model as a reserve for the hardest reasoning, routes all other work down four cost tiers (the cheapest as native subagents), and keeps an independent review ladder on every PR. Includes the frontier effort floor and the quota re-check rule.
- **[model-culture-implementation-routing.md](model-culture-implementation-routing.md)** — A model-agnostic recipe for a local routing guide that maps task phases to routes by working style and cost tier, with one routing config as the source of truth.
- **[ultra-prompt.md](ultra-prompt.md)** — Writes one compact, self-complete prompt for a bounded orchestration run (worked against Claude Code's Ultra-Code feature), with a routing block, a verified character count, and a keyword-safety convention that keeps the spec inert.

## Review and merge

- **[tenth-man.md](tenth-man.md)** — The standard merge gate. One fresh, independent reviewer per seat from an ordered review ladder (a production profile runs its peer seats as a set); the verdict binds to the exact head SHA; only P0/P1 block; production code also gets a separate security axis. Covers guarded autonomous merge, and also judges routine process exceptions such as an extra repair cycle.
- **[double-up-code-review.md](double-up-code-review.md)** — Adds one advisory reviewer from a further model family on top of the ladder for high-risk PRs. A confirmed advisory P0/P1 blocks; the owner cannot dismiss one alone, so a fresh non-author reviewer must agree before it becomes a follow-up.
- **[expert-code-review-panel.md](expert-code-review-panel.md)** — An optional two-expert review with a dissent seat and a separate security seat that argue to consensus. Deeper evidence beside the review ladder, not a replacement for it.

## Delivery

- **[production-pipeline.md](production-pipeline.md)** — The umbrella workflow: Spec Front, then Two Loops, then Ship. A human interview and research fan-out up front, an inner agentic loop (implement, review ladder, visual sweep, batched fix spec) inside an outer human loop on feel only, then guarded autonomous merge. The operator is touched only for the interview, the feel pass and true human gates.
- **[visual-review.md](visual-review.md)** — An autonomous UX gate. It walks the exact build in an isolated computer-use session, screenshots every meaningful state, triages findings into one fix spec, and iterates until an explicit pass. Required for every production patch, including backend-only work.
- **[operator-browser-session.md](operator-browser-session.md)** — Lets a computer-use agent drive the operator's own logged-in browser for console work in a bounded run: an exact task file, GUI locks, page text as untrusted data, reversible steps only, and a stop at credentials, payments, consent and the final public submit.

## Working with the operator

- **[operator-blocker.md](operator-blocker.md)** — Presents only true human gates to the operator: one evidence-backed decision per message, recommendation first, after the agent has tried every agent route itself.
- **[session-convergence.md](session-convergence.md)** — Closes or hands off substantial work with verified state. Every loose end is fixed, given to a named owner, or raised as one explicit decision at a true human gate. Each reply ends with one sentence that says what happens next, or that the session can be archived.
- **[expert-meeting.md](expert-meeting.md)** — Runs a two-expert panel on a non-code decision, with seats chosen by route tier, and ends with one actionable recommendation, preserved dissent and durable artifacts.
- **[content-presentation.md](content-presentation.md)** — Routes "show me" requests to the editor or to a rendered page (reviewable page, harness artifact, or browser), with a provenance label and a durable copy for deliverables.

## Writing and publishing skills

- **[writing-for-agents.md](writing-for-agents.md)** — How to write skills, instruction files, worker briefs and work records that agents follow reliably, plus the ASD-STE100 style rule for text written for the operator.
- **[skill-publish.md](skill-publish.md)** — Publishes a local skill as a redacted, role-based, harness-agnostic public spec. Work goes through an isolated branch and a PR, and every commit is scanned for identifying info.

## Housekeeping

- **[preserve-artifacts.md](preserve-artifacts.md)** — Makes generated deliverables durable before an agent reports them: version control or an artifact vault, a provenance label and hash manifest, heavy evidence on a work volume, and no silent fallback when storage is missing.
- **[file-self-destruct.md](file-self-destruct.md)** — Arms a timed, one-path, self-removing wipe on every transient plaintext secret file. Secrets to keep go into a managed store first, and then the original is shredded.

## Just for fun

- **[home-digital-twin.md](home-digital-twin.md)** — Builds a faithful, walkable 3D model of your own home from phone scans (LiDAR + RoomPlan) and photos, and lets you try real store furniture in place at true size, with a key that cycles products and a price card on screen. Built on a vector floor plan, so walls and ceilings come out straight.

## What "portable" means here

These are specifications, not implementations. They:

- Describe routes by role — frontier reserve, daily driver, fast worker tier,
  correctness reviewer, security reviewer, computer-use executor — not by vendor.
  Brand names appear only in sections marked **Example Only**, and the installer
  must re-research and replace them.
- Do not assume one harness. Native subagents, hooks, skills folders, worktrees
  and hosted artifacts are described as generic capabilities, with products only
  as examples.
- Do not assume a machine, operating system or path layout. Output directories
  and open commands are defaults.
- Require only local shell access and whatever CLI or subagent mechanism the host
  uses to call a model.
- Allow long timeouts for expert and review work, with stage time limits that
  record a timeout as a procedural failure, never as a pass.

Three specs name an external project or product on purpose:

- `ultra-prompt` is about one product feature — Claude Code's Ultra-Code runs —
  and its mechanics section tells the installer to re-verify version details. Its
  keyword-safety convention (the hot keyword written only as `ultra*code` in a
  code span) is deliberate, not a typo.
- `production-pipeline` and `writing-for-agents` credit
  [Matt Pocock's skills repository](https://github.com/mattpocock/skills) as
  upstream for their front-half stages and writing concepts.

## How to use

### 1. Ask a capable agent to install the skill

Hand the spec to an agent with file-writing access:

> Install the skill described in `tenth-man.md` on this machine. Fill the route
> table from my routing config. Place the skill where this harness keeps skills,
> and run the spec's smoke test.

The spec is written to be enough on its own: it tells the agent what to research
locally, which placeholders to fill, and how to verify the install.

### 2. Read the spec and implement it yourself

Each file is structured for a human reader: numbered workflow, verbatim prompt
templates, explicit artifact layout, failure modes and a smoke test. Most skills
are a few hundred lines of shell or Python around local CLIs. Some are only rules
and need no runtime.

## Safety defaults

All specs fail safe: no production deploys, no public submissions, no payments, no
credential entry and no external messages without the operator. Reversible,
internal steps — test builds, draft saves, eligible merges — are the agents' job.
The one intentional destructive skill is `file-self-destruct`, a scoped
single-path deletion of a secret file you asked to expire.

## Keeping these current

Specs rot quietly: the local skill keeps moving and the published copy does not.
`skill-publish` carries the refresh procedure. Run a pass when a mirrored local
skill changes materially, when a model family in an example ships a major version
or retires a tier, when a doctrine changes, or when a pinned product version is
long past. Prefer turning a stale brand name into a role name over updating it to
the newest brand — the role will still be right next quarter.

## Contributing

Pull requests welcome. Keep specifications implementation-agnostic. If a prompt
template, workflow step or artifact field changes, update the affected spec and
say why.

## License

MIT. See [LICENSE](LICENSE).
