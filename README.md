# agent-skills

Portable, implementation-agnostic specifications for four LLM agent skills:

- **[expert-code-review-panel.md](expert-code-review-panel.md)** — a two-expert code review panel with evidence freeze, adversarial critique, alternating discussion rounds, convergence rule, optional implementation, and verification.
- **[expert-meeting.md](expert-meeting.md)** — the same two-expert pattern for non-code topics: strategy, product, hiring, research, architecture. Context freeze instead of evidence freeze; no implementation phase.
- **[content-presentation.md](content-presentation.md)** — routes "show me / open / present" requests to the right surface: text editor for code and plain-text files, HTML-rendered-in-browser for data, digests, tables, and assembled reports. Includes a reusable dark-theme stylesheet.
- **[eleventh-man-pr-review.md](eleventh-man-pr-review.md)** — a PR review gate that combines the normal differential 10th Man antagonist with an independent Gemini CLI antagonist, then requires follow-up fixes to be re-reviewed by the same full gate.

Each file is self-contained. It describes trigger conditions, inputs, outputs, workflow, prompt templates, artifact layout, redaction rules, installation recipe, and a smoke-test contract.

## What "portable" means here

These are specifications, not implementations. They:

- Avoid provider, model, account, API, or SDK assumptions unless a skill explicitly names a route as part of its contract. `eleventh-man-pr-review` intentionally requires a Gemini CLI reviewer.
- Do not assume a particular harness. The installation recipe in each file can be adapted to wherever that system stores skills, slash commands, prompts, or tool definitions.
- Do not assume a particular machine, operating system, or path layout. Output directories and open-commands are defaults, not requirements.
- Require only local shell access and whatever CLI or wrapper the host uses to call an LLM (for example `claude -p`, `codex exec`, `ollama run`, or a local model-server wrapper).

The two-expert specs explicitly call for two genuinely different LLMs — ideally different families, vendors, or training pipelines — so the panel benefits from cross-model disagreement. Fallbacks for single-model setups are documented.

## How to use

Two ways.

### 1. Ask a capable LLM to install the skill

Paste or hand the spec to an LLM agent with file-writing access (Claude Code, Codex CLI, Cursor, a coding agent, or anything equivalent) and ask it to install the skill in your harness. The spec is written to be sufficient for that on its own:

> Install the skill described in `expert-meeting.md` on this machine. Use this system's default LLM CLI for both expert routes. Place the skill at the harness-appropriate location.

The LLM will create the skill file, wire up the runner, and (if the spec's smoke-test section is followed) verify the install deterministically.

### 2. Read the spec and implement it yourself

Each file is structured for a human reader. The prompt templates are verbatim. The workflow is numbered. The artifact layout is explicit. A competent engineer can build either panel skill in a few hundred lines of shell or Python that shells out to local CLIs. The content-presentation skill is mostly a routing rule plus a stylesheet and needs no runtime of its own.

## Why these four

They are the skills I use most often, in this order:

1. **content-presentation** — replaces the anti-pattern of dumping large content back into the chat. The routing is binary: editor for source, browser for everything else.
2. **expert-code-review-panel** — two frontier models reviewing the same ref independently, then arguing until they converge or one of them refuses. Catches what a single-model review misses.
3. **expert-meeting** — same idea for non-code decisions. Useful before committing to a direction, especially when the failure mode of single-model advice is overconfidence.
4. **eleventh-man-pr-review** — a narrower merge gate for PRs where one antagonist is not enough: normal 10th Man dissent plus Gemini CLI, with fail-closed re-review rules.

All four are designed to fail safe: no destructive actions, no deploys, no external messages, no code changes unless explicitly requested.

## Contributing

Pull requests welcome. Keep specifications implementation-agnostic. If a prompt template, workflow step, or artifact field changes, update the affected spec in this repo and note why.

## License

MIT. See [LICENSE](LICENSE).
