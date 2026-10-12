# Node SDK

## Point it at gupsh

**Runtime status: unobserved.** The stock Node SDK exposes `clientOptions.baseUrl`; no shim is needed. Install with `pnpm add @bentonow/bento-node-sdk` in the consuming app. The snippet belongs in server code, with all values supplied by the deployment environment:

```ts
import { Analytics } from "@bentonow/bento-node-sdk";

const bento = new Analytics({
  siteUuid: process.env.GUPSH_SITE_UUID!,
  authentication: {
    publishableKey: process.env.GUPSH_PUBLISHABLE_KEY!,
    secretKey: process.env.GUPSH_SECRET_KEY!,
  },
  clientOptions: { baseUrl: process.env.GUPSH_API_BASE_URL!.replace(/\/$/, "") },
});

// After signup, pass the source app's user.id as the subscriber uuid.
```

Set `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1`; the SDK appends endpoint paths such as `/stats/site` to that full base. The SDK reads `baseUrl` in [BentoClient](https://github.com/bentonow/bento-node-sdk/blob/fda1e095c6bcb8db96660d5310d83957b02ff14f/src/sdk/client/index.ts) and exposes the `Analytics` constructor in [src/index.ts](https://github.com/bentonow/bento-node-sdk/blob/fda1e095c6bcb8db96660d5310d83957b02ff14f/src/index.ts). If a package hides the SDK call site, review the package transport before using it.

Known behaviour: the SDK accepts only HTTP 200/201 for most API calls and the `upsertSubscriber` flow immediately fetches the subscriber after import, so the runtime must answer those shapes synchronously. No client-side host bug remains when `baseUrl` is set. Runtime behaviour against gupsh is still unobserved.

For bulk sync, use the SDK batch import path and chunk according to the API limits; do not send one request per user. Keep the secret key on the server.
