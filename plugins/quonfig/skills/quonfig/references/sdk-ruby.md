# Ruby — `quonfig` gem

Server-side SDK; evaluates in-process.

## Install

```ruby
gem 'quonfig'
```

## Initialize once

```ruby
require 'quonfig'

client = Quonfig::Client.new(sdk_key: ENV['QUONFIG_BACKEND_SDK_KEY'])
```

Or as a process singleton (e.g. a Rails initializer), then `Quonfig.instance`:

```ruby
Quonfig.init(Quonfig::Options.new(sdk_key: ENV.fetch('QUONFIG_BACKEND_SDK_KEY')))
```

The constructor blocks on the first load. The backend key decides the
environment. Local / offline mode reads `QUONFIG_DIR` + `QUONFIG_ENVIRONMENT`:

```ruby
client = Quonfig::Client.new(datadir: '/path/to/workspace', environment: 'development')
```

## Feature flag

```ruby
if client.enabled?('new-dashboard', user: { key: current_user.id })
  # new path
end
```

## Config values

```ruby
client.get_string('app.display-name')
client.get_int('rate-limit', default: 100, context: { user: { key: 'user-123' } })
client.get_bool('flags.new-checkout', default: false)
```

Also `get_float`, `get_string_list`, `get_duration` (ms), `get_json`. A missing
key returns `default:`; with no default, `on_no_default` (`:raise` by default)
decides.

## Context

```ruby
client.with_context(user: { key: 'user-123', plan: 'pro' }) do |bound|
  bound.enabled?('beta-feature')
end
```

`in_context` is a deprecated alias. `global_context:` is a constructor option.

## Log levels

SemanticLogger: `filter: Quonfig.instance.semantic_logger_filter(config_key: 'log-level.my-app')`.
Stdlib Logger: `logger_key:` option plus `client.stdlib_formatter(logger_name:)`.

Shutdown: `client.stop`. No `qfg generate` target for Ruby.
