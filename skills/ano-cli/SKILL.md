---
name: ano-cli
description: |
  CLI for Ano team communication. Channels, messages, DMs, users, workspaces,
  tables, search, real-time streaming, and agent setup. Use for ANY Ano action.
triggers:
  - send a message
  - send message to
  - post in channel
  - reply in thread
  - read messages
  - read channel
  - search messages
  - find messages
  - send dm
  - send direct message
  - list channels
  - show channels
  - list users
  - list members
  - list workspaces
  - list tables
  - show tables
  - query table
  - read table
  - create table
  - add row
  - add table item
  - update row
  - update table item
  - comment on table item
  - list automations
  - create automation
  - new automation
  - edit automation
  - pause automation
  - resume automation
  - delete automation
  - run automation
  - automation runs
  - webhook setup
  - ano login
  - ano auth
  - ano connect
  - ano setup
  - ano doctor
  - ano show
  - check ano
  - notify team
  - notify channel
  - update the team
  - post an update
  - ano.dev
  - api.ano.dev
  - ano_cwk_
invocable: true
argument-hint: "[command] [args...]"
---

# Ano CLI — Agent Skill

CLI for Ano team communication. Read/send messages, list channels and members,
read and write tables, build automations, search conversations, stream
real-time events, and manage agent integrations.

**The CLI is the canonical surface for Claude Code.** When a user asks you to
do something in Ano, prefer `ano <command>` over MCP tool calls. MCP is a
fallback for environments without the CLI installed.

## Agent Invariants

1. **Always use `--agent` or `--json` output.** Never parse styled TTY output.
   Use `--agent` for raw JSON; `--json` for envelope with breadcrumbs.
2. **Never fabricate IDs.** Channel/user/message IDs are UUIDs. Get them from
   `ano channels list --agent` or `ano users list --agent` first.
3. **Resolve by name before acting.** "Post in #general" → list channels, find
   the ID, then send. "DM Leo" → list users, find the ID, then send.
4. **Respect rate limits.** 60 requests/minute. Exit code 5 = rate limited.
   Wait 10+ seconds before retrying.
5. **Check exit codes.** Non-zero = failure. Parse the error envelope.
6. **Never expose API keys.** Don't log or include `ano_cwk_*` or `ano_usr_*` keys in output.
7. **Content supports markdown.** Bold, links, code blocks, lists all work.
8. **Reply in threads** to keep channels clean. Use `--thread <message_id>`.
9. **Follow breadcrumbs.** JSON responses include suggested next commands.
10. **Run `ano doctor`** before escalating connectivity issues.
11. **Fetch a table's schema before writing to it.** `ano tables get <table-id> --agent`
    returns the field-definition IDs. `create-item` / `update-item` require the
    `--fields` JSON to be keyed by field-definition ID, not by the human-readable
    field name.
12. **On exit code 3 (AUTH), run the Triggered Auth flow before surfacing the error.**
    Don't dump "run `ano auth login`" on the user as a manual step — orchestrate it
    inline using `--print-workspaces` + `auth complete`. See "Triggered Auth" below.

## Output Modes

| Flag      | Format                                    | When to use               |
| --------- | ----------------------------------------- | ------------------------- |
| `--agent` | Raw JSON (one object per line)            | Default for agents        |
| `--json`  | Envelope: `{ok, data, breadcrumbs, meta}` | When you need breadcrumbs |
| `--md`    | GFM markdown tables                       | Presenting to humans      |
| `--quiet` | Same as `--agent`                         | Scripting                 |
| (none)    | Styled with colors                        | Interactive TTY           |

### JSON envelope (`--json`)

```json
{
  "ok": true,
  "data": [...],
  "breadcrumbs": [
    {"action": "read_messages", "cmd": "ano messages read --channel <id>", "description": "Read messages"}
  ],
  "meta": {"timestamp": "...", "version": "2.4.0"}
}
```

### Error output

```json
{
  "ok": false,
  "error": "Invalid or expired API key",
  "code": 3,
  "hint": "Run \"ano auth login\" or pass --key"
}
```

## CLI Introspection

```bash
ano channels list --help --agent    # Structured JSON for one command
ano commands --json                 # Full command catalog
```

`--help --agent` returns:

```json
{
  "command": "ano channels list",
  "path": ["ano", "channels", "list"],
  "description": "List channels in the workspace",
  "args": [],
  "flags": [],
  "subcommands": []
}
```

## Quick Reference

| Task                                    | Command                                                                      |
| --------------------------------------- | ---------------------------------------------------------------------------- |
| **Auth**                                |                                                                              |
| Save API key                            | `ano auth login --key <key>`                                                 |
| Browser login                           | `ano auth login --workspace-id <id>` (requires CLI v2.1.0+)                  |
| Triggered auth (orchestrators) — step 1 | `ano auth login --print-workspaces` (CLI v2.2.0+)                            |
| Triggered auth — step 2                 | `ano auth complete --workspace-id <id>` (CLI v2.2.0+)                        |
| Check auth                              | `ano auth status --agent`                                                    |
| Remove credentials                      | `ano auth logout`                                                            |
| **Read**                                |                                                                              |
| List channels                           | `ano channels list --agent`                                                  |
| List users                              | `ano users list --agent`                                                     |
| List workspaces                         | `ano workspaces list --agent`                                                |
| Read messages                           | `ano messages read --channel <id> --agent`                                   |
| Read (limited)                          | `ano messages read --channel <id> --limit 10 --agent`                        |
| Search messages                         | `ano messages search "query" --agent`                                        |
| Search (limited)                        | `ano messages search "query" --limit 5 --agent`                              |
| Show URL content                        | `ano show <url> --agent`                                                     |
| **Write**                               |                                                                              |
| Send message                            | `ano messages send "text" --channel <id> --agent`                            |
| Reply in thread                         | `ano messages send "text" --channel <id> --thread <msg_id> --agent`          |
| Send with @mention                      | `ano messages send "text" --channel <id> --mention <user_id> --agent`        |
| Send DM (by name)                       | `ano dm send "text" --to "Name" --agent`                                     |
| Send DM (by email)                      | `ano dm send "text" --email user@co.com --agent`                             |
| Send DM (by ID)                         | `ano dm send "text" --user-id <id> --agent`                                  |
| **Tables**                              |                                                                              |
| List tables                             | `ano tables list --agent`                                                    |
| Get table + schema                      | `ano tables get <table-id> --agent`                                          |
| Query items                             | `ano tables query <table-id> --agent`                                        |
| Query (filtered)                        | `ano tables query <table-id> --filter '<json-array>' --agent`                |
| Query (sorted)                          | `ano tables query <table-id> --sort '<json>' --agent`                        |
| Create table                            | `ano tables create "<name>" --agent`                                         |
| Create item                             | `ano tables create-item --table <id> --fields '<json>' --agent`              |
| Update item                             | `ano tables update-item <item-id> --fields '<json>' --agent`                 |
| Archive item                            | `ano tables update-item <item-id> --archive --agent`                         |
| Comment on item                         | `ano tables comment <item-id> "body" --agent`                                |
| **Automations (guided)**                |                                                                              |
| New automation (Claude UX)              | `ano new automation "every weekday 9am post yesterday's signups to #growth"` |
| Edit automation (Claude UX)             | `ano edit automation <id> "add a Slack notification on failure"`             |
| **Automations (granular)**              |                                                                              |
| Compile (no save)                       | `ano automation compile "prompt" --agent`                                    |
| Create from prompt                      | `ano automation create "prompt" --agent`                                     |
| Create from JSON                        | `ano automation create-compiled --file plan.json --agent`                    |
| Update                                  | `ano automation update <id> --name "..." --agent`                            |
| List                                    | `ano automation list --agent`                                                |
| Recent runs                             | `ano automation runs <id> --agent`                                           |
| Test (dry-run)                          | `ano automation run <id> --agent`                                            |
| Run for real                            | `ano automation run <id> --no-dry-run --agent`                               |
| Pause                                   | `ano automation pause <id>`                                                  |
| Resume                                  | `ano automation resume <id>`                                                 |
| Delete                                  | `ano automation delete <id>`                                                 |
| Webhook setup/rotate                    | `ano automation webhook-setup <id> --agent`                                  |
| **Real-time**                           |                                                                              |
| Start SSE bridge                        | `ano connect`                                                                |
| Bridge + agent mode                     | `ano connect --openclaw <url>`                                               |
| Bridge + health                         | `ano connect --health-port 8080`                                             |
| Install service                         | `ano connect install-service`                                                |
| Remove service                          | `ano connect uninstall-service --service-name <name-or-hash>`                |
| **Diagnostics**                         |                                                                              |
| Full diagnostics                        | `ano doctor --agent`                                                         |
| Command catalog                         | `ano commands --json`                                                        |
| Setup Claude                            | `ano setup claude`                                                           |
| Setup OpenClaw                          | `ano setup openclaw`                                                         |

## Triggered Auth (when CLI is unauthenticated)

If `ano <command> --agent` returns exit code **3 (AUTH)** — DO NOT dump the
error and the `ano auth login` hint on the user. Run the triggered-auth flow
inline. The user gets ONE browser click + one in-chat workspace pick, instead
of a context switch and a manual setup detour.

**Requires CLI v2.2.0+.** If the installed CLI is older, fall back to telling
the user to upgrade (`npm install -g @ano-chat/cli@latest --force`) before
retrying.

### Detect the installed version first

Before invoking the new flags, run `ano --version`. The output is a single
semver line (e.g. `2.2.0`). Compare:

- Major < 2 OR (major == 2 AND minor < 2) → CLI is too old. Tell the user
  to upgrade: `npm install -g @ano-chat/cli@latest --force`. Don't try to
  invoke the new flags — they don't exist and the CLI errors with
  "unknown option."
- Major == 2 AND minor >= 2 → triggered auth is supported.
- Major >= 3 → assume forward-compatibility unless you've seen a breaking
  change in the changelog.

If the CLI binary is missing entirely (`ano: command not found`), tell
the user to install it first: `npm install -g @ano-chat/cli`.

### Pick the right --endpoint

The `--print-workspaces` and `auth complete` commands accept `--endpoint`.
Choose:

- If the user mentions **staging**, **api-staging**, **api-staging.ano.dev**,
  **ano-staging**, or anything indicating a non-prod environment → use
  `--endpoint https://api-staging.ano.dev`.
- Otherwise → omit the flag (CLI defaults to `https://api.ano.dev`, which
  is what real users want).
- When unsure (e.g. the user is a developer and the request is ambiguous
  between envs), ask via AskUserQuestion: "Are you connecting to production
  or staging?"

### Decision tree

```
Got exit code 3 (AUTH) on any command?
├── 1. ano auth login --print-workspaces --endpoint <env>
│      • Browser opens; user clicks Authorize once.
│      • CLI caches the access token to ~/.config/ano/.session (5-min TTL).
│      • CLI prints {"workspaces":[{"id":"...","name":"..."}, ...]} to stdout.
│      • CLI exits without minting a key.
├── 2. Parse the workspaces JSON from stdout.
├── 3. Render an in-chat picker (e.g. AskUserQuestion in Claude Code) with
│      one option per workspace name. Wait for the user's pick.
├── 4. ano auth complete --workspace-id <picked-id>
│      • CLI reads the cached token, mints the key, writes credentials.json,
│        deletes the cached token.
│      • Stdout: {"ok":true,"profile":"default","workspace":{...}}
├── 5. Retry the original command. It should now succeed.
└── 6. If step 4 returns exit code 3 ("expired"), the user took >5 min;
       loop back to step 1 to start a fresh OAuth.
```

### What the orchestrator MUST do

- Pick `--endpoint` per the heuristic in "Pick the right --endpoint" above.
- Treat `--print-workspaces` stdout as the canonical workspace list. Don't
  hard-code IDs.
- Treat `auth complete --workspace-id <id>` stdout as machine-readable
  (it's a single JSON line). Parse `{ok, profile, workspace}`.
- Surface the original task as soon as auth succeeds — don't make the user
  re-issue the request.

### What the orchestrator MUST NOT do

- Don't run `ano auth login` (no flags) — it requires a TTY for the workspace
  picker, which orchestrators don't have. Use the `--print-workspaces` /
  `auth complete` pair instead.
- Don't capture or log the access token or the minted CLI key. Both are
  short-lived/sensitive.
- Don't re-run `--print-workspaces` if `auth complete` fails on a NON-auth
  error (network blip during mint, invalid workspace id) — the cached token
  is still valid for 5 minutes; just retry `auth complete` or ask the user
  to verify the workspace pick. **BUT** if `auth complete` returns exit
  code 3 (AUTH) or its error mentions "expired" / "no cached login session",
  the 5-minute TTL has elapsed and you MUST re-run `--print-workspaces` to
  start a fresh OAuth round before retrying.

### Why two steps

The CLI's interactive workspace picker requires `process.stdin.isTTY`.
Claude Code's bash bridge (and most orchestrators) pipe stdin from the
parent process — not a TTY. The `--print-workspaces` / `auth complete`
pair lets the orchestrator render its OWN picker while the CLI handles
OAuth and key minting.

## Decision Trees

### Finding Content

```
Need to find something?
├── Know the channel? → ano messages read --channel <id> --agent
├── Need to search? → ano messages search "query" --agent
├── Which channels exist? → ano channels list --agent
├── Who's in the workspace? → ano users list --agent
├── Have a URL? → ano show <url> --agent
└── Multiple workspaces? → ano workspaces list --agent
```

### Sending Content

```
Want to send something?
├── To a channel → ano messages send "text" --channel <id> --agent
├── Reply in thread → add --thread <msg_id>
├── With @mention → add --mention <user_id>
└── DM someone → ano dm send "text" --to "Name" --agent
```

### Working with Tables

```
Need structured data (lists, databases, rows)?
├── What tables exist?   → ano tables list --agent
├── Schema + field IDs?  → ano tables get <table-id> --agent
├── Read rows?           → ano tables query <table-id> --agent
├── Filter rows?         → ano tables query <table-id> --filter '[{"field_id":"f1","operator":"eq","value":"done"}]' --agent
├── Add a row?           → ano tables get <table-id> --agent   (first, to learn field IDs)
│                        → ano tables create-item --table <table-id> --fields '{"<field_id>":"..."}' --agent
├── Edit a row?          → ano tables update-item <item-id> --fields '{"<field_id>":"..."}' --agent
├── Archive a row?       → ano tables update-item <item-id> --archive --agent
└── Comment on a row?    → ano tables comment <item-id> "body text" --agent
```

### Working with Automations

```
Building or managing an automation?
├── New automation (recommended) → ano new automation "every weekday 9am ..."
├── Edit an existing one         → ano edit automation <id> "what changed"
├── List existing                → ano automation list --agent
├── See recent runs              → ano automation runs <id> --agent
├── Test before enabling         → ano automation run <id> --agent  (dry-run)
├── Fire for real once           → ano automation run <id> --no-dry-run --agent
├── Pause / resume               → ano automation pause|resume <id>
├── Webhook trigger setup        → ano automation webhook-setup <id> --agent
└── Delete (irreversible)        → ano automation delete <id>
```

The guided `ano new automation` and `ano edit automation` commands are the
preferred path — they're designed to be invoked by Claude Code and walk the
user through the spec collaboratively. Use them unless the user has a
pre-built JSON plan (then `ano automation create-compiled --file plan.json`).

For brand-new automations Claude Code is composing in chat, use the deeper
"Building Automations Offline" reference further down — it lets you
assemble the full spec without compile-iterate roundtrips, then submit
once via `ano automation create-compiled --file -`.

### Setting Up Agent Access

```
├── Have API key? → ano auth login --key <key>
├── No key, need browser-based login (TTY) → ano auth login [--workspace-id <id>]
├── No key, need browser-based login (non-TTY orchestrator)
│   → ano auth login --print-workspaces  (step 1)
│   → ano auth complete --workspace-id <id>  (step 2)
├── Hit exit code 3 mid-task → see "Triggered Auth" above
├── One-off commands → use ano messages/channels/users directly
├── Persistent bridge → ano connect install-service
├── OpenClaw agent → ano connect --openclaw <url>
└── Diagnose issues → ano doctor --agent
```

## Common Workflows

### Read a channel and reply

```bash
channels=$(ano channels list --agent)
# Parse to find channel ID for "general"
messages=$(ano messages read --channel "$CHANNEL_ID" --limit 20 --agent)
ano messages send "Here's my analysis..." --channel "$CHANNEL_ID" --agent
```

### Search, then reply in thread

```bash
results=$(ano messages search "deployment issue" --agent)
# Extract channel_id and message_id from results
ano messages read --channel "$CHANNEL_ID" --limit 50 --agent
ano messages send "Fix applied" --channel "$CHANNEL_ID" --thread "$MSG_ID" --agent
```

### DM with user lookup

```bash
users=$(ano users list --agent)
# Find user ID for "Jane"
ano dm send "Can you review PR #42?" --to "Jane" --agent
```

### Real-time bridge with OpenClaw

```bash
# Start persistent agent bridge
ano connect install-service \
  --key ano_cwk_... \
  --openclaw http://localhost:3000 \
  --health-port 8080

# Verify
curl http://127.0.0.1:8080/healthz
```

### stdin/stdout bridge protocol

Events stream as JSON lines on stdout:

```json
{"type":"connected","workspace":"Acme","channels":5,"members":12}
{"type":"message","channel_id":"...","content":"Hello","sender_name":"Jane"}
{"type":"dm","content":"Hey agent","sender_name":"Bob"}
```

Send commands on stdin:

```json
{"action":"send_message","channel_id":"...","content":"Hello"}
{"action":"send_dm","recipient_name":"Jane","content":"Hey"}
{"action":"typing","channel_id":"..."}
```

## Building Automations Offline (Build-Before-Talk)

**Lead-time rule:** when a user asks you to create an automation, build the
**complete spec** in chat first, then submit it to Ano in a single call. Do
NOT iterate via `automation compile` — that costs an LLM round-trip per
revision. The full spec language is documented below; you have everything
you need to design a valid plan offline.

The submission command is:

```bash
ano automation create-compiled --file - --agent <<'EOF'
{ ...the full plan JSON, schema below... }
EOF
```

It returns `{automation_id, name, enabled: false, ...}`. The automation
lands in `unconfirmed` state — the user approves it via the Automations
page or DM in Ano.

### Step 1: Gather workspace context BEFORE designing

You need IDs and names from the user's workspace to fill in the plan. One
batch of reads, then design offline:

```bash
ano channels list --agent      # channel IDs (note `is_public`, `has_guests`)
ano users list --agent         # user IDs + emails for send_dm
ano workspaces list --agent    # if multi-workspace, confirm which one
```

If the user mentions a coworker by name ("post as Maya"), also look at the
coworkers exposed in the workspace context. If the user mentions a SQL or
HTTP integration ("query our prod Postgres"), check whether the connection
exists — if not, you'll still emit the action and Ano will surface a
phantom Connect chip on the Automations page for the user to wire up.

### Step 2: Pick the trigger

There are exactly **five trigger types**. Pick the one that matches and
fill its `trigger_config`:

| `trigger_type`  | `trigger_config` shape                                             | When to use                                                    |
| --------------- | ------------------------------------------------------------------ | -------------------------------------------------------------- |
| `schedule`      | `{ cron: "0 9 * * 1-5", tz: "Europe/Stockholm" }`                  | Time-based (every weekday 9am, hourly, etc.)                   |
| `message_match` | `{ channel_id, pattern, sender_id? }`                              | Fire when a message in a channel matches a regex/string        |
| `mention`       | `{ channel_ids?: string[] }`                                       | Fire when @mentioned (workspace-wide unless channels narrowed) |
| `channel_event` | `{ channel_id, event_type: "reaction_added"\|"member_joined"... }` | Fire on a channel-level event                                  |
| `webhook`       | `{}` (config is empty; URL + signing secret minted after save)     | External system POSTs a payload                                |

**Cron grammar (5-field standard):** `minute hour dom mon dow`

- `minute` 0–59, `hour` 0–23, `dom` 1–31 or `*`, `mon` 1–12 or `*`, `dow` 0–6 or `*` (Sun=0)
- Lists (`1-5`), steps (`*/15`), and ranges (`9-17`) all work
- Always set `tz` explicitly. Default to the workspace timezone (read from `workspace.timezone`); fall back to `"UTC"` if unknown
- Examples: `"0 9 * * 1-5"` = 9am every weekday, `"*/15 * * * *"` = every 15 minutes, `"0 */4 * * *"` = top of every 4th hour

### Step 3: Compose the action chain

Five action tools are available. Use the **exact** tool names below
(verbatim — these are NOT CLI commands, they're runtime tool names the
automation engine resolves):

| `tool`         | `args` shape                                                                  | Output for chaining            |
| -------------- | ----------------------------------------------------------------------------- | ------------------------------ |
| `send_message` | `{ channel_id, content }`                                                     | (none — terminal action)       |
| `send_dm`      | `{ user_id, content }`                                                        | (none — terminal action)       |
| `sql_query`    | `{ connection: "<connection_name>", query: "SELECT ..." }`                    | `{{stepN.rows}}` (array)       |
| `http_request` | `{ connection?: "<name>", method: "GET"\|"POST"\|..., url, headers?, body? }` | `{{stepN.body}}` (parsed JSON) |
| `run_skill`    | `{ skill_id, args }`                                                          | `{{stepN.output}}`             |

**Template variables (chaining):** earlier-step outputs are interpolated
into later-step args using `{{stepN.PATH}}`:

- `{{step1.rows.length}}` — count of SQL rows
- `{{step1.rows.0.name}}` — first row's `name` column
- `{{step2.body.user.email}}` — nested field from an HTTP response

`stepN` is **1-indexed** in the order they appear in `actions[]`.

**Sender attribution** (`sender_kind` at the top level of the plan):

- `bot` (default) — generic Ano bot avatar
- `coworker` — pair with `coworker_id`; messages appear from that coworker
- `human` — posts as the human owner; **the safety lint warns on this**
  because channel members may think the human typed it manually

`bot_avatar` is an optional single emoji (e.g. `"📊"`).

### Step 4: Apply safety rules BEFORE submitting

Pre-check the plan against these rules — they're what Ano's safety lint
will warn on. Fixing them in chat is faster than after-the-fact warnings.

| Rule                                 | What to do                                                                                                                   |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| `SELECT *` going to a public channel | Always add a `LIMIT N` (typically 50). Unbounded `SELECT *` → public channel = channel flood.                                |
| Sensitive columns in SQL             | Avoid `password`, `secret`, `token`, `api_key`, `salt`, `private_key`, `credit_card`, `ssn`. Mask at query time or refuse.   |
| Posting as `human`                   | Default to `bot` unless the user explicitly asks. If `human`, warn the user that channel members will see it as their voice. |
| Public-channel destination           | If posting to a public channel, mention this to the user (they may want a private channel instead).                          |
| Guest-visible channel                | If `has_guests=true` on the destination channel, flag it — guests will see the output.                                       |
| Connection name not found            | Still emit the action; warn the user the connection chip will be phantom until they wire it up.                              |

### Step 5: Interview script

When the user says "create an automation", run this interview in chat
**before** any Ano calls. The goal is to walk away with every field of
the plan filled in:

1. **Trigger:**
   - "When should this fire — on a schedule, when someone messages a channel, when you're @mentioned, on a channel event, or when an external system pings a webhook?"
   - If schedule: "What cadence and at what time / timezone?"
   - If message_match: "Which channel? What pattern should match?"
2. **Actions:**
   - "What should it do? Walk me through the steps in order."
   - For each step, identify which of the 5 tools applies. If the user says "post in Slack" they probably mean `send_message` to an Ano channel.
   - For SQL/HTTP steps, ask which connection.
   - If outputs from earlier steps need to flow in, build the `{{stepN...}}` reference yourself — don't ask the user to write template syntax.
3. **Sender:** "Should this post as a bot, as one of your coworkers (which one?), or as you?"
4. **Name:** propose one based on the prompt; let the user override.
5. **Recap:** read the plan back in plain English. **Don't show JSON** unless asked. Confirm.
6. **Submit:** one shot — `ano automation create-compiled --file - --agent`.

### Worked examples

#### Example 1 — Daily digest from Postgres

User: _"Every weekday at 9am Stockholm time, post the count of new signups from yesterday to #growth."_

```json
{
  "trigger_type": "schedule",
  "trigger_config": { "cron": "0 9 * * 1-5", "tz": "Europe/Stockholm" },
  "actions": [
    {
      "tool": "sql_query",
      "args": {
        "connection": "neon-prod",
        "query": "SELECT COUNT(*) AS n FROM users WHERE created_at::date = CURRENT_DATE - 1 LIMIT 1"
      }
    },
    {
      "tool": "send_message",
      "args": {
        "channel_id": "ch_abc123",
        "content": "📊 New signups yesterday: {{step1.rows.0.n}}"
      }
    }
  ],
  "name": "Daily signup count → #growth",
  "sender_kind": "bot",
  "bot_avatar": "📊"
}
```

#### Example 2 — Mention-triggered triage

User: _"When @mentioned in #support, query our HelpScout API and DM me the open ticket count."_

```json
{
  "trigger_type": "mention",
  "trigger_config": { "channel_ids": ["ch_support_xyz"] },
  "actions": [
    {
      "tool": "http_request",
      "args": {
        "connection": "helpscout",
        "method": "GET",
        "url": "https://api.helpscout.net/v2/conversations?status=active"
      }
    },
    {
      "tool": "send_dm",
      "args": {
        "user_id": "u_user_789",
        "content": "Open tickets: {{step1.body._embedded.conversations.length}}"
      }
    }
  ],
  "name": "Mention in #support → DM ticket count",
  "sender_kind": "bot"
}
```

#### Example 3 — Webhook to channel

User: _"When Stripe webhooks us a charge.failed event, post the customer email to #billing-alerts."_

```json
{
  "trigger_type": "webhook",
  "trigger_config": {},
  "actions": [
    {
      "tool": "send_message",
      "args": {
        "channel_id": "ch_billing_alerts",
        "content": "❌ Charge failed for {{step0.body.data.object.receipt_email}} — amount {{step0.body.data.object.amount}}"
      }
    }
  ],
  "name": "Stripe charge.failed → #billing-alerts",
  "sender_kind": "bot",
  "bot_avatar": "❌"
}
```

(For webhooks, `{{step0.body.*}}` references the inbound payload.)

### Validation checklist before submitting

- [ ] `trigger_type` is one of the five literals
- [ ] `trigger_config` matches the shape for that trigger type
- [ ] If `schedule`: `cron` parses as 5 fields; `tz` is set
- [ ] `actions` has at least one entry
- [ ] Every `tool` is one of the five literals
- [ ] Every `channel_id` / `user_id` / `connection` resolves to something the user named in the interview
- [ ] Every `{{stepN.PATH}}` reference points at a step that produces that output
- [ ] `name` is ≤80 chars and human-readable
- [ ] `sender_kind` is set; `coworker_id` is present iff `sender_kind="coworker"`
- [ ] No `SELECT *` to a public channel without a `LIMIT`
- [ ] No sensitive-column patterns in SQL queries

### After submission

```bash
# Submit the spec
ano automation create-compiled --file - --agent <<EOF
{...}
EOF
# → {"id":"auto_abc","name":"…","enabled":false,...}

# Verify
ano automation list --agent

# Test (dry-run) before enabling
ano automation run auto_abc --agent

# Fire for real once
ano automation run auto_abc --no-dry-run --agent
```

The user enables it in the Ano UI — that's a deliberate human gate so an
automation can never silently start firing.

## Exit Codes

| Code | Name       | Meaning             | Fix                                                          |
| ---- | ---------- | ------------------- | ------------------------------------------------------------ |
| 0    | OK         | Success             | —                                                            |
| 1    | USAGE      | Bad arguments       | `ano <cmd> --help`                                           |
| 2    | NOT_FOUND  | Resource missing    | Verify ID/URL                                                |
| 3    | AUTH       | Invalid/missing key | Run **Triggered Auth** flow (see above) — don't dump on user |
| 4    | FORBIDDEN  | No permission       | Check key scopes                                             |
| 5    | RATE_LIMIT | 60/min exceeded     | Wait 10s, retry                                              |
| 6    | NETWORK    | Connection failed   | `ano doctor --agent`                                         |
| 7    | API_ERROR  | Server error        | Retry                                                        |

## Authentication

Resolution chain (highest priority first):

1. `--key` flag
2. `ANO_API_KEY` environment variable
3. `.ano/config.json` (project-level)
4. `~/.config/ano/credentials.json` (global, via `ano auth login`)

```bash
# Save credentials (programmatic — paste a key)
ano auth login --key ano_cwk_... --endpoint https://api-staging.ano.dev

# Browser login (interactive TTY)
ano auth login --workspace-id <id>

# Browser login (non-TTY orchestrator) — see "Triggered Auth" section
ano auth login --print-workspaces
ano auth complete --workspace-id <id>

# Check
ano auth status --agent

# For non-default endpoints
ano auth login --key ano_cwk_... --endpoint https://api-staging.ano.dev --profile staging
```

## Configuration

```
~/.config/ano/
├── credentials.json    # API keys per profile
├── config.json         # Global defaults
└── .session            # Short-lived OAuth token cache (between
                        # `auth login --print-workspaces` and `auth complete`)

.ano/
└── config.json         # Project-level overrides (workspace_id, endpoint)
```

| Env Variable       | Description                                 |
| ------------------ | ------------------------------------------- |
| `ANO_API_KEY`      | API key                                     |
| `ANO_ENDPOINT`     | API endpoint (default: https://api.ano.dev) |
| `ANO_WORKSPACE_ID` | Default workspace                           |
| `NO_COLOR`         | Disable ANSI colors                         |

## Event Types (SSE Bridge)

| Type              | Trigger         | Key Fields                                 |
| ----------------- | --------------- | ------------------------------------------ |
| `message`         | Channel message | channel_id, content, sender_name, mentions |
| `thread_reply`    | Thread reply    | channel_id, thread_id, content, parent     |
| `dm`              | Direct message  | channel_id, content, sender_name           |
| `reaction`        | Emoji reaction  | message_id, emoji, sender_name             |
| `channel_added`   | Joined channel  | channel_id, user_id                        |
| `channel_removed` | Left channel    | channel_id, user_id                        |

Agent mode (`--openclaw`) auto-responds to DMs, thread replies, and @mentions.

## Rate Limiting

- 60 requests/minute per API key, sliding window
- Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- CLI retries automatically on 429 with backoff
- Batch reads with `--limit` instead of many small requests
- A typical workflow (list channels + read + send) uses 3 of 60 requests

## Learn More

- CLI repo: https://github.com/LeoNilsson/ano-cli
- Ano: https://ano.dev
