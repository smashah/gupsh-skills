# Bento Elixir SDK

## Point it at gupsh

**Unverified against gupsh.** This upstream Bento package may hardcode Bento hosts or wire behavior. No source-reviewed host override is available here. Inspect its transport and configure `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` only if the package supports an explicit base URL; do not assume the remaining upstream examples route to gupsh.


Use for Elixir and Phoenix applications that should use Mix deps, application config, tuple result handling, supervised workers, and Oban/GenServer-style background processing.

## Install And Detect

Install in `mix.exs`:

```elixir
def deps do
  [
    {:bento_sdk, "~> 0.1.1"}
  ]
end
```

Then run:

```bash
mix deps.get
```

Detect from project/files:

- `mix.exs` dependency `:bento_sdk`
- `config :bento_sdk`
- calls like `BentoSdk.Subscribers.import(...)`

Configure:

```elixir
config :bento_sdk,
  site_uuid: System.fetch_env!("GUPSH_SITE_UUID"),
  username: System.fetch_env!("GUPSH_PUBLISHABLE_KEY"),
  password: System.fetch_env!("GUPSH_SECRET_KEY")
```

The SDK uses `username` for the Bento publishable key and `password` for the Bento secret key.

## Preferred Integration Recipes

Subscribers:

- Use `BentoSdk.Subscribers.import(list_of_maps)` for sync/import flows.
- Treat import as upsert-style sync. Include email, first/last names, tags, remove tags, and fields.
- Use `find`, `create`, or `find_or_create` only for single interactive paths.
- `Subscribers.update` uses the import endpoint for one subscriber; do not map it over a large list.

Events:

- Use `BentoSdk.Events.import_events([...])` for batches and backfills.
- Use `BentoSdk.Events.track(email, type, fields, details)` for one event.
- Events can create users and trigger automations; use them for behavior.

Tags and fields:

- Use `BentoSdk.Subscribers.add_tag`, `add_tags`, `remove_tag`, `add_field`, and `remove_field` for profile state.
- Use `BentoSdk.Tags.get/create` and `BentoSdk.Fields.get/create` for schema management.

Transactional email, broadcasts, stats, validation:

- Use `BentoSdk.Emails.send`, `send_transactional`, and `send_bulk`.
- Use `BentoSdk.Broadcasts.get/create`.
- Use `BentoSdk.Stats.get_site/get_segment/get_report`.
- Use `BentoSdk.Utility.validate_email` and `check_blacklist`.

## Use This, Not That

- Use batch imports from supervised jobs/workers, not `Enum.each` with one API call per user.
- Pattern match `{:ok, value}` and `{:error, reason}`, not unchecked happy paths.
- Use runtime config or releases env, not checked-in secrets.
- Use Oban, Broadway, GenServer, or supervised tasks for retries/backfills, not request-process loops.

## Verification

```bash
mix deps.get
mix test
mix format --check-formatted
```

Test with Mox or the SDK behavior where available. Cover tuple error handling, batch chunking, and supervised retry paths.
