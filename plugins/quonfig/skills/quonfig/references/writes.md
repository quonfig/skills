## `qfg` or the Quonfig MCP?

- **A repo is checked out and you have a shell** -> use `qfg`. It is scriptable,
  prints JSON with `--json`, and works next to the code you are changing.
- **Slack, claude.ai, or anywhere without a checkout** -> use the Quonfig MCP
  server (`https://mcp.quonfig.com/mcp`). It reads and writes the hosted
  workspace directly. Do not try to shell out to `qfg` from a surface that has
  no repo.

Both surfaces cover the same everyday loop — create a flag or config, change
what an environment serves, set a log level — and both land as a git commit in
the workspace repo.

## Setting a value KEEPS that environment's targeting rules

Both surfaces write the same way, so learn it once:

`qfg set-default` (alias `toggle`), `qfg set-rollout`, and the MCP
`set_flag`/`set_config` verbs replace exactly ONE rule — the environment's
**fallback**, the unconditional rule at the end of its rule list that decides
what users receive when no targeting rule matches. Every targeting rule above
it is kept, and both surfaces report how many. An environment with no rules of
its own has them copied from default first, so inherited targeting is kept too.

| Surface | By default | To set the value for EVERYONE |
|---------|------------|-------------------------------|
| `qfg set-default` / `qfg set-rollout` | Keeps targeting; reports "Kept N targeting rule(s)" | add `--replace-targeting` |
| MCP `set_flag` / `set_config` | Keeps targeting; reports the kept count in its result | send `replaceTargeting: true` |

Setting the value for everyone DELETES that environment's targeting rules. They
stay in git history, and the write returns a `previousCommitSha` to restore from.
Show the rules to a human and get agreement before you send it.

**Still read before you write.** `qfg info <key>` (or MCP `get_flag`) shows the
rules your value is about to sit next to. For anything a fallback cannot express —
multi-rule targeting, reordering, editing one targeting rule — edit the JSON
in the workspace repo or use MCP `set_document`.

`qfg log-level --target ...` and MCP `set_log_level` are **surgical** in the same
way: they upsert the level for the loggers you name and leave every other rule in
place.
