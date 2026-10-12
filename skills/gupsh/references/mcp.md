# gupsh MCP

Use MCP for live account inspection and guided account operations from chat. App runtime code uses an SDK or HTTP client instead.

The hosted Streamable HTTP endpoint is `https://mcp.gupsh.app/mcp`. Configure these headers in the MCP client from secret storage: `X-Bento-Publishable-Key`, `X-Bento-Secret-Key`, and `X-Bento-Site-UUID`. Keep the secret key out of shared configs, source, logs, and chat output. These header names are part of the Bento-compatible wire contract.

For a local stdio client, use `npx -y @gupsh/mcp setup` for setup guidance or run `npx -y @gupsh/mcp`. See the [Gupsh MCP source](https://github.com/smashah/gupsh-mcp); npm publication and live endpoint behavior are not established by this skill.

The hosted surface preserves the Bento-prefixed `bento_*` names across 20 tools. The local stdio surface uses unprefixed names for the same operations. For example:

| Operation | Hosted | Local |
| --- | --- | --- |
| List sequences | `bento_list_sequences` | `list_sequences` |
| Create sequence email | `bento_create_sequence_email` | `create_sequence_email` |
| Update sequence email | `bento_update_sequence_email` | `update_sequence_email` |
| List tags | `bento_list_tags` | `list_tags` |
| Get subscriber | `bento_get_subscriber` | `get_subscriber` |

List before write, use a returned sequence `id` for creation and numeric `templateId` for updates, and review broadcasts as drafts before sending. Subscriber import uses `/batch/subscribers` for up to 1000 rows and does not trigger automations; a signup flow needs a separate event. Destructive, high-volume, or email-sending actions need a clear target and preview.
