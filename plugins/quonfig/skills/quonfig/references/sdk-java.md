# Java — `com.quonfig:sdk-java`

Server-side SDK; evaluates in-process.

## Install

```kotlin
implementation("com.quonfig:sdk-java:1.5.0")
```

```xml
<dependency>
    <groupId>com.quonfig</groupId>
    <artifactId>sdk-java</artifactId>
    <version>1.5.0</version>
</dependency>
```

Check Maven Central for the latest version. Logging add-ons:
`sdk-java-logback`, `sdk-java-log4j2`.

## Initialize once

`Options` reads `QUONFIG_BACKEND_SDK_KEY` and `QUONFIG_ENVIRONMENT` from the
environment.

```java
import com.quonfig.sdk.BoundQuonfig;
import com.quonfig.sdk.Options;
import com.quonfig.sdk.Quonfig;
import com.quonfig.sdk.eval.ContextSet;

Options opts = Options.builder()
    .sdkKey(System.getenv("QUONFIG_BACKEND_SDK_KEY"))
    .build();
Quonfig quonfig = new Quonfig(opts);
quonfig.initFuture().get(); // wait for the first load
```

Local / offline mode (the SDK does not read `QUONFIG_DIR`; `environment` is
required):

```java
Options opts = Options.builder()
    .datadir("./workspace-data")
    .environment("development")
    .build();
```

`Quonfig` is `AutoCloseable` — `close()` on shutdown.

## Context

```java
ContextSet ctx = new ContextSet()
    .withNamedContext("user", Map.of("key", user.getId(), "email", user.getEmail()));
BoundQuonfig bound = quonfig.withContext(ctx); // optional
```

## Feature flag

```java
if (quonfig.featureIsOn("new-dashboard", ctx)) {
    // new path
}
```

## Config values

The default is a required argument; context is the optional third:

```java
String s = quonfig.getString("config.string", "<missing>", ctx);
Long n = quonfig.getInt("rate-limit", 100L, ctx);
```

Also `getBool`, `getDouble`, `getStringList`, `getDuration`, `getJson`.

## Log levels

`.loggerKey("log-level.my-app")` on the builder, then
`QuonfigLogbackTurboFilter.install(quonfig)` (`com.quonfig.sdk.logback`) or
`QuonfigLog4j2Filter.install(quonfig)` (`com.quonfig.sdk.log4j2`).

No `qfg generate` target for Java.
