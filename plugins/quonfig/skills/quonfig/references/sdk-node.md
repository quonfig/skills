# Node.js — `@quonfig/node`

Server-side SDK. Downloads the whole workspace and evaluates every check
in-process (no network call per evaluation).

## Install

```bash
npm install @quonfig/node
```

## Initialize once, at startup

```typescript
import { Quonfig } from "@quonfig/node";

// sdkKey falls back to process.env.QUONFIG_BACKEND_SDK_KEY when omitted.
const quonfig = new Quonfig({ sdkKey: process.env.QUONFIG_BACKEND_SDK_KEY });
await quonfig.init();
```

- The **backend SDK key decides the environment**. Do not also pin
  `environment` in SDK-key mode (the SDK warns and ignores it).
- Create one client per process and share it (export it from a module). If the
  app already has one, reuse it — grep for `new Quonfig(` first.
- Local / offline mode reads a workspace checkout instead of the network. Here an
  environment is required (`environment` option or `QUONFIG_ENVIRONMENT`), and
  Node does **not** read `QUONFIG_DIR` itself:

```typescript
const quonfig = new Quonfig({
  datadir: "./workspace-data",
  environment: "development",
  dataDirAutoReload: true, // off by default — must be opted in
});
await quonfig.init();
```

## Feature flag

```typescript
if (quonfig.isEnabled("new-dashboard", { user: { key: user.id, email: user.email } })) {
  // new path
}
```

Signature: `isEnabled(key, contexts?)`. `isFeatureEnabled` is a deprecated alias
— write `isEnabled` in new code.

## Config values

```typescript
const limit = quonfig.getNumber("rate-limit");
const regions = quonfig.getStringList("allowed-regions");
const hero = quonfig.get("homepage-hero", { user: { key: "user-123", country: "US" } });
const timeout = quonfig.get("api.timeout", undefined, 30); // only get() takes a default
```

Typed getters: `getString`, `getNumber`, `getBool`, `getStringList`,
`getDuration`, `getJSON` — each `(key, contexts?)`, no default argument. With
the default `onNoDefault: "error"`, reading a key that does not exist (and has
no default) throws, so create the key before shipping code that reads it.

## Context

```typescript
// Bound context for repeated lookups (e.g. per request)
const userClient = quonfig.withContext({ user: { key: "user-123", plan: "pro" } });
userClient.get("feature-x");
userClient.isEnabled("beta-feature");
```

`inContext` is a deprecated alias of `withContext`. The `globalContext`
constructor option applies a context to every evaluation (e.g. service name).

## Typed client (optional)

`qfg generate --targets node-ts -o generated` writes
`generated/quonfig-server.ts`, a typed wrapper with one accessor per key:

```typescript
import { QuonfigTypesafeNode } from "./generated/quonfig-server";

const typed = new QuonfigTypesafeNode(quonfig);
typed.flagSmoke({ user: { key: user.id } }); // flag.smoke
```

If any config uses templates, the generated code imports `mustache` (qfg
prints a note); add it as a dependency then.
Re-run `qfg generate` after creating keys.

## Log levels

Pass `loggerKey: "log-level.my-app"` to the constructor, then wire the logger:
`createWinstonFormat` from `@quonfig/node/winston` or `createPinoHooks` from
`@quonfig/node/pino`. Read the package README for the exact wiring.

## Shutdown

`await quonfig.close()` on shutdown; `await quonfig.flush()` at the end of a
serverless invocation to send telemetry.
