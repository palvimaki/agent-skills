# Skill Publish Skill Specification

This document defines a reusable skill for publishing a private or local skill
as a public, portable skill specification. It is intentionally
provider-agnostic, model-agnostic, account-agnostic, machine-agnostic,
repository-agnostic, and implementation-agnostic.

The skill can be implemented by any capable LLM agent that can read a source
skill, redact identifying material, rewrite the skill as a generic recipe, and
publish it to the user's open-source skill repository.

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
- Commit and push only after redaction and verification pass.

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
8. Add a smoke test that verifies the public spec produces the intended local
   behavior without relying on private infrastructure.
9. Scan the new artifact for identifying info and secrets.
10. Update the target repository README or index if that is the local pattern.
11. Review the diff.
12. Commit and push to the public repository.

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
- comments that reveal non-public incidents or operations.

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

## Failure Modes

Block publication when:

- redaction finds a secret or private identifier that cannot be safely removed;
- the target repo has unrelated dirty changes;
- the public rewrite still depends on private infrastructure;
- the model guidance is stale or presented as universal;
- the agent cannot verify what will be published.
