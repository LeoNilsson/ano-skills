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

The version appears in **three** files and all three must be kept in sync. The Claude Code plugin manager reads `.claude-plugin/plugin.json` (canonical) and `.claude-plugin/marketplace.json` — if either lags, `claude plugin update` reports "already at the latest version" even when npm has a newer release.

1. `.claude-plugin/plugin.json` → `version` (canonical for Claude Code's update cache)
2. `.claude-plugin/marketplace.json` → `metadata.version` **and** `plugins[0].version`
3. `package.json` → `version` (npm distribution; not used by Claude Code, but keep in sync for human clarity)
4. Commit all three together, tag `vX.Y.Z`, and push

Users then pick it up with `claude plugin marketplace update ano-skills && claude plugin update ano-skills@ano-skills`. There is no min-version mechanism — users on auto-update get it next time their Claude Code refreshes; manual users run `/plugin update`.

### Joint release flow with the Ano CLI

The `@ano-chat/cli` binary (separate repo) and these skills are **versioned independently**, but the skill content describes the CLI surface. They drift if you change one without the other. Treat them as a paired release:

1. **CLI side** (`@ano-chat/cli` repo) — ship the new commands / flags / behaviour. Bump CLI version, `npm publish`. Run `ano commands --json` and confirm the introspection output matches what's documented.
2. **Skills side (this repo)** — update `skills/ano-cli/SKILL.md` to cover the new surface in:
   - `triggers:` frontmatter (so Claude Code matches user intent to the skill)
   - **Quick Reference** table (every new command gets a row)
   - **Decision Trees** (every new entity-group decision tree updated)
   - Worked examples (when relevant)
   - Bump per the three-file rule above. MINOR for new commands/flags; PATCH for doc clarifications; MAJOR for surface removals.
3. **Cross-check:** the Ano monorepo (`server/`) holds a lock test (`server/automation-spec-skill-examples.test.ts`) that validates the SKILL.md "Building Automations Offline" example specs against the compiler schema. If you change the engine's accepted shape **or** rewrite those examples, both repos must update; CI in the monorepo will fail otherwise.
4. **Version skew** is not gated. If a user has CLI v2.3.0 but skill v0.5.0 documents a v2.4.0 flag, Claude Code will tell them to run a command that fails. Mitigate by versioning the skill in lockstep with CLI releases — skill doc may say "(requires CLI vX.Y.Z+)" inline on commands that need it.

If you only update the CLI without bumping the skill, agents using stale skills will give wrong commands. If you only update the skill, agents will reference behaviour that doesn't exist yet.
