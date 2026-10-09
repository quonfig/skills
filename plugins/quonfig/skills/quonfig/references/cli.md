# `qfg` command reference (app-developer loop)

Install: `npm install -g @quonfig/cli`. Every command takes `--json` for
structured output; prompts turn off automatically when stdin is not a TTY, so
pass every required flag. `qfg <command> --help` is authoritative.

## Auth and workspace

```bash
qfg login                          # browser device sign-in, once per machine
qfg whoami                         # who am I, which orgs
qfg workspace                      # active workspace + everything reachable
qfg workspace switch acme/production
qfg list -w acme/production        # -w <org>/<workspace> on any API command
```

CI: set `QUONFIG_API_KEY` **and** `QUONFIG_WORKSPACE=<org>/<workspace>`.

## Look before you create

```bash
qfg list --feature-flags --json    # what exists (also --configs, --log-levels, --segments)
qfg info checkout.new-flow         # every environment's rules + 24h evaluations
qfg get checkout.timeout --environment production   # evaluated value in one env
```

## Create

```bash
qfg create checkout.new-flow --type boolean-flag --value=false
qfg create checkout.banner-text --type string --value="Free shipping today"
qfg create checkout.timeout --type int --value=30
qfg create checkout.retry-window --type duration --value PT90S
qfg create log-level.checkout-api --type log_level --value WARN
```

- `--type`: `boolean-flag | boolean | string | int | double | string-list | json | duration | log_level`.
  Use `boolean-flag` for a feature flag (it is also sent to client SDKs);
  `boolean` is a plain config.
- `create` sets the default for all environments; there is no `--environment`.
- If the key already exists the command fails ("... already exists"). Do not
  retry with a tweaked key — use the existing one or agree a new name with a
  human.

## Change what an environment serves

```bash
qfg set-default checkout.new-flow --value=true --environment=development
qfg set-rollout checkout.new-flow --environment=staging --true-percent 20 --confirm
qfg set-rollout checkout.layout --environment=production --weights "control:50,treatment:50" --confirm
qfg set-default checkout.new-flow --value=false --environment=production --replace-targeting  # kill switch: off for everyone
```

`--environment` is required when not interactive. These keep targeting rules
unless you pass `--replace-targeting` — see [writes.md](writes.md).

## Log levels

```bash
qfg log-level log-level.checkout-api --value=WARN
qfg log-level log-level.checkout-api --target=Checkout.Payments --value=DEBUG --environment=staging
```

## SDK keys

```bash
qfg sdk-key create --environment production --type server    # backend: QUONFIG_BACKEND_SDK_KEY
qfg sdk-key create --environment development --type browser  # frontend SDKs
qfg sdk-key list
```

Never commit a key; put it in the app's secret store / `.env` (gitignored).

## Typed clients (TypeScript only)

```bash
qfg generate --targets node-ts -o generated     # @quonfig/node
qfg generate --targets react-ts -o generated    # @quonfig/react (default target)
```

## Run a process with config injected as env vars

```bash
qfg run --env DATABASE_URL=db.url --environment=staging -- npm run migrate
```

## Personal override (only you see it)

```bash
qfg override checkout.new-flow true --env=development
qfg override checkout.new-flow --remove
```

Fires only for SDK clients that send the `quonfig-user.email` context.

## Workspace repo (editing JSON directly)

```bash
qfg pull --dir ./quonfig-workspace     # clone/refresh the workspace repo
qfg verify ./quonfig-workspace         # validate (local, no login)
qfg push --dir ./quonfig-workspace     # publish committed edits
qfg config-schema                      # operators, field names, value types
```

Inside a workspace repo, follow that repo's `AGENTS.md` / `CLAUDE.md`.
