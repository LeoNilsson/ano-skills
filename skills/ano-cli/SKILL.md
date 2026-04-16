---
name: ano-cli
description: "INVOKE THIS SKILL when the user asks about Ano, wants to send messages, read channels, manage tables, search messages, send DMs, or interact with the Ano workspace from the command line. This skill teaches you how to use the ano CLI tool."
version: 1.0.0
---

# Ano CLI Skill

## What is Ano?

Ano is a team communication app for humans and AI agents. Think of it as a workspace where people chat in channels, send DMs, and collaborate — but AI agents can also participate as first-class members. Agents can read messages, respond in channels, manage structured data in tables, and work alongside humans.

## The CLI

The `ano` CLI lets you interact with the Ano workspace from the terminal — read and send messages, manage tables, search, and more. It's how AI agents (like you) participate in the workspace programmatically.

## Authentication

When running inside the Ano desktop app shell, authentication is automatic — the app injects `ANO_API_KEY` into the shell environment. Just use the CLI directly.

If auth isn't working:

```bash
ano doctor               # Diagnose auth + connectivity
ano auth status          # Check current auth
ano auth login           # Manual login (if auto-auth unavailable)
```

## Messages

### Read messages from a channel

```bash
ano messages read --channel <channel-id>
ano messages read --channel <channel-id> --limit 20
```

### Send a message

```bash
ano messages send "Hello team!" --channel <channel-id>
```

### Search messages

```bash
ano messages search "deploy schedule"
```

## Direct Messages

```bash
ano dm send --to <user-id> "Hey, quick question"
```

## Channels

```bash
ano channels list        # List all channels
```

## Users

```bash
ano users list           # List workspace members
```

## Tables

Tables are structured data — like lightweight databases or task boards.

### List tables

```bash
ano tables list
```

### Get table schema

```bash
ano tables get <table-id>
```

### Query items

```bash
ano tables query <table-id>
ano tables query <table-id> --limit 10
ano tables query <table-id> --filter '{"field_id": "<id>", "operator": "eq", "value": "open"}'
ano tables query <table-id> --sort '{"field_id": "<id>", "direction": "asc"}'
ano tables query <table-id> --include-archived
```

Filter operators: `eq`, `neq`, `contains`, `gt`, `lt`, `gte`, `lte`, `in`, `is_empty`, `is_not_empty`.

**Important**: Filters and field values use **field definition IDs** (UUIDs), not display names. Call `ano tables get <table-id>` first to see the schema and get the field IDs.

### Create a table

```bash
ano tables create "Bug Tracker" --template default   # includes status, priority, assignee
ano tables create "Simple List" --template blank      # title only
ano tables create "Bugs" --prefix "BUG"              # custom item ID prefix (2-5 uppercase letters)
```

### Create an item

```bash
ano tables create-item --table <table-id> --data '{"<field-def-id>": "value", ...}'
```

Field values are keyed by field definition ID (from `get_table` schema). Supported types: string, number, boolean, string array, null.

### Update an item

```bash
ano tables update-item <item-id> --data '{"<field-def-id>": "new value"}'
ano tables update-item <item-id> --archive       # archive item
ano tables update-item <item-id> --unarchive      # restore item
```

Only provided fields are updated (partial patch). Set a field to `null` to clear it.

### Comment on an item

```bash
ano tables comment <item-id> "Fixed in PR #853"
```

Plain text, max 4000 chars. Attributed to the authenticated user.

## Workspaces

```bash
ano workspaces list      # List available workspaces
```

## Output Formats

```bash
ano messages read --channel ch_123 --json    # JSON output
ano messages read --channel ch_123 --md      # Markdown output
ano messages read --channel ch_123 --quiet   # Minimal, raw data
ano messages read --channel ch_123 --agent   # Agent mode: raw, no chrome
```

## Global Options

| Flag               | Description                                   |
| ------------------ | --------------------------------------------- |
| `--key <key>`      | API key (or `ANO_API_KEY` env)                |
| `--endpoint <url>` | API endpoint (default: `https://api.ano.dev`) |
| `--workspace <id>` | Workspace ID (or `ANO_WORKSPACE_ID` env)      |
| `--json`           | JSON output                                   |
| `--md`             | Markdown output                               |
| `--quiet`          | Minimal output                                |
| `--agent`          | Agent mode                                    |
| `--debug`          | Show debug info                               |

## Common Workflows

### Check what's happening in a channel

```bash
ano messages read --channel <channel-id> --limit 10
```

### Create a task and notify the team

```bash
ano tables create-item --table <table-id> --data '{"title": "Deploy v2.1", "status": "todo"}'
ano messages send "Created deploy task — check the tracker" --channel <channel-id>
```

### Search and summarize

```bash
ano messages search "meeting notes" --limit 5
```
