---
name: gupsh
description: Build correct gupsh (Bento-compatible) integrations. Use when adding gupsh to an app, choosing hosted or local MCP, the gupsh CLI, or an upstream Bento SDK, syncing subscribers, tracking events and purchases, sending email, or configuring credentials.
---

# gupsh

Read [onboarding](references/onboarding.md) when connecting an owner app. Choose one surface for the task and read its reference; use the [helper](scripts/gupsh-sdk.mjs) to identify an existing SDK or print installation guidance. The helper prints text and does not install packages.

| Surface | Use | Reference |
| --- | --- | --- |
| Hosted MCP `https://mcp.gupsh.app/mcp` | Chat account operations; configure `X-Bento-Publishable-Key`, `X-Bento-Secret-Key`, `X-Bento-Site-UUID` headers in the MCP client. | `references/mcp.md` |
| Local MCP `npx -y @gupsh/mcp` | Stdio MCP clients. | `references/mcp.md` |
| CLI `npx @gupsh/cli` (binary `gupsh`) | Terminal and CSV work. | `references/cli.md` |
| Node `@bentonow/bento-node-sdk` | Server-side application integration; set `clientOptions.baseUrl` to `https://api.gupsh.app/api/v1`. | `references/node.md` |
| Other upstream SDKs | Use only the language-specific Gupsh transport recipe or heed its unverified warning. | `references/python.md`, `references/go.md`, `references/ruby.md`, `references/laravel.md`, `references/php.md`, `references/drupal.md`, `references/dotnet.md`, `references/elixir.md`, `references/n8n.md` |

The fork's MCP and CLI repositories are [gupsh-mcp](https://github.com/smashah/gupsh-mcp) and [gupsh-cli](https://github.com/smashah/gupsh-cli). Source and install guidance do not prove an npm release or a live service.

## Integration rules

- A linked subscriber's Bento `uuid` is the owner app's Better Auth `user.id`; include it in the subscriber import row, not a custom field. Skip anonymous users.
- In SmashStack apps, use the Better Auth post-create hook for signup sync, then send a `$signed_up` event. Purchases come from verified payment webhooks after persistence, with the provider order ID as the unique key and minor units as the amount. Do not send revenue to the portfolio analytics dataset.
- Honor consent through the app's intended double opt-in path. A signup or import alone does not imply marketing subscription; unsubscribe users without consent. Read [onboarding](references/onboarding.md) for the concrete flow.
- Inject `GUPSH_SITE_UUID`, `GUPSH_PUBLISHABLE_KEY`, `GUPSH_SECRET_KEY`, and `GUPSH_API_BASE_URL` from the deployment environment. In SmashStack, declare these in the relevant Worker's `.env.schema`; put sensitive keys in its allowlist and credential manifest. Do not put literal keys in source, tests, docs, logs, browser bundles, or chat output.
- Use batch subscriber/event APIs for syncs; do not loop over one API request per user. Chunk at the documented endpoint limits. Keep secret-key operations on the server.
- Use events for behavior and automation triggers; use commands for profile and subscription changes. `POST /fetch/commands` queues work and returns `{ results }`, not an updated subscriber.
- Preview high-volume or destructive account work before execution. Inspect tags, fields, sequences, and broadcasts before creating duplicates or targeting a record.

## API details

The API base is `https://api.gupsh.app/api/v1`; do not omit `/api/v1` for the Node SDK. Subscriber and event batches accept up to 1000 items; transactional email batches accept up to 60. Subscriber import does not trigger automations, so send the intended event separately. New API broadcasts are drafts; list broadcasts explicitly when the default sent-only view is insufficient. Use returned sequence IDs and numeric `templateId` values for sequence email updates.
