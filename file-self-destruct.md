# File Self-Destruct Skill Specification

This document defines a reusable skill: **whenever a secret is written to disk in
unencrypted form, a deterministic, timed, surgical self-destruct is armed for
that file at the moment it is created or imported.** It is intentionally
provider-agnostic, scheduler-agnostic, and machine-agnostic.

Any capable LLM agent that can write a file and register a one-shot scheduled job
on the host can implement it. No LLM API, SDK, or provider dependency is
required.

> **Doctrine version: 2026-08-19.** Re-verify scheduler behavior on your own
> platform before you install. This is the one skill in this repository that
> deletes a file, so verify it rather than trusting it.

## Contract

- The arming step happens in the **same step** that writes the file, before any
  report back. Deferred arming is the failure mode this skill exists to stop.
- One explicit path. Never a glob, never a directory, never a pattern.
- Use the platform's native one-shot scheduler, not the legacy cron daemon.
- The shortest workable expiry. Default short, extend only for a stated reason.
- Never echo the secret into the transcript, the log, or the job definition.
- Never arm on the managed secret store. That store's protection is permissions
  and encryption, not expiry.
- The job removes itself after it fires, leaving nothing behind to fire twice.

## Skill Identity

Name: `file-self-destruct`.

Purpose: guarantee that plaintext secret files always expire. The instant such a
file is created or imported, arm a scheduled job that deletes *exactly that file*
at a fixed future time — so no plaintext secret is ever left on disk relying on
someone to remember to clean it up.

## The Invariant

A plaintext secret sitting in a file is a leak waiting to happen. So the deletion
must be:

- **Deterministic** — fires on wall-clock time via a system scheduler,
  independent of whether the agent, shell, or session that created the file is
  still alive.
- **Timed** — a bounded TTL; default to a short window (e.g. 10 minutes) and pick
  the shortest that works.
- **Surgical** — deletes exactly one explicit path. Never a glob, never a
  directory.

Arm it in the **same step** that writes the file, before reporting back. It is
not optional and not deferred to a later cleanup pass.

## When to Use

- Displaying or exporting a credential, API key, token, password, wireless/WiFi
  key, connection string, `.env`, or any secret to a file for a human to read or
  copy.
- Staging a loose secret file for a tool that only reads credentials from disk.
- Any time plaintext secret material lands on disk outside a managed or encrypted
  store.

## Scope / Boundary

Applies to **ad-hoc / transient** plaintext secret files: credential displays,
dumps, exports, loose env-file drops, temp token files.

Does **NOT** apply to a managed or encrypted secret store — the intended
permanent home for credentials, governed by access permissions and/or
encryption-at-rest, not by expiry. Never arm a self-destruct on the store; doing
so would wipe live credentials. Pair this skill with the host's secret-storage
workflow so that *permanent* secrets are routed into the managed store and only
*transient* plaintext files self-destruct.

## Inputs

- The exact destination **file path**.
- A **TTL** in minutes (default a short window such as `10`).
- The **secret content on stdin** — never as a command-line argument (it would
  leak into shell history and the process list), and never echoed into the chat
  transcript.

## Outputs

- The file created with **owner-only permissions** (e.g. `0600`).
- A **registered one-shot deletion job** for that exact path.
- A report: file path, permission mode, scheduled wipe time, and the scheduler
  job identifier. **Never the secret value.**

## Choosing the Scheduler (implementation-agnostic)

The deletion mechanism MUST be deterministic, one-shot, surgical, unattended
(runs without prompting and survives the agent exiting), and ideally traceless
(removes its own scheduling artifacts after firing).

Prefer the host's **native scheduler**. Do **not** reach for legacy `cron` by
reflex:

- **macOS** — a per-user launchd LaunchAgent with a `StartCalendarInterval`
  (Hour + Minute), loaded via `launchctl bootstrap gui/<uid> <plist>`. Avoid
  `cron`: on recent macOS the legacy cron daemon is gated behind a
  privacy / Full-Disk-Access consent prompt, so installing a user crontab can
  surface a permission dialog and, if it is dismissed, the job may **silently
  never fire** — which defeats the entire guarantee this skill exists to provide.
- **Linux with systemd** — a transient user timer:
  `systemd-run --user --on-active=10m rm -f <path>` (self-cleaning), or `at` if
  `atd` is enabled.
- **Portable fallback** — a detached background process (`setsid` / `nohup` that
  sleeps then deletes). Works anywhere, but does **not** survive a reboot, so use
  it only for short windows and state that limitation.

Whatever the user calls it — "cron job", "timed task", "scheduled task" — choose
the platform-native, promptless, unattended one-shot mechanism, and **verify the
job is actually registered** before reporting success.

## Critical Teardown Ordering

If the self-destruct job cleans up after itself (recommended), it must act in
this order:

1. **Delete the target secret file** — the essential action.
2. **Delete its own scheduling artifacts** (e.g. the launchd plist and any helper
   script).
3. **Deregister / unload itself LAST.**

Deregistering a job often terminates the very process performing the teardown, so
any cleanup placed *after* deregistration may never run — leaving orphaned
scheduler artifacts even though the secret was deleted. Wipe first, self-remove
second, deregister last.

## Workflow

1. Receive the secret on stdin, plus the destination path and TTL.
2. Write the file under a restrictive umask; set owner-only permissions (`0600`).
3. Compute the fire time = now + TTL.
4. Generate a uniquely-named one-shot job that deletes exactly that path at the
   fire time and then self-cleans per the teardown ordering above.
5. Register the job with the platform-native scheduler.
6. Verify the job is registered and the file permissions are correct.
7. Report path, mode, wipe time, and job id. Never the secret value.

## What NOT to Do

- Do not pass the secret as a command-line argument (it leaks to shell history
  and the process list).
- Do not echo the secret into the chat transcript.
- Do not use a recurring scheduler entry where a one-shot is required.
- Do not delete by glob or directory — one explicit path only.
- Do not arm self-destruct on a managed or encrypted secret store.
- Do not deregister the job before its own cleanup has run.
- Do not default to legacy `cron` on systems where it requires special
  privileges or can fail silently.

## Installation Recipe (implementation-agnostic)

An LLM installing this skill on a fresh system should:

1. Create a skill or prompt entry the harness surfaces whenever the agent is
   about to write a plaintext secret to a file. Use the equivalent skill, prompt,
   tool, or agent-config location for that harness.
2. Ship a small helper the agent can call instead of composing scheduler commands
   each time, e.g. `self-destruct <path> [ttl_minutes]` that reads the secret on
   stdin, writes it `0600`, and arms the one-shot deletion.
3. Detect the platform and select the native scheduler (launchd / systemd / `at`
   / portable fallback) accordingly.
4. Encode the teardown ordering so fired jobs are traceless.
5. Pair with the host's secret-storage workflow so permanent secrets go to the
   managed store and only transient files self-destruct.
6. If the system has organization-wide LLM instruction files, add a short
   reference so agents discover and apply the invariant without being told.

## Smoke Test

Deterministic and offline:

- **Input:** a non-secret fixture written via the helper with a 1-minute TTL.
- **Assert:** the file exists with owner-only permissions, and the one-shot job
  is registered with the scheduler.
- **Wait** past the fire minute.
- **Assert:** the file is gone, the scheduling artifacts are gone, and the job is
  deregistered.

The smoke test requires no network access. It verifies permissions, scheduling,
firing, and traceless cleanup only.
