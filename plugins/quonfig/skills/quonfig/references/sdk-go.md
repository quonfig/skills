# Go — `github.com/quonfig/sdk-go`

Server-side SDK; evaluates in-process.

## Install

```bash
go get github.com/quonfig/sdk-go
```

## Initialize once

```go
import quonfig "github.com/quonfig/sdk-go"

// WithSdkKey falls back to $QUONFIG_BACKEND_SDK_KEY when omitted.
client, err := quonfig.NewClient(
    quonfig.WithSdkKey(os.Getenv("QUONFIG_BACKEND_SDK_KEY")),
)
if err != nil {
    log.Fatal(err)
}
defer client.Close()
```

`NewClient` blocks for the first load (up to `WithInitTimeout`, default 10s).
The backend key decides the environment.

Local / offline mode (the SDK does not read `QUONFIG_DIR`; pass it):

```go
client, err := quonfig.NewClient(
    quonfig.WithDataDir(os.Getenv("QUONFIG_DIR")),
    quonfig.WithEnvironment("development"), // or $QUONFIG_ENVIRONMENT
)
```

## Context

```go
ctx := quonfig.NewContextSet().
    WithNamedContextValues("user", map[string]interface{}{
        "key":   user.ID,
        "email": user.Email,
    })
```

Pass it per call (`nil` = no context), or bind it:
`bound := client.WithContext(ctx)` gives methods without the ctx argument
(`bound.FeatureIsOn(key)`, `bound.GetStringValue(key)`).

## Feature flag

```go
on, found := client.FeatureIsOn("new-dashboard", ctx)
if on {
    // new path
}
```

Returns `false` (and `found == false`) when the key does not exist.

## Config values

```go
s, found, err := client.GetStringValue("config.string", ctx)
n, found, err := client.GetIntValue("rate-limit", ctx) // int64
```

Also `GetBoolValue`, `GetFloatValue`, `GetStringSliceValue`,
`GetDurationValue`, `GetJSONValue` — all `(key, *ContextSet)` returning
`(value, found, err)`. No default argument: handle `found == false` yourself.

## Log levels

`quonfig.WithLoggerKey("log-level.my-app")` plus
`slog.New(quonfig.NewQuonfigHandler(client, inner, loggerPath))`.

No `qfg generate` target for Go.
