# Gupsh Skills

Agent guidance for integrating apps with [gupsh](https://github.com/smashah/gupsh-app), adapted from [Bento Skills](https://github.com/bentonow/bento-skills) under the MIT license. It covers Gupsh MCP and CLI surfaces plus source-reviewed routes for upstream Bento SDKs. The Gupsh API is Bento-compatible, while SDK packages retain their upstream package names.

Install the `gupsh` skill from this GitHub repository:

```bash
npx skills add smashah/gupsh-skills
```

The [skill](skills/gupsh/SKILL.md) routes agents to focused references. Its [helper](skills/gupsh/scripts/gupsh-sdk.mjs) prints detect/install guidance without installing packages or reading credentials:

```bash
node skills/gupsh/scripts/gupsh-sdk.mjs list
node skills/gupsh/scripts/gupsh-sdk.mjs detect .
node skills/gupsh/scripts/gupsh-sdk.mjs install mcp
```

Use `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` for supported SDK redirects and inject the site UUID and keys through the owner app's deployment environment. The reviewed language recipes are source-backed; runtime behavior against Gupsh is still unobserved. Neither the MCP nor CLI fork is claimed as published to npm here.

MIT. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
