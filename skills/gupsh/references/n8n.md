# Bento n8n Community Node

## Point it at gupsh

**Unverified against gupsh.** This upstream Bento package may hardcode Bento hosts or wire behavior. No source-reviewed host override is available here. Inspect its transport and configure `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` only if the package supports an explicit base URL; do not assume the remaining upstream examples route to gupsh.


Use for no-code/low-code workflows in self-hosted or cloud n8n. Prefer native node operations and n8n credential storage over custom Function node HTTP calls.

## Install And Detect

Self-hosted install:

```bash
cd ~/.n8n
npm install n8n-nodes-bento
n8n start
```

Cloud install:

- Search for the Bento community node in the n8n community node marketplace.
- Install it.
- Create a Bento API credential with Publishable Key, Secret Key, and Site UUID.
- Use the credential Test action before enabling production workflows.

Detect from project/files:

- package `n8n-nodes-bento`
- n8n package metadata pointing to `dist/nodes/Bento/Bento.node.js`
- credential type `BentoApi`

## Preferred Integration Recipes

Subscribers:

- Use the Bento node's Create Subscriber, Get Subscriber, Update Subscriber, and Subscriber Command operations.
- For a large source list, split workflow items into batches with n8n batching before Bento operations.
- Treat subscriber update/import workflows as upsert-style audience syncs.

Events:

- Use Track Event operation for behavior and automation triggers.
- Preserve event type, email, fields, and details from upstream nodes.

Tags and fields:

- Use List/Create Fields and List/Create Tags to manage audience schema.
- Use Subscriber Command for add/remove tag, add/remove field, subscribe, unsubscribe, and change email flows.

Transactional email, broadcasts, automation content:

- Use Send Transactional Email for operational messages.
- Use List Broadcasts and Send Broadcast for campaign workflows.
- Use List Sequences, Create Sequence Email, List Workflows, Get Email Template, and Update Email Template for automation content operations.

Stats and validation:

- Use Site Metrics, Segment Metrics, and Report Metrics for reporting workflows.
- Use Validate Email and Blacklist Check and Content Moderation where appropriate. Do not use Geolocation Lookup; the API returns `{}`.

## Use This, Not That

- Use n8n credentials, not hardcoded keys in Function nodes.
- Use Bento node operations, not hand-written HTTP Request nodes unless the node lacks the operation.
- Use Split In Batches and Continue On Fail for large imports, not one unbounded workflow execution.
- Use separate development credentials/workflows before production.
- Use n8n execution logs and error workflows for retries.

## Verification

```bash
npm list n8n-nodes-bento
n8n start
```

In n8n, add a Bento credential and click Test. Then run a workflow with one test subscriber/event before turning on production triggers.
