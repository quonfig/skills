# .NET — `Quonfig.Sdk`

Server-side SDK; evaluates in-process.

## Install

```bash
dotnet add package Quonfig.Sdk
```

Companions: `Quonfig.Sdk.AspNetCore`, `Quonfig.Sdk.Extensions.Logging`,
`Quonfig.Sdk.Serilog`.

## Initialize once

`SdkKey` and `Environment` fall back to `QUONFIG_BACKEND_SDK_KEY` and
`QUONFIG_ENVIRONMENT`.

```csharp
await using var quonfig = new Quonfig.Sdk.Quonfig(new QuonfigOptions
{
    SdkKey = Environment.GetEnvironmentVariable("QUONFIG_BACKEND_SDK_KEY"),
});
await quonfig.InitAsync(); // await before relying on the sync getters
```

Local / offline mode (the SDK does not read `QUONFIG_DIR`; set it):

```csharp
var options = new QuonfigOptions { Datadir = "./workspace-data", Environment = "development" };
```

ASP.NET Core: `builder.Services.AddQuonfig(opts => ...)` registers it with a
hosted service that drives init and shutdown.

## Context

```csharp
var ctx = new ContextSet
{
    ["user"] = new ContextProperties { ["key"] = user.Id, ["email"] = user.Email },
};
var bound = quonfig.WithContext(ctx); // optional
```

## Feature flag

```csharp
if (quonfig.IsFeatureEnabled("new-dashboard", ctx))
{
    // new path
}
```

## Config values

Argument order is **(key, contexts, defaultValue)** — context before default:

```csharp
var s = quonfig.GetString("config.string", ctx);
var n = quonfig.GetInt("rate-limit", ctx);
```

Also `GetLong`, `GetBool`, `GetDouble`, `GetStringList`, `GetJson`,
`GetDuration`. With no default, a missing key throws (`OnNoDefault = Throw`).

## Log levels

Set `LoggerKey = "log-level.my-app"` on the options, then
`builder.Logging.AddQuonfigFilter(quonfig);`.

No `qfg generate` target for .NET.
