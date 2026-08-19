# agent-skills

Elevate your agentic coding (and publish your own skills!) with these powerful skills

Portable, implementation-agnostic specifications for thirteen LLM agent skills.

Every spec opens with a dated **doctrine version** line and a short **Contract**
block — the normative core in about ten lines — so you can judge freshness at a
glance and act on the rules without reading the whole file. Model names, tier
names, and product versions anywhere in this repository are worked examples with
a shelf life; the roles and the rules are the portable part.

Current doctrine version: **2026-08-19**.

- **[expert-code-review-panel.md](expert-code-review-panel.md)** — a two-expert code review panel with evidence freeze, adversarial critique, alternating discussion rounds, convergence rule, optional implementation, and verification.
- **[expert-meeting.md](expert-meeting.md)** — the same two-expert pattern for non-code topics: strategy, product, hiring, research, architecture. Context freeze instead of evidence freeze; no implementation phase.
- **[content-presentation.md](content-presentation.md)** — routes "show me / open / present" requests to the right surface: text editor for code and plain-text files, HTML-rendered-in-browser for data, digests, tables, and assembled reports. Includes a reusable dark-theme stylesheet.
- **[tenth-man.md](tenth-man.md)** — a standalone antagonistic review gate for code, plans, architecture, research, operations, and release decisions, with severity-ranked findings and a hard `GO` / `NOGO` verdict. Now also defines **the second axis**: a merge gate is correctness *and* security, run as two separate reviewers with separate scopes and separate verdicts, because a mixed checklist reliably loses the security findings to the correctness findings.
- **[double-up-code-review.md](double-up-code-review.md)** — re-scoped: the two blocking axes are the standard gate, and Double-Up is the **third, advisory** reviewer you add on top for high-risk PRs. Advisory findings are triaged into the blocking axes, never merged on their own authority. Includes the security-axis prompt template and the fail-closed rules.
- **[model-culture-implementation-routing.md](model-culture-implementation-routing.md)** — a model-agnostic recipe for creating a local routing skill that assigns exploration, design, execution, and review phases by current model temperament and observed strengths.
- **[skill-publish.md](skill-publish.md)** — a publication workflow for turning private/local skills into public-safe, model-agnostic skill specifications with redaction, examples, README updates, commit, and push.
- **[file-self-destruct.md](file-self-destruct.md)** — arms a deterministic, timed, surgical self-destruct on any plaintext secret file the moment it is created or imported, using the host's native one-shot scheduler (launchd / systemd / `at`) rather than legacy cron, with traceless teardown.
- **[mythos-reserve-routing.md](mythos-reserve-routing.md)** — a token-effectiveness routing protocol for getting the most out of the latest, most expensive frontier model across two frontier subscriptions: treat the "Mythos-class" model (a role, not a brand) as a strategic reserve for the reasoning only it can do, route all volume work to a capable workhorse ladder, keep two mandatory independent review axes on every PR, and decide inline-vs-delegate by token economics. Budget optimization follows automatically — but the goal is maximum benefit per frontier token, not frugality. Fully role-based: the workhorse is described as a three-rung ladder (orchestration / standard / mechanical) sharing one account-level rate window, so no rung rescues another's exhaustion. Includes an effort ceiling, a fail-closed reviewer rule, a data-residency rule, and a self-update clause for when the landscape shifts.
- **[ultra-prompt.md](ultra-prompt.md)** — writes a best-practice prompt for a bounded orchestration run — the worked mechanics are Claude Code's Ultra-Code feature (its `ultra*code` effort tier and the dynamic many-subagent runs it spawns) and pairs it with a tailored token-efficiency configuration (effort tier, scope, per-stage model, run hygiene). Product-specific by design — it is *about* one named feature — but runner-agnostic: any model can follow it. Ships with a two-tier keyword-safety convention so the spec can't arm a run even if pasted into a live agent: the hot keyword appears only as `ultra*code` (an asterisk break inside a code span, never bare in prose), slash-commands are kept off line-starts, and ordinary words are left untouched. The asterisk is stripped only in the final prompt handed to the user. Every prompt it writes now carries a routing block (which route runs which stage, at what effort, with what stop condition), and the spec states plainly that an orchestration run is armed on the workhorse plan, never on a metered reserve plan.

- **[production-pipeline.md](production-pipeline.md)** — a complete agentic development pipeline: Spec Front → Two Loops → Ship. One human thinking phase up front (a relentless one-question-at-a-time interview plus a parallel research fan-out), then an inner agentic loop that iterates to correctness (implement → two-axis code review → autonomous visual sweep → batched fix spec, max 3 sweeps) nested inside an outer human loop that iterates on *feel* only. Every finding becomes a spec before it becomes code; no review follows the operator's green light. The front-half stages are adapted from [Matt Pocock's public engineering skills](https://github.com/mattpocock/skills) (credited as upstream in the spec); the nested two-loop endgame, visual gate, re-spec phase, and label ladder are original doctrine. The inner loop now runs both review axes, and adds a mandatory **triage** stage: sweep output is a pile of observations, not a verdict, and one route with the whole picture de-duplicates, verifies against the cited artifact, and ranks before anything becomes a fix spec.

- **[visual-review.md](visual-review.md)** — the autonomous visual and UX gate, extracted from the pipeline as a skill you can run on its own: walk the running build, screenshot every meaningful state, judge each against the spec's acceptance criteria, triage the observations, batch them into one fix spec, and iterate — at most three times — until an explicit `VERDICT: PASS`. It closes the gap between "the tests pass" and "it looks and behaves right", and it keeps human attention for the one thing an agent cannot judge: feel. Includes the walker brief, the capability check that decides whether a route can hold the role at all, the scoped mid-run sweep for foundation work, and the screenshot-specific redaction rules.

- **[operator-blocker.md](operator-blocker.md)** — a presentation protocol for handing a human the decisions only they can make. Announce the count and stop; then one blocker per message, each a self-contained plain-language nugget with the recommendation first. No identifiers, no stacking, no manufactured decisions. Written for the operator who runs several agent sessions at once and holds none of the codebases in their head — for whom a wall of blockers naming pull requests and subsystems is not a queue but a wall.

Each file is self-contained. It describes trigger conditions, inputs, outputs, workflow, prompt templates, artifact layout, redaction rules, installation recipe, and a smoke-test contract.

## What "portable" means here

These are specifications, not implementations. They:

- Avoid provider, model, account, API, or SDK assumptions. Skills describe route roles and behavior contracts, not fixed vendors.
- Do not assume a particular harness. The installation recipe in each file can be adapted to wherever that system stores skills, slash commands, prompts, or tool definitions.
- Do not assume a particular machine, operating system, or path layout. Output directories and open-commands are defaults, not requirements.
- Require only local shell access and whatever CLI or wrapper the host uses to call an LLM.
- Allow sufficient timeout for expert and review processes. Some models, local hardware, and larger evidence bundles can take substantially longer than ordinary shell commands; implementations should expose configurable timeouts and avoid killing a healthy long-running expert just because it is slow.

`mythos-reserve-routing` used to name concrete models and products as its worked example. It no longer does: every route in it is a role — "Mythos-class reserve", the workhorse's orchestration / standard / mechanical rungs, correctness reviewer, security reviewer, advisory reviewer. Its self-update clause has your current frontier model re-derive the concrete mapping whenever subscriptions or model rankings change, and its installation recipe tells you to keep that mapping in a machine-readable routing config rather than in prose.

`ultra-prompt` is the one spec intentionally tied to a single product — Claude Code's Ultra-Code feature — because its whole job is to write good prompts for *that* feature. It stays portable in the only sense that matters here: any model can run it, and its mechanics section tells the installing agent to re-verify version-specific details against current docs. Its two-tier keyword-safety convention (the hot keyword broken as `ultra*code` inside a code span, slash-commands kept off line-starts, common words left alone) is deliberate, not a typo — see the spec.

`production-pipeline` names one external project — [Matt Pocock's skills repo](https://github.com/mattpocock/skills) — as the credited upstream for its front-half stages, but stays fully role-based on models: every route is a placeholder the installing agent fills after local research, and the one concrete mapping in it is marked example-only.

The two-expert specs explicitly call for two genuinely different LLMs — ideally different families, vendors, or training pipelines — so the panel benefits from cross-model disagreement. Fallbacks for single-model setups are documented.

## How to use

Two ways.

### 1. Ask a capable LLM to install the skill

Paste or hand the spec to an LLM agent with file-writing access and ask it to install the skill in your harness. The spec is written to be sufficient for that on its own:

> Install the skill described in `expert-meeting.md` on this machine. Use this system's default LLM CLI for both expert routes. Place the skill at the harness-appropriate location.

The LLM will create the skill file, wire up the runner, and (if the spec's smoke-test section is followed) verify the install deterministically.

### 2. Read the spec and implement it yourself

Each file is structured for a human reader. The prompt templates are verbatim. The workflow is numbered. The artifact layout is explicit. A competent engineer can build either panel skill in a few hundred lines of shell or Python that shells out to local CLIs. The content-presentation skill is mostly a routing rule plus a stylesheet and needs no runtime of its own.

## Why these thirteen

They are the skills I use most often, in this order:

1. **content-presentation** — replaces the anti-pattern of dumping large content back into the chat. The routing is binary: editor for source, browser for everything else.
2. **expert-code-review-panel** — two frontier models reviewing the same ref independently, then arguing until they converge or one of them refuses. Catches what a single-model review misses.
3. **expert-meeting** — same idea for non-code decisions. Useful before committing to a direction, especially when the failure mode of single-model advice is overconfidence.
4. **tenth-man** — the default standalone dissent gate: one skeptical reviewer, hard verdict, blockers first — plus the second-axis rule that makes a merge gate two reviewers, not one.
5. **double-up-code-review** — for PRs where the two blocking axes are still not enough confidence: a third independent family as an advisory voice, with fail-closed rules and delta-scoped re-review.
6. **model-culture-implementation-routing** — prevents stale brand-based routing by having the installer research the current local model set, then map each route to the phase where its working style creates leverage.
7. **skill-publish** — turns useful local skills into reusable public specifications without leaking private context or freezing one user's model stack as universal.
8. **file-self-destruct** — makes plaintext secret files expire by construction: a deterministic, timed, single-path deletion armed the moment the file is written, on the platform-native scheduler, so a leaked credential file is never left lying around.
9. **mythos-reserve-routing** — maximizes what the latest frontier model actually delivers by stopping the most common subscription failure mode: burning a week's quota on work a workhorse model does identically, then having nothing left for the problem only the frontier model can solve. Expensive tokens buy thinking; cheap tokens buy doing — without ever dropping a review axis. The leak people fix last is the review lane itself, because a per-PR gate looks small and runs on every PR.
10. **ultra-prompt** — turns a task into a disciplined prompt for Claude Code's Ultra-Code runs, where the difference between a tight, scoped, plan-first brief and a vague one-liner is a 10x token swing. Always returns the prompt *plus* the cheapest configuration that still clears the bar, and ships with a keyword-safety convention so the spec itself can't arm a run even if pasted into a live agent.
11. **visual-review** — the gate that keeps the human out of the correctness loop entirely. Everything an attentive observer could catch, an agent catches first, with a screenshot to prove it; the human's scarce judgment is spent only on feel.
12. **operator-blocker** — the counterpart on the other side of that handoff. Getting the work right is half of it; the other half is asking for a decision in a form a busy human can actually answer in one pass.
13. **production-pipeline** — the umbrella workflow the other skills serve: spec everything first, let agents iterate to correctness in a bounded inner loop with review and an autonomous visual gate inside it, and reserve the human for the two things only a human can do — the design decisions up front and the feel judgment at the end. Findings become specs before they become code, and nothing re-reviews after the operator's green light.

All are designed to fail safe: no deploys, no external messages, no code changes unless explicitly requested. The single intentional exception is `file-self-destruct`, whose whole purpose is a scoped, single-path deletion of a secret file you asked to expire.

## Keeping these current

Specs rot quietly: the local skill they were written from keeps moving, and the published copy does not. `skill-publish` carries the refresh procedure. Run a pass when a mirrored local skill changes materially, when a family named in an example ships a major version or retires a tier, when a doctrine changes (a new required gate, a new bound, a retired rule), or when a spec pins a product version that shipped builds have long passed. Prefer converting a stale brand name into a role name over updating it to the newest brand name — the role will still be right next quarter.

## Contributing

Pull requests welcome. Keep specifications implementation-agnostic. If a prompt template, workflow step, or artifact field changes, update the affected spec in this repo and note why.

## License

MIT. See [LICENSE](LICENSE).
