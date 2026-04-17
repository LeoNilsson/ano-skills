# @ano-chat/skills

Claude Code skills for [Ano](https://ano.dev) — team communication for humans and agents.

## Install

```bash
claude plugin install @ano-chat/skills
```

## What's included

### ano-cli

Teaches Claude Code how to use the [`ano` CLI](https://www.npmjs.com/package/@ano-chat/cli): read and send messages, manage channels, DMs, tables, search, and more.

### ano-payloads

Teaches Claude Code how to parse `<ano_payload>` XML blocks — structured messages sent from the Ano desktop app via the "Send to Shell" gesture.

## Related

- **[`@ano-chat/cli`](https://www.npmjs.com/package/@ano-chat/cli)** — the `ano` binary these skills teach Claude Code to call. Install with `npm install -g @ano-chat/cli`.

## Releasing a new version

The version appears in **three** files and all three must be kept in sync. The Claude Code plugin manager reads `.claude-plugin/marketplace.json` — if that one lags, `claude plugin update` reports "already at the latest version" even when npm has a newer release.

1. `package.json` → `version`
2. `.claude-plugin/marketplace.json` → `metadata.version` **and** `plugins[0].version`
3. Commit all three together, tag `vX.Y.Z`, and push

Users then pick it up with `claude plugin marketplace update ano-skills && claude plugin update ano-skills@ano-skills`.
