# Skill Publish Skill Specification

This document defines a reusable skill for publishing a private or local skill
as a public, portable skill specification. It is intentionally
provider-agnostic, model-agnostic, account-agnostic, machine-agnostic,
repository-agnostic, and implementation-agnostic.

The skill can be implemented by any capable LLM agent that can read a source
skill, redact identifying material, rewrite the skill as a generic recipe, and
publish it to the user's open-source skill repository.

> **Doctrine version: 2026-08-19.** This spec now also defines the house style
> every published spec in this repository follows: a doctrine-version line, a
> short Contract block, role names instead of brand names, and a currency
> check. Apply it to new publications and to refreshes of old ones.

## Contract

- Publish a specification, not a copy of the local skill.
- Redact first, verify second, commit third. A failed redaction blocks the push.
- Describe routes by role. Brand names belong only in a section marked as an
  example.
- Every published spec opens with a dated doctrine-version line and a Contract
  block of about ten lines.
- Never publish an absolute local path, hostname, username, private
  organization or project name, wrapper command, ticket identifier, or internal
  directive date.
- Stop on unrelated dirty changes in the target repository.
- Update the repository index in the same change as the new spec.

## Skill Identity

Name: `skill-publish`

Purpose: convert a local skill, workflow, or prompt into a public skill
specification that another agent can use to create a locally applicable version
for its own user.

Use this skill when the user asks to:

- publish a skill to a public agent-skills or open-source skill repository;
- make a private skill public-safe;
- remove identifying information, secrets, paths, accounts, or infrastructure
  details from a skill;
- rewrite a model-specific or organization-specific skill as a portable,
  model-agnostic recipe;
- preserve a concrete local example while labeling it as illustrative only.

## Non-Negotiable Constraints

- Never publish secrets, credentials, private keys, API tokens, account IDs,
  cookies, session IDs, credential URLs, private hostnames, local usernames,
  private organization names, internal project names, or absolute local paths.
- Do not publish a skill as if the source user's local setup is universal.
- Do not hardcode provider, model, account, CLI, OS, repository, or path
  assumptions unless they are clearly marked as placeholders or examples.
- Make the public artifact a specification or recipe, not a dump of the private
  local skill.
- If model names appear, use them only as examples unless the skill is
  explicitly about that model. Explain that installers must research and adapt
  the examples to their current local model set.
- Require the installing agent to research current strengths, weaknesses,
  limits, and tool behavior before filling local blanks.
- Preserve the useful principle of the source skill while removing private
  operational detail.
- Use the style of the target public repository.
- Describe every route, model, and tool by the **role it plays**, not by brand.
  Role names do not go stale, and they do not reveal whose stack the skill came
  from. Keep brand names inside a clearly marked example section.
- Open every published spec with a dated doctrine-version line, so a reader can
  judge freshness without reading the git history, and with a Contract block of
  about ten lines carrying the normative core — an installing agent should be
  able to act correctly from the Contract alone and read the body for detail.
- Redact internal wrapper and command names. They are private infrastructure
  even when they contain no secret, and they tell a reader nothing portable.
- Redact internal directive dates and decision references. "A standing rule
  says X" is portable; a dated internal directive is a fingerprint.
- Commit and push only after redaction and verification pass.

## Timeout and Long-Running Work

Publishing can involve long-running review, redaction, rendering, test, or
repository checks. Some expert routes may also be slow because of model choice,
remote queues, or local hardware. Implementations must expose configurable
timeouts, choose generous defaults for expert and verification steps, preserve
partial output when possible, and avoid treating slow but active work as failed
solely because it exceeds a short generic command timeout.

## Required Inputs

The skill should work with these inputs:

- `source_skill`: local skill text, file, folder, prompt, or workflow to publish.
- `target_repo`: public skill repository. Default: the user's configured public
  agent-skills repository.
- `target_name`: public skill name. Default: derive a concise kebab-case name
  from the source skill.
- `publication_style`: target repo conventions. Default: inspect existing files
  and match their structure.
- `example_policy`: whether to keep local examples. Default: keep only if they
  clarify the principle and label them as non-authoritative examples.

## Required Outputs

Always produce:

- a public-safe skill specification in the target repository;
- any target index or README update required by the repository style;
- a redaction summary;
- verification results;
- a git commit and push when the target repository is clean and the user asked
  to publish.

Do not modify unrelated files. If the target repository has unrelated dirty
changes, stop and report the conflict unless the user explicitly authorized
working around them.

## Workflow

1. Locate the target public skill repository.
2. Inspect its existing file format, tone, naming, README/index pattern, and
   redaction conventions.
3. Read the source skill or workflow.
4. Extract the transferable principle, trigger conditions, required inputs,
   required outputs, workflow, redaction rules, installation recipe, and smoke
   tests.
5. Remove or replace local-only material:
   - usernames, hostnames, paths, account names, project names;
   - private organization references;
   - secrets and credential-like strings;
   - provider subscriptions, API keys, and internal CLI wrappers;
   - assumptions about one machine, repo, branch, or deployment target.
6. Rewrite the skill as a portable recipe that tells the installing agent how
   to create a local version for its own user.
7. If the source contains specific model-route guidance, convert it into a
   model-agnostic method:
   - require current research at install time;
   - use placeholders for local model routes;
   - keep source model descriptions only in an "Example Only" section when they
     clarify the principle;
   - state that examples must be modified according to local findings,
     constraints, tools, limits, and observed model performance.
8. Add the doctrine-version line and the Contract block in the repository's
   house style.
9. Add a smoke test that verifies the public spec produces the intended local
   behavior without relying on private infrastructure.
10. Scan the new artifact for identifying info and secrets.
11. Update the target repository README or index if that is the local pattern.
12. Review the diff.
13. Commit and push to the public repository.

## Model-Agnostic Rewrite Rule

When converting a model-specific skill, do not say "always use Model X for
Phase Y" in the public spec. Instead, instruct the installing agent to:

1. Discover available local model or agent routes.
2. Research current strengths, weaknesses, limits, and tool behavior using
   current official docs, release notes, model cards, local routing configs,
   local task history, and recent evaluations where relevant.
3. Fill a local route table with placeholders such as `<Expansion Route>`,
   `<Coherence Route>`, `<Execution Route>`, and `<Review Route>`.
4. Preserve any source model mapping only as an illustrative example.
5. State that local findings override the example.

Example language:

```text
The following model mapping is an example only. It illustrates the principle and
must be replaced or edited according to current local findings, available tools,
privacy constraints, rate limits, cost, latency, and observed model
performance.
```

## Redaction Checklist

Before publishing, scan for:

- private key blocks and credential formats;
- environment variables containing `key`, `token`, `secret`, `password`,
  `credential`, `cookie`, or `session`;
- URLs with embedded credentials;
- local usernames and home-directory paths;
- hostnames and machine names;
- private organization, client, or project names;
- account IDs, emails, phone numbers, messaging IDs;
- internal domain names or deployment targets;
- private issue trackers, ticket IDs, PR URLs, or branch names;
- internal wrapper, script, and command names, even when they hold no secret;
- dated internal directives, decision logs, and standing-order references;
- comments that reveal non-public incidents or operations;
- product names, customer names, and domain names belonging to the source
  environment.

Use placeholders such as `<user>`, `<repo>`, `<target_repo>`, `<local skill
directory>`, `<Execution Route>`, and `[REDACTED_SECRET]`.

## Smoke Test

After drafting the public spec, run these checks:

```text
Can an agent with no private context understand what to build?
```

Expected: yes, because the public spec explains purpose, inputs, workflow,
outputs, redaction, and installation.

```text
Does the spec reveal private local details?
```

Expected: no. Any local details are placeholders or removed.

```text
Does the spec hardcode one provider or model as universal?
```

Expected: no. Any concrete model mapping is clearly marked as illustrative and
requires current local research before installation.

```text
Does the target README or index list the new skill?
```

Expected: yes, when the target repository maintains such an index.

```text
Does the spec open with a dated doctrine-version line and a Contract block?
```

Expected: yes. A reader must be able to judge freshness and act on the
normative core without reading the whole file.

```text
Grep the new file for the source environment's fingerprints.
```

Expected: no hits. Check for home-directory paths, host names, user names,
internal wrapper names, product names, ticket prefixes, and internal directive
dates before every push.

## Publication Commands

Command shapes are illustrative. Use the host's safest equivalent:

```bash
git -C "<target_repo>" status --short
git -C "<target_repo>" diff --check
git -C "<target_repo>" diff
git -C "<target_repo>" add "<new_skill_spec>" "<index_file>"
git -C "<target_repo>" commit -m "Add <skill-name> skill spec"
git -C "<target_repo>" push origin "<branch>"
```

If the repository requires pull requests, create a branch and PR instead of
pushing directly to the protected branch.

## Currency Refresh

A published spec ages in a way its author does not notice, because the local
skill it came from keeps moving and the public copy does not. Run a refresh pass
when any of these is true:

- a local skill the repository mirrors has changed materially;
- a model family in an example section has shipped a major version, or has
  renamed or retired a tier named in the spec;
- a doctrine changed — a new required gate, a new bound, a retired rule;
- a spec pins a product version that shipped builds have long passed.

A refresh pass covers: the doctrine-version line, the Contract block, the
example sections, any pinned version numbers, and the cross-references between
specs. Prefer converting a stale brand name to a role name over updating it to
the newest brand name — the role will still be right next quarter.

## Failure Modes

Block publication when:

- redaction finds a secret or private identifier that cannot be safely removed;
- the target repo has unrelated dirty changes;
- the public rewrite still depends on private infrastructure;
- the model guidance is stale or presented as universal;
- the spec has no doctrine-version line or no Contract block;
- the agent cannot verify what will be published.
