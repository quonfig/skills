# Quonfig agent skills

Teach your coding agent (Claude Code, Codex, Cursor, ...) to use
[Quonfig](https://quonfig.com) feature flags, dynamic config, and log levels
correctly in your application code: pick and wire up the right SDK, create
flags with the `qfg` CLI, gate code with the right context, roll out per
environment, and retire flags when they are done.

## Install

**Claude Code** (plugin: the skill plus the Quonfig MCP server):

```bash
claude plugin marketplace add quonfig/skills
claude plugin install quonfig@quonfig
```

**Already using the `qfg` CLI** (writes `.claude/skills/quonfig/` into this repo, no network):

```bash
qfg agent install-skill --add-pointer
```

**Any agent via [skills.sh](https://skills.sh):**

```bash
npx skills add quonfig/skills --skill quonfig
```

**Manually:** copy `plugins/quonfig/skills/quonfig/` into your agent's skills
directory (for Claude Code, `.claude/skills/quonfig/`).

Docs: https://docs.quonfig.com/docs/tools/agent-skill

## Layout

```
.claude-plugin/marketplace.json        # marketplace "quonfig"
plugins/quonfig/
  .claude-plugin/plugin.json           # plugin "quonfig" (versioned)
  .mcp.json                            # https://mcp.quonfig.com/mcp
  skills/quonfig/SKILL.md              # the skill
  skills/quonfig/references/*.md       # per-SDK and per-surface detail
```

## Releasing a change

Bump `version` in `plugins/quonfig/.claude-plugin/plugin.json` on every content
change. The `qfg` CLI vendors this skill at build time: after a change lands
here, run `node scripts/sync-agent-skill.mjs` in the cli repo and release it.

`references/writes.md` is also embedded verbatim in the `AGENTS.md` that
`qfg init` writes into workspace repos, so edit it as text that must make
sense in both places.

## License

MIT
