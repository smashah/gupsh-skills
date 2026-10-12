# gupsh CLI

Use the CLI for terminal account work, CSV imports, and machine-readable output. The Gupsh fork lives at [smashah/gupsh-cli](https://github.com/smashah/gupsh-cli); npm publication is unverified, so inspect the repository before treating `npx` as available.

```bash
npx @gupsh/cli --help
gupsh auth status
gupsh stats site --json
```

The binary is `gupsh`. Inject `GUPSH_SITE_UUID`, `GUPSH_PUBLISHABLE_KEY`, `GUPSH_SECRET_KEY`, and `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` from the environment or CLI-managed profile, with separate credentials for staging and production. Do not echo keys.

For bulk audience sync, use `gupsh subscribers import contacts.csv --dry-run --limit 10` to preview, then `gupsh subscribers import contacts.csv --confirm --json` only when the target account and input are intended. Do not loop over individual subscriber writes. Use `gupsh sequences list --json` before targeting an email: create appends a new email; update patches the existing numeric `templateId`. A broadcast created through the API starts as a draft for review. The CLI command surface follows the [source fork](https://github.com/smashah/gupsh-cli/tree/work/cli-146); no runtime command execution has been observed for this skill.
