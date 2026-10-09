# Quonfig MCP server

`https://mcp.quonfig.com/mcp` — streamable HTTP. Use it where there is no repo
checkout or shell (Slack via Claude Tag, claude.ai), or to answer "what is
production serving right now?" from inside a coding session. The plugin
install registers it as `quonfig`; elsewhere add it with
`claude mcp add --transport http quonfig https://mcp.quonfig.com/mcp`.
Developers sign in through the browser (OAuth); shared bots use a `qf_sa_`
service-account key. Either way the caller can only do what its Quonfig roles
allow.

Fourteen read tools and six write tools. Full list:
https://docs.quonfig.com/docs/api/mcp-server#tools

## Answering questions

- Current state of one flag: `get_flag` (returns per-environment rules,
  rollout percentages, and variants). Do not infer state from `list_flags`
  alone.
- A rule with the `IN_SEG` operator names a segment, it does not list its
  members. Call `get_segment` before claiming who receives the flag.
- "Who changed this, and when": `get_flag_history` / `get_config_history`.
  For workspace-wide questions use `get_recent_changes`. For "when did this
  last change" across many items, read `lastModified` off the `list_flags` /
  `list_configs` rows instead of calling a history tool per item.
- Always name the environment in your answer. A flag is usually on in
  development and off in production; an answer that omits the environment is
  wrong more often than right.

## Changing a flag or config

- `set_flag` and `set_config` are the everyday writes, and each sets the
  scope's FALLBACK: the unconditional rule at the end of its rule list, what
  users receive when no targeting rule matches. Targeting rules above it are
  kept — say so, naming the `preservedTargetingRuleCount` the write returns.
  A scope with no rules of its own is copied from the default rules first, so
  inherited targeting is kept too.
- Read the item first, then say plainly what will change and in which scope
  before you call it.
- `replaceTargeting: true` is a different request: it sets the value for
  everyone, deleting the scope's targeting rules. Show the rules and ask a
  human; send it only after someone agrees — neither tool can put those rules
  back afterwards. Report the returned `previousCommitSha` and
  `replacedTargetingRuleCount`.
- Never guess an environment name. Call `list_environments`. A scope is
  either one of those names or the literal `"default"` — the rules every
  environment without its own entry inherits, which for most configs is the
  only place a value is stored. Naming an environment SHADOWS the default
  there; if the request is ambiguous ("turn it off"), ask which was meant.

## Log levels

- `set_log_level` is SURGICAL, unlike the two above: it adds or overwrites
  exactly one rule and leaves every sibling in place, so it never needs a
  confirmation to avoid destroying targeting.
- `key` is the SERVICE (`log-level.api-delivery`) — there is one document
  per service. Pass `target` (a logger path prefix) to change one logger, or
  omit it to set the scope's fallback level. `list_log_levels` shows what is
  stored.

## Creating

- `create_flag` and `create_config` only ADD. A new flag is created OFF in
  every environment — enabling it is a separate `set_flag` call. A new
  config needs an explicit `valueType`.
- `ALREADY_EXISTS` means the key is taken; read `collidingType` to see by
  what. NEVER retry a create with a mutated key — agree a different key
  with a human instead.

## Raw documents and undo

- `get_document` / `set_document` read and replace the stored JSON for a
  flag, config, or log level (`type`: `flag`, `config`, or `log-level` —
  note the hyphen; `get_recent_changes` spells the same kind `log_level`).
  Use them for edits the write verbs cannot express — multi-rule targeting,
  variants, metadata — and to undo any write.
- `set_document` is a FULL REPLACEMENT: a field you omit is deleted. Always
  start from a `get_document` result and edit it; never build the document
  by hand.
- To undo: `get_document` with `at` set to the bad write's
  `previousCommitSha`, then `get_document` again without `at` for the
  current `commitSha`, then `set_document` the old document pinned to that
  CURRENT sha. Say what you are restoring before you write it.
- A `STALE_COMMIT_SHA` failure means someone else changed the item. Re-read
  and re-apply — never retry the same `set_document` arguments.

## More than one workspace

With several workspaces, the first call comes back asking which one and
listing the options; retry with the workspace the human names.
