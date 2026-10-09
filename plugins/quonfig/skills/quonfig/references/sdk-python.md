# Python — `quonfig`

Server-side SDK; evaluates in-process.

## Install

```bash
pip install quonfig
```

## Initialize once

```python
import os

from quonfig import Quonfig

client = Quonfig(sdk_key=os.environ["QUONFIG_BACKEND_SDK_KEY"])
client.init()
```

Env fallbacks read by the SDK: `sdk_key` <- `QUONFIG_BACKEND_SDK_KEY`,
`environment` <- `QUONFIG_ENVIRONMENT`, `datadir` <- `QUONFIG_DIR`.

Local / offline mode:

```python
client = Quonfig(datadir="/path/to/workspace", environment="development")
client.init()
```

## Feature flag

```python
enabled = client.is_feature_enabled("my.flag", contexts={"user": {"key": user.id}})
```

Signature: `is_feature_enabled(key, default=False, contexts=None)`. The second
positional argument is the **default**, so always pass `contexts=` by keyword.

## Config values

```python
value = client.get_string("my.key", default="fallback")
limit = client.get_int("rate-limit", default=100, contexts={"user": {"plan": "pro"}})
```

Getters: `get`, `get_string`, `get_int`, `get_float`, `get_bool`,
`get_string_list`, `get_json`, `get_duration` — all
`(key, default=..., contexts=None)`. With no default, a missing key follows
`on_no_default` (raises by default).

## Context

```python
# Bound context (request handlers etc.)
user_client = client.with_context({"user": {"key": "u123", "plan": "pro"}})
enabled = user_client.is_feature_enabled("my.flag")

# Scoped: per thread / asyncio task, restored on exit
with client.scoped_context({"user": {"key": "u123"}}):
    enabled = client.is_feature_enabled("my.flag")
```

## Log levels

`logger_key="log-level.my-app"` plus `QuonfigLoggerFilter(client)` for stdlib
logging or `QuonfigLoggerProcessor` for structlog.

Shutdown: `client.close()`; `client.flush()` in serverless. No `qfg generate`
target for Python.
