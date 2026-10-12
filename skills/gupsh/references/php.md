# Bento PHP SDK

## Point it at gupsh

**Unverified against gupsh.** This upstream Bento package may hardcode Bento hosts or wire behavior. No source-reviewed host override is available here. Inspect its transport and configure `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` only if the package supports an explicit base URL; do not assume the remaining upstream examples route to gupsh.


Use for plain PHP, Composer libraries, Symfony/custom apps, and non-Laravel PHP integrations.

## Install And Detect

Install:

```bash
composer require bentonow/bento-php-sdk
```

Detect from project/files:

- `composer.json` dependency `bentonow/bento-php-sdk`
- `use bentonow\Bento\BentoAnalytics`
- `$bento->V1->Batch` or `new BentoAnalytics(...)`

Configure once from env:

```php
use bentonow\Bento\BentoAnalytics;

$bento = new BentoAnalytics([
    'authentication' => [
        'publishableKey' => getenv('GUPSH_PUBLISHABLE_KEY'),
        'secretKey' => getenv('GUPSH_SECRET_KEY'),
    ],
    'siteUuid' => getenv('GUPSH_SITE_UUID'),
]);
```

## Preferred Integration Recipes

Subscribers:

- Use `$bento->V1->Batch->importSubscribers(['subscribers' => $rows])` for audience syncs.
- Chunk to the SDK batch limit before calling Bento.
- Treat subscriber import as upsert-style sync; include email, fields, tags, and remove tags when available.

Events:

- Use `$bento->V1->Batch->importEvents(['events' => $events])` for backfills or high volume.
- Use `$bento->V1->track(...)` or `trackPurchase(...)` for simple single-event paths.
- Use Bento event types for behavior that should trigger automations.

Tags and fields:

- Use `$bento->V1->Commands->addTag/removeTag/addField/removeField/subscribe/unsubscribe(...)`.
- Use commands for profile state, not for behavior.

Transactional email, broadcasts, stats, validation:

- Use SDK methods where present. If the PHP SDK lacks a newer endpoint, isolate raw HTTP in one adapter and keep Bento auth/config shared.
- Use `$bento->V1->Fields->getFields()` and `$bento->V1->Tags->getTags()`.
- Use `$bento->V1->Experimental->validateEmail(...)`, `guessGender`, `checkBlacklist` for enrichment/validation.

## Use This, Not That

- Use `Batch->importSubscribers`, not a `foreach` loop of subscriber creates.
- Use Composer autoload and one configured client, not repeated manual includes/client construction.
- Use Commands for tags/fields/subscription changes, not custom endpoint strings sprinkled through the app.
- Use env/config, not hardcoded credentials.

## Verification

```bash
composer install
composer run-script test
vendor/bin/phpunit
```

Test with mocked HTTP responses. Cover batch chunking, invalid email rows, command payloads, and exception handling.
