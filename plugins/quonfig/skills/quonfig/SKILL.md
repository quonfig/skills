---
name: quonfig
description: Use Quonfig feature flags, dynamic config, and log levels correctly from an application codebase. Covers picking and wiring up the right Quonfig SDK (Node, browser JS, React, React Native, Go, Python, Ruby, Java, .NET, Swift), creating flags and configs with the qfg CLI, gating code with the SDK and the right context, rolling out per environment, typed clients, and retiring flags — plus the Quonfig MCP tools when there is no checkout. Use this skill whenever the user wants to add, check, roll out, toggle, read, or remove a feature flag or remote config value, put code behind a flag, add a kill switch, change a log level at runtime, or mentions Quonfig, qfg, @quonfig/*, QUONFIG_BACKEND_SDK_KEY, or a flag key in this repo.
---

# Quonfig

Quonfig stores feature flags, configs, segments, and log levels as JSON files in
a git repo (the **workspace**). Your application reads them through a Quonfig
SDK; people and agents change them with the `qfg` CLI, the Quonfig MCP server,
or the web app. Every change is a git commit.

This skill is for an **application repo** (code that *reads* flags). If the
current repo is itself a workspace (it has `quonfig.json` and `feature-flags/`,
`configs/` directories), follow that repo's `AGENTS.md`/`CLAUDE.md` instead:
edit the JSON there and run `qfg verify`.

## 1. Orient before writing code

- **Find the SDK.** Look for an existing client before adding one: grep for
  `@quonfig/`, `quonfig` in `go.mod` / `Gemfile` / `pyproject.toml` /
  `requirements.txt` / `pom.xml` / `build.gradle*` / `*.csproj` /
  `Package.swift`, and for where the client is constructed. Reuse it — one
  client per process.
- **Read the reference for this language** (only the one you need):

  | Stack | Reference |
  |-------|-----------|
  | Node.js server (`@quonfig/node`) | [references/sdk-node.md](references/sdk-node.md) |
  | Browser JS, React, React Native | [references/sdk-browser.md](references/sdk-browser.md) |
  | Go | [references/sdk-go.md](references/sdk-go.md) |
  | Python | [references/sdk-python.md](references/sdk-python.md) |
  | Ruby | [references/sdk-ruby.md](references/sdk-ruby.md) |
  | Java / Kotlin | [references/sdk-java.md](references/sdk-java.md) |
  | .NET | [references/sdk-dotnet.md](references/sdk-dotnet.md) |
  | Swift (iOS/macOS) | [references/sdk-swift.md](references/sdk-swift.md) |

- **Pick the surface for changing flags.** You have a shell and a checkout, so
  use `qfg` (check with `qfg whoami`; install with `npm install -g @quonfig/cli`,
  sign in with `qfg login` — the human does the browser step). Use the MCP
  tools only when `qfg` is unavailable or for a quick read of live state.
  Details: [references/writes.md](references/writes.md).

## 2. Adding a feature flag (the core loop)

1. **Name it.** Keys are lowercase, dot-separated, grouped by area:
   `checkout.new-flow`, `search.semantic-ranking`. Check it isn't taken and
   isn't already what you need: `qfg list --feature-flags --json`.
2. **Create it, off.** `qfg create checkout.new-flow --type boolean-flag --value=false`.
   If it already exists, stop and ask — never invent a variant of the key.
3. **Gate the code** with the SDK's flag call and a context that identifies
   who is asking (at minimum `user.key`, so rollouts and targeting work):
   - Node: `quonfig.isEnabled("checkout.new-flow", { user: { key: user.id } })`
   - Go: `on, _ := client.FeatureIsOn("checkout.new-flow", ctx)`
   - Python: `client.is_feature_enabled("checkout.new-flow", contexts={"user": {"key": user.id}})`
   - others: see the language reference.
   Keep the off path identical to today's behavior, so the flag can ship dark.
4. **Turn it on where it is safe** — normally development first:
   `qfg set-default checkout.new-flow --value=true --environment=development --confirm`.
   Always name the environment. (`--confirm` skips the interactive prompt —
   without a terminal, `set-default` and `set-rollout` refuse to run without it.) Do not change production (or roll out there)
   unless the human asked for it.
5. **Tell the human** the key, which environments serve what, and the rollout
   command for later (`qfg set-rollout checkout.new-flow --environment=production --true-percent 10 --confirm`).

## 3. Configs and log levels

- A dynamic value instead of a constant: `qfg create checkout.timeout --type int --value=30`,
  then read it with the SDK's typed getter (see the language reference).
  Types: `string`, `int`, `double`, `boolean`, `string-list`, `json`, `duration`.
- Create a key **before** shipping code that reads it — most server SDKs throw
  on a missing key with no default.
- Runtime log levels: one `log-level.<service>` key per service, wired into
  the app's logger via the SDK's `loggerKey`; change one logger with
  `qfg log-level log-level.checkout-api --target=Checkout.Payments --value=DEBUG --environment=staging`.
- TypeScript apps can generate a typed client: `qfg generate --targets node-ts`
  (server) or `--targets react-ts` (React).

## 4. Rules that prevent the classic mistakes

- **Name the environment** in every write and every answer about state. A flag
  is usually on in development and off in production.
- **Setting a value keeps targeting rules.** `set-default`/`set-rollout` change
  only the fallback rule; `--replace-targeting` deletes the environment's
  targeting and needs a human's agreement first. Read with `qfg info <key>`
  before you write. See [references/writes.md](references/writes.md).
- **Backend vs frontend keys.** Server SDKs use `QUONFIG_BACKEND_SDK_KEY` and
  evaluate locally; browser/mobile SDKs use a frontend key and get values
  evaluated for one context. Never ship a backend key to a browser or app.
  Never commit any key.
- **The SDK key decides the environment** in server SDKs. Set
  `QUONFIG_ENVIRONMENT` / `environment` only for local datadir mode.

## 5. More

- Full `qfg` cheat sheet (create types, rollouts, SDK keys, overrides, `qfg run`):
  [references/cli.md](references/cli.md)
- Quonfig MCP tools (Slack, claude.ai, no checkout; undo with `set_document`):
  [references/mcp.md](references/mcp.md)
- Removing a flag once its rollout is done (`qfg cleanup`):
  [references/retire-a-flag.md](references/retire-a-flag.md)
- Docs: https://docs.quonfig.com
