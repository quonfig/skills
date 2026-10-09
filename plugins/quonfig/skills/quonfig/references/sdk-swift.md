# Swift (iOS 15+ / macOS 12+) — `Quonfig`

A frontend SDK: it evaluates on the server for one context, so use a
**frontend (browser) SDK key** (`qfg sdk-key create --environment <env> --type browser`),
never the backend key.

## Install (SwiftPM)

```swift
dependencies: [
    .package(url: "https://github.com/quonfig/sdk-swift.git", from: "0.2.0"),
],
targets: [
    .target(
        name: "YourApp",
        dependencies: [
            .product(name: "Quonfig", package: "sdk-swift"),
        ]
    ),
]
```

## Initialize once

```swift
import Quonfig

let quonfig = try await Quonfig.initialize(
    context: QuonfigContext([
        "user": ["key": .string("u_123"), "email": .string("a@b.com")],
    ]),
    options: .init(sdkKey: frontendSdkKey, domain: "quonfig.com")
)
```

The SDK reads no environment variables; the key decides the environment.

## Flags and values

```swift
let on    = quonfig.isEnabled("new-checkout")                 // Bool
let color = quonfig.string("button-color", default: "blue")   // String
let limit = quonfig.int("rate-limit", default: 100)           // Int
let hosts = quonfig.stringList("allowed-hosts", default: [])  // [String]
```

Reads are synchronous and return the default until values arrive.

## Context

```swift
try await quonfig.updateContext(
    QuonfigContext(["user": ["key": .string("u_456")]])
)
```

## Log levels and shutdown

`QuonfigLogger(quonfig: quonfig, loggerKey: "log-level.my-app")`, or
`quonfig.shouldLog(.debug, loggerKey: "log-level.my-app")`.
`await quonfig.shutdown()` when done.
