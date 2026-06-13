# agent-skills

Elevate your agentic coding (and publish your own skills!) with these powerful skills

Portable, implementation-agnostic specifications for ten LLM agent skills:

- **[expert-code-review-panel.md](expert-code-review-panel.md)** — a two-expert code review panel with evidence freeze, adversarial critique, alternating discussion rounds, convergence rule, optional implementation, and verification.
- **[expert-meeting.md](expert-meeting.md)** — the same two-expert pattern for non-code topics: strategy, product, hiring, research, architecture. Context freeze instead of evidence freeze; no implementation phase.
- **[content-presentation.md](content-presentation.md)** — routes "show me / open / present" requests to the right surface: text editor for code and plain-text files, HTML-rendered-in-browser for data, digests, tables, and assembled reports. Includes a reusable dark-theme stylesheet.
- **[tenth-man.md](tenth-man.md)** — a standalone antagonistic review gate for code, plans, architecture, research, operations, and release decisions, with severity-ranked findings and a hard `GO` / `NOGO` verdict.
- **[double-up-code-review.md](double-up-code-review.md)** — a stricter PR review gate that combines the normal differential 10th Man antagonist with an independent second antagonist, then requires follow-up fixes to be re-reviewed by the same full gate.
- **[model-culture-implementation-routing.md](model-culture-implementation-routing.md)** — a model-agnostic recipe for creating a local routing skill that assigns exploration, design, execution, and review phases by current model temperament and observed strengths.
- **[skill-publish.md](skill-publish.md)** — a publication workflow for turning private/local skills into public-safe, model-agnostic skill specifications with redaction, examples, README updates, commit, and push.
- **[file-self-destruct.md](file-self-destruct.md)** — arms a deterministic, timed, surgical self-destruct on any plaintext secret file the moment it is created or imported, using the host's native one-shot scheduler (launchd / systemd / `at`) rather than legacy cron, with traceless teardown.
- **[mythos-reserve-routing.md](mythos-reserve-routing.md)** — a token-effectiveness routing protocol for getting the most out of the latest, most expensive frontier model across two frontier subscriptions: treat the "Mythos-class" model (a role, not a brand) as a strategic reserve for the reasoning only it can do, route all volume work to a capable workhorse, keep a mandatory independent review gate on every PR, and decide inline-vs-delegate by token economics. Budget optimization follows automatically — but the goal is maximum benefit per frontier token, not frugality. Uses today's Claude Code + Codex GPT-5.5 stack as the worked example and includes a self-update clause for when the landscape shifts.
- **[ultra-prompt.md](ultra-prompt.md)** — writes a best-practice prompt for Claude Code's Ultra-Code orchestration feature (its `ultra*code` effort tier and the dynamic many-subagent runs it spawns) and pairs it with a tailored token-efficiency configuration (effort tier, scope, per-stage model, run hygiene). Product-specific by design — it is *about* one named feature — but runner-agnostic: any model can follow it. Ships with a two-tier keyword-safety convention so the spec can't arm a run even if pasted into a live agent: the hot keyword appears only as `ultra*code` (an asterisk break inside a code span, never bare in prose), slash-commands are kept off line-starts, and ordinary words are left untouched. The asterisk is stripped only in the final prompt handed to the user.

Each file is self-contained. It describes trigger conditions, inputs, outputs, workflow, prompt templates, artifact layout, redaction rules, installation recipe, and a smoke-test contract.

## What "portable" means here

These are specifications, not implementations. They:

- Avoid provider, model, account, API, or SDK assumptions. Skills describe route roles and behavior contracts, not fixed vendors.
- Do not assume a particular harness. The installation recipe in each file can be adapted to wherever that system stores skills, slash commands, prompts, or tool definitions.
- Do not assume a particular machine, operating system, or path layout. Output directories and open-commands are defaults, not requirements.
- Require only local shell access and whatever CLI or wrapper the host uses to call an LLM.
- Allow sufficient timeout for expert and review processes. Some models, local hardware, and larger evidence bundles can take substantially longer than ordinary shell commands; implementations should expose configurable timeouts and avoid killing a healthy long-running expert just because it is slow.

`mythos-reserve-routing` names concrete models and products (Claude Code, Codex, GPT-5.5, Gemini CLI) more freely than the other specs, but only as a worked example — the spec itself is role-based ("Mythos-class reserve", "workhorse"), and its self-update clause has your current frontier model re-derive the concrete mapping whenever subscriptions or model rankings change.

`ultra-prompt` is the one spec intentionally tied to a single product — Claude Code's Ultra-Code feature — because its whole job is to write good prompts for *that* feature. It stays portable in the only sense that matters here: any model can run it, and its mechanics section tells the installing agent to re-verify version-specific details against current docs. Its two-tier keyword-safety convention (the hot keyword broken as `ultra*code` inside a code span, slash-commands kept off line-starts, common words left alone) is deliberate, not a typo — see the spec.

The two-expert specs explicitly call for two genuinely different LLMs — ideally different families, vendors, or training pipelines — so the panel benefits from cross-model disagreement. Fallbacks for single-model setups are documented.

## How to use

Two ways.

### 1. Ask a capable LLM to install the skill

Paste or hand the spec to an LLM agent with file-writing access and ask it to install the skill in your harness. The spec is written to be sufficient for that on its own:

> Install the skill described in `expert-meeting.md` on this machine. Use this system's default LLM CLI for both expert routes. Place the skill at the harness-appropriate location.

The LLM will create the skill file, wire up the runner, and (if the spec's smoke-test section is followed) verify the install deterministically.

### 2. Read the spec and implement it yourself

Each file is structured for a human reader. The prompt templates are verbatim. The workflow is numbered. The artifact layout is explicit. A competent engineer can build either panel skill in a few hundred lines of shell or Python that shells out to local CLIs. The content-presentation skill is mostly a routing rule plus a stylesheet and needs no runtime of its own.

## Why these ten

They are the skills I use most often, in this order:

1. **content-presentation** — replaces the anti-pattern of dumping large content back into the chat. The routing is binary: editor for source, browser for everything else.
2. **expert-code-review-panel** — two frontier models reviewing the same ref independently, then arguing until they converge or one of them refuses. Catches what a single-model review misses.
3. **expert-meeting** — same idea for non-code decisions. Useful before committing to a direction, especially when the failure mode of single-model advice is overconfidence.
4. **tenth-man** — the default standalone dissent gate: one skeptical reviewer, hard verdict, blockers first.
5. **double-up-code-review** — a narrower merge gate for PRs where one antagonist is not enough: normal 10th Man dissent plus a second independent reviewer, with fail-closed re-review rules.
6. **model-culture-implementation-routing** — prevents stale brand-based routing by having the installer research the current local model set, then map each route to the phase where its working style creates leverage.
7. **skill-publish** — turns useful local skills into reusable public specifications without leaking private context or freezing one user's model stack as universal.
8. **file-self-destruct** — makes plaintext secret files expire by construction: a deterministic, timed, single-path deletion armed the moment the file is written, on the platform-native scheduler, so a leaked credential file is never left lying around.
9. **mythos-reserve-routing** — maximizes what the latest frontier model actually delivers by stopping the most common subscription failure mode: burning a week's quota on work a workhorse model does identically, then having nothing left for the problem only the frontier model can solve. Expensive tokens buy thinking; cheap tokens buy doing — without ever dropping the PR review gate.
10. **ultra-prompt** — turns a task into a disciplined prompt for Claude Code's Ultra-Code runs, where the difference between a tight, scoped, plan-first brief and a vague one-liner is a 10x token swing. Always returns the prompt *plus* the cheapest configuration that still clears the bar, and ships with a keyword-safety convention so the spec itself can't arm a run even if pasted into a live agent.

All are designed to fail safe: no deploys, no external messages, no code changes unless explicitly requested. The single intentional exception is `file-self-destruct`, whose whole purpose is a scoped, single-path deletion of a secret file you asked to expire.

## Contributing

Pull requests welcome. Keep specifications implementation-agnostic. If a prompt template, workflow step, or artifact field changes, update the affected spec in this repo and note why.

## License

MIT. See [LICENSE](LICENSE).
