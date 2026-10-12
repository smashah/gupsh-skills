# Laravel SDK

## Point it at gupsh

**Runtime status: untested on staging; unobserved.** Install with `composer require bentonow/bento-laravel-sdk`. Map the injected `GUPSH_SITE_UUID`, `GUPSH_PUBLISHABLE_KEY`, and `GUPSH_SECRET_KEY` to the package's existing Bento config inputs on the server, alongside `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1`. No key belongs in client code or checked-in config.

The Saloon connector and mail transport take different paths. In `app/Providers/AppServiceProvider.php`, override the package's `'bento'` binding after all providers have registered and rewrite the package's hardcoded mail destination through Laravel's global HTTP request middleware:

```php
use Bentonow\BentoLaravel\BentoConnector;
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\RequestInterface;

public function boot(): void
{
    $base = rtrim((string) getenv('GUPSH_API_BASE_URL'), '/');
    if ($base === '') {
        throw new \RuntimeException('GUPSH_API_BASE_URL is required');
    }

    $this->app->singleton('bento', fn () => new class($base) extends BentoConnector {
        public function __construct(private string $gupshBase) {}
        public function resolveBaseUrl(): string { return $this->gupshBase; }
    });

    $origin = parse_url($base);
    Http::globalRequestMiddleware(function (RequestInterface $request) use ($origin) {
        if ($request->getUri()->getHost() !== 'app.bentonow.com') {
            return $request;
        }
        $uri = $request->getUri()
            ->withScheme($origin['scheme'])
            ->withHost($origin['host'])
            ->withPort($origin['port'] ?? null);
        return $request->withUri($uri);
    });
}
```

The [connector](https://github.com/bentonow/bento-laravel-sdk/blob/1c8241fc187ff24914ffbd6c64415885e21648b7/src/BentoConnector.php) sets Basic auth and `site_uuid` and hardcodes the API base. The [service provider](https://github.com/bentonow/bento-laravel-sdk/blob/1c8241fc187ff24914ffbd6c64415885e21648b7/src/BentoLaravelServiceProvider.php) registers `'bento'` in `register`; an app provider's `boot` runs after registration. The [mail transport](https://github.com/bentonow/bento-laravel-sdk/blob/1c8241fc187ff24914ffbd6c64415885e21648b7/src/BentoTransport.php) uses `Http::baseUrl('https://app.bentonow.com')`, so the connector binding alone does not redirect `MAIL_MAILER=bento`. `Http::globalRequestMiddleware` requires Laravel 10.14 or newer.

Known SDK behavior: mail error handling gives a dedicated message for 401 and a Bento-specific 500, but gupsh's author rejection uses 403, which appears as a generic transport error. The SDK's `upsertSubscriber` imports and then immediately fetches; the runtime must write before answering the import. Connector and mail transport calls have not been observed against gupsh.

For bulk sync, use the SDK batch import path and chunk according to the API limits; do not send one request per user. Keep the secret key on the server.
