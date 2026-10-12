# Bento .NET SDK

## Point it at gupsh

**Unverified against gupsh.** This upstream Bento package may hardcode Bento hosts or wire behavior. No source-reviewed host override is available here. Inspect its transport and configure `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` only if the package supports an explicit base URL; do not assume the remaining upstream examples route to gupsh.


Use for ASP.NET Core and .NET services that should use dependency injection, `appsettings`, typed request records, and async services.

## Install And Detect

Install:

```bash
dotnet add package Bento.SDK
```

Detect from project/files:

- `.csproj` package reference `Bento.SDK`
- `using Bento.Extensions`
- `builder.Services.AddBentoClient(builder.Configuration)`
- injected interfaces like `IBentoSubscriberService`

Configure:

```json
{
  "Bento": {
    "PublishableKey": "...",
    "SecretKey": "...",
    "SiteUuid": "..."
  }
}
```

Register once:

```csharp
using Bento.Extensions;

builder.Services.AddBentoClient(builder.Configuration);
```

## Preferred Integration Recipes

Subscribers:

- Inject `IBentoSubscriberService`.
- Use `ImportSubscribersAsync<T>(IEnumerable<SubscriberRequest>)` for sync/import flows.
- Treat import as upsert-style sync; include email, names, tags, remove tags, and fields.
- Use `FindSubscriberAsync` or `CreateSubscriberAsync` only for targeted interactive paths.

Events:

- Inject `IBentoEventService`.
- Use `TrackEventsAsync<T>(IEnumerable<EventRequest>)` for batches.
- Use `TrackEventAsync<T>(EventRequest)` only for single application events.

Tags and fields:

- Inject `IBentoCommandService` for subscriber state changes.
- Use `ExecuteBatchCommandsAsync<T>(IEnumerable<CommandRequest>)` for tag/field/subscription/email-change batches.
- Use field/tag services to list/create definitions.

Transactional email, broadcasts, stats, validation:

- Inject `IBentoEmailService` and send `EmailRequest`.
- Inject `IBentoBroadcastService` and use `CreateBroadcastAsync`.
- Inject `IBentoStatsService` for `GetSiteStatsAsync`, segment, and report stats.
- Inject validation and blacklist services for enrichment. Geolocation is disabled (`{}` response).

## Use This, Not That

- Use `AddBentoClient` and injected services, not direct `HttpClient` calls in controllers.
- Use async methods with cancellation where the application exposes it.
- Use `ImportSubscribersAsync` and `TrackEventsAsync`, not `foreach` loops of single calls.
- Use configuration providers or environment overrides for credentials, not checked-in secrets.
- Put imports and bulk email in background services/jobs, not request handlers.

## Verification

```bash
dotnet restore
dotnet build
dotnet test
```

If using the local SDK examples, functional tests can run with dummy credentials and assert that the test summary appears. In app tests, mock injected Bento service interfaces and assert request records and batch sizes.
