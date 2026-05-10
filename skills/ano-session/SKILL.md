---
name: ano-session
description: |
  Record agent sessions in the workspace's Ano "Agent Status" list so the
  team can see which humans are running which agents on which workstreams,
  in real time. Auto-fires at session start, on significant milestones
  (commits, PRs, test runs), and at session end.

  Posts ONLY when the user has opted their machine in (`ano session enable`).
  When opted out (or never opted in), the skill makes exactly one CLI call
  per Claude Code session — no continuous noise. See "One-shot rule" below.
triggers:
  - claude code session
  - working on
  - new branch
  - new worktree
  - starting work
  - made progress
  - finished
  - shipping
invocable: false
---

# Ano Session — Agent Status Tracking

## What this skill does

Each Claude Code session is a **workstream**: one human, one branch / worktree,
one objective. This skill records that workstream as a row in the workspace's
**Agent Status** list (an Ano Lists template) so the rest of the team can see,
at a glance:

- Who's running an agent right now
- What branch / worktree they're on
- A short title for what's being worked on
- Status (active / paused / done / failed) and the latest progress note

The agent posts via the Ano CLI:

- `ano session start --title "..."` — at the START of the session, exactly once.
- `ano session update --summary "..."` — when something noteworthy happens.
- `ano session end --status done --summary "..."` — at the END.

## The one-shot rule (load-bearing)

**At the start of a fresh Claude Code session, do this in order:**

1. Detect a one-line title for the session from the user's first instruction.
   Aim for ≤ 80 chars, present-tense verb. Examples:
   - "Refactor auth middleware"
   - "Investigate notification fan-out regression"
   - "Add Slack bridge MVP"

2. Run `ano session start --title "<the title>"` exactly **ONCE**. Do NOT
   run it again later in the same session.

3. **Read STDOUT.** The CLI prints `session_id=<uuid>` to stdout when the
   user is opted in. If you see a line matching `^session_id=([0-9a-f-]+)$`,
   capture the UUID and proceed to milestone + end behavior.

4. **If no `session_id=…` line appeared on stdout** (the user is unset or
   disabled), do **NOT** call `ano session update` or `ano session end` for
   the rest of this Claude Code session. Just stop. The CLI may have written
   a one-line "tracking available — run `ano session enable` to opt in"
   message to STDERR — that's intentional and the user has been informed.

This rule is what bounds the surface to **at most one CLI call per Claude
Code session** when the user hasn't opted in. Without it, you'd attempt to
post on every commit + PR + test run, and the user would see "disabled" no-op
messages on every action. The CLI's three-state opt-in (unset / enabled /
disabled) handles the rest — you don't need to check it yourself.

## Milestones (only when start succeeded)

After capturing a `session_id`, run `ano session update --summary "..."` after
**meaningful** events. Keep the summary 1-2 sentences. Examples of when to update:

- After a successful commit (`git commit` returned 0): summarize the commit.
- After `gh pr create` succeeded: include the PR number/URL.
- After a notable test pass (CI green, big suite passing locally).
- After landing a significant refactor.

Do NOT update on every tiny edit — the noise dilutes the signal. One update
per "thing the user would mention in standup" is the right cadence.

The CLI uses a per-cwd cache for the `session_id`, so you don't need to pass
`--session-id` explicitly — `ano session update --summary "..."` is enough.

## Ending a session

Before your final user-visible turn (the one with the wrap-up summary), run:

```
ano session end --status done --summary "<1-2 sentence final summary>"
```

If something went wrong and you're stopping mid-task, use `--status failed`
and explain in the summary. If the user asked to pause, use `--status paused`.

## What the skill does NOT do

- It does not check the opt-in state itself. The CLI handles all gating —
  just trust the "session_id on stdout?" signal.
- It does not retry on errors. If `ano session start` fails (network blip,
  workspace doesn't have Agent Status enabled, etc.), STDERR will explain;
  the skill abandons further calls for the session, same as the unset case.
- It does not call any other `ano …` subcommands. Use the `ano-cli` skill
  for those.

## Worked example (verbatim)

User: "Help me refactor the auth middleware on `feature/auth-mw`."

Assistant runs (silently, before responding):

```
$ ano session start --title "Refactor auth middleware"
session_id=8c2d4e1a-b6f3-4a59-8d2e-3f4c8e7a1b9f
```

Captures `8c2d4e1a-…`. Proceeds with the work.

After landing the first commit:

```
$ ano session update --summary "Extracted the JWT verification path into auth/verify.ts; tests passing."
```

After opening a PR:

```
$ ano session update --summary "Opened PR #1234 — request review from @leo."
```

At the end of the session, before the wrap-up message:

```
$ ano session end --status done --summary "Refactor merged and deployed to staging. PR #1234."
```

## Worked example — opted-out user (one-shot)

User: "Help me debug the flaky test."

Assistant runs:

```
$ ano session start --title "Debug flaky test"
$
```

(stdout empty; stderr printed the discovery message to the user's terminal.)
The assistant does NOT call `ano session update` or `ano session end` for the
rest of the conversation. Multiple commits happen — none trigger CLI calls.
Total CLI invocations for the session: 1.
