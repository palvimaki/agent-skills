# Preserve Artifacts Skill Specification

This document defines a reusable skill for making generated deliverables
durable before an agent reports them. It is intentionally provider-agnostic,
model-agnostic, harness-agnostic, and machine-agnostic.

Any capable LLM agent with file access and a shell can implement it. A small
copy-and-manifest script makes it reliable, but the rules work without one.

> **Doctrine version: 2026-10-20.** This spec adds the provenance label, the
> work-volume rule for heavy evidence, and the no-silent-fallback rule. The
> storage locations are local choices; the rules are the portable part.

## Contract

- A user-facing deliverable never lives only in a temp directory, a worktree, a
  session scratch area or an untracked repository output.
- Before you report a file path, put the file in a durable location: version
  control, or a durable documents folder (the artifact vault).
- Verify each reported path exists immediately before the final reply.
- Label every preserved deliverable with an ISO 8601 timestamp, the model and
  the effort. Write `effort unknown` when you do not know it.
- Preserved binaries get a manifest with source path, preserved path, size and
  hash.
- Heavy evidence (builds, logs, screenshots, recordings) goes to the designated
  work volume, not the system disk.
- If the designated location is missing, stop and report it. Never fall back
  silently to another disk.
- Never preserve secrets, credential dumps, key material or symbolic links.
- If preservation fails, do not claim the files are ready.

## Skill Identity

Name: `preserve-artifacts`.

Purpose: stop the most common silent failure of agent work: the reply points to
a file that a later cleanup, checkout or worktree removal deletes.

Use this skill when:

- the task created a report, bundle, HTML page, PDF, archive, screenshot set,
  transcript, media file or other deliverable the operator will open later;
- the deliverable currently lives in a repository checkout, a worktree, a temp
  directory, a downloads folder or another path that cleanup could remove;
- the final reply will cite a local file path.

Do not use it for source files that belong in the repository. Commit those
through the normal repository workflow.

## The Problem This Solves

Agents work in places that are built to disappear: scratch directories,
disposable worktrees, untracked build folders. The agent's final reply is
correct at the moment it is written. A day later, a `clean` step, a branch
switch or a scheduled cleanup removes the only copy, and the reply now points
at nothing. Nobody notices until the operator needs the file.

## Durable Targets

Choose the most suitable durable location:

| Deliverable | Durable target |
|---|---|
| Source, docs or config that belong in the product | Commit in version control, on a branch that outlives the worktree |
| Small final deliverables for the operator | `<artifact vault>` in a durable documents folder |
| Bulky evidence: builds, logs, screenshots, recordings | `<work volume>/<evidence area>` |
| Sensitive material | The approved `<secret store>`, never the vault |

When in doubt, preserve a copy in the artifact vault and report the vault
manifest path.

## Required Inputs

- `paths`: every file or directory the final reply will cite.
- `slug`: a short task name for the package directory.
- `generated_by`: the generating model family, version and effort, for example
  `<model> <version> <effort>` or `<model> effort unknown`.
- `note`: one line on what the files are.

## Required Outputs

- A package directory, for example
  `<artifact vault>/<YYYY-MM-DD>/<slug>-<HHMMSS>/`, private to the local user.
- `manifest.json`: machine-readable entries for each file: source path,
  preserved path, size, SHA-256 hash, timestamp, `generated_by` and note.
- `MANIFEST.md`: the same facts for a human reader, with the provenance label at
  the top.
- The durable paths, printed for the final reply.

## Provenance Label

Every preserved deliverable carries a label in this shape:

```text
Generated: <ISO 8601 timestamp with offset> by <model> <version>, <effort or "effort unknown">.
```

Put it at the top of text deliverables and in the manifest for binaries. When
someone revises a deliverable, add a `Revised:` line below the original. Keep
the original line, even when timestamps look inconsistent. The label is
evidence, not decoration.

## Workflow

1. List every file the final reply will cite.
2. Classify each one with the Durable Targets table.
3. Refuse any path that matches a secret pattern: a secret directory, an SSH
   directory, environment files, key, certificate, keystore or
   credentials-looking names, and every symbolic link. Preserve a sanitized
   version instead when one is useful.
4. Check that the target location exists. If the work volume or the vault is
   missing, stop and report it. Do not create a mount point. Do not write to the
   system disk instead.
5. Copy the files, or commit them for version-controlled targets.
6. Write both manifests with hashes and the provenance label.
7. Verify that each preserved path exists and each hash matches.
8. Report the durable path in the final reply. Prefer the preserved path over
   the original scratch path; give the original only as context.

## Final Reply Contract

For each generated deliverable, the final reply includes exactly one of:

- a repository path whose blob is in a commit on a branch that remains after
  worktree removal (a staged file is not durable; until that commit exists,
  preserve a vault copy and cite the vault path);
- a durable vault or work-volume path plus its `MANIFEST.md`;
- a clear statement that the task produced no local file.

## Reference Script Shape

A local script makes the workflow repeatable. Its shape is illustrative:

```bash
<preserve command> \
  --slug "<short-task-name>" \
  --generated-by "<model> <version> <effort>" \
  --note "<what these files are>" \
  "<absolute path to file or directory>" ...
```

The script should require `--generated-by`, refuse secret patterns and
symbolic links, set private permissions on the package, write both manifests,
and exit non-zero on any copy or hash failure.

## Cleanup

After preservation, repository cleanup may remove the untracked scratch copies.
Never remove the vault copy in the same task. Remove a temporary folder with its
literal path in its own command, or leave it in the session scratch area for the
harness to clean.

## Failure Modes

- **The reply points to a path that later disappears.** Cause: the only copy
  lived in a scratch path, worktree or untracked folder. Fix: preserve before the
  final reply.
- **Preservation copied something unsafe.** Cause: the path was a secret, a
  credential dump or a symbolic link. Fix: refuse the path; preserve a sanitized
  artifact.
- **The package cannot be trusted later.** Cause: no manifest, no hash, no
  provenance. Fix: generate both manifests every time.
- **Bulky evidence filled the system disk.** Cause: the work volume was missing
  and the agent fell back silently. Fix: stop and report the missing volume.

## Installation Recipe

1. Create a skill or standing instruction named `preserve-artifacts`.
2. Set its trigger to fire whenever a reply will cite a generated local file.
3. Choose and record the local `<artifact vault>`, `<work volume>` and
   `<secret store>` locations in the operator's configuration, not in the skill
   text.
4. Install a preserve script with the shape above, or instruct the agent to
   follow the Workflow by hand.
5. Add the provenance-label rule to the always-loaded instructions so it applies
   to every generated report.

## Smoke Test

1. Generate an HTML report in a temp directory. Confirm the reply cites a vault
   path with a manifest, not the temp path.
2. Confirm the manifest has a hash, a timestamp and `generated_by` with an
   effort value or `effort unknown`.
3. Include an environment file in the input list. Confirm it is refused.
4. Include a symbolic link. Confirm it is refused.
5. Unmount or rename the work volume and request a screenshot set. Confirm the
   agent stops and reports it, and writes nothing to the system disk.
6. Delete the original temp directory. Confirm the vault copy still opens.

## Redaction

When installing from or publishing back to a private environment, remove home
directory paths, volume and host names, vault folder names that reveal the
organization, and script names. Publish the rules, not the file system.
