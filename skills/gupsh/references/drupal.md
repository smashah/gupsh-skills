# Bento Drupal SDK

## Point it at gupsh

**Unverified against gupsh.** This upstream Bento package may hardcode Bento hosts or wire behavior. No source-reviewed host override is available here. Inspect its transport and configure `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` only if the package supports an explicit base URL; do not assume the remaining upstream examples route to gupsh.


Use for Drupal 10 sites and custom modules that should integrate through Drupal services, admin config, cron, queues, Webform, Commerce, and Drupal mail.

## Install And Detect

Developer install:

```bash
composer config repositories.bento-drupal-sdk vcs https://github.com/bentonow/bento-drupal-sdk
composer require drupal/bento_sdk:dev-main
drush en bento_sdk
```

The local README also mentions `composer require bentonow/bento-drupal-sdk`; prefer the package name available to the consuming project. If Composer cannot resolve it, add the VCS repository and require `drupal/bento_sdk`.

Detect from project/files:

- `bento_sdk.info.yml`, `bento_sdk.module`, or `bento_sdk.services.yml`
- `composer.json` type `drupal-module` or dependency `drupal/bento_sdk`
- `\Drupal::service('bento.sdk')`

Configure in Drupal admin at `/admin/config/bento/settings`, with production secret key in `GUPSH_SECRET_KEY` when possible.

## Preferred Integration Recipes

Subscribers:

- Use `$bento = \Drupal::service('bento.sdk')`.
- Use `$bento->importSubscribers([...])` for sync/import flows.
- Treat import as upsert-style sync; pass email, first/last names, tags, remove tags, and custom fields.
- Use `$bento->createSubscriber(...)` only for truly single-user interactive paths.

Events:

- Use `$bento->sendEvent([...])`; the service queues events for background processing.
- Let Drupal cron process the queue and retry manager.
- Use `$bento->sendEventSync(...)` only inside queue workers or controlled fallback paths.

Tags and fields:

- Use `$bento->addTag`, `removeTag`, `addField`, and `removeField` for profile state.
- Use event tracking for behavioral automation triggers.

Transactional email:

- Use the module's Drupal mail integration when routing site mail through Bento.
- Select a verified author in admin config. The module fetches and caches authors.
- Rely on the documented fallback to Drupal mail when Bento cannot send.

Webform and Commerce:

- Prefer built-in Webform processing instead of custom submit handlers. It maps standard email/name fields and queues events.
- Prefer built-in Commerce tracking for cart, order, payment, and abandoned-cart events.
- Use queue stats and logs to monitor high-volume sites.

Validation:

- Enable email validation in module settings when the site needs API validation.
- The service caches validation results; do not call validation repeatedly in tight loops.

## Use This, Not That

- Use Drupal services/config/forms, not standalone PHP client construction.
- Use queue/cron processing, not synchronous API calls during page render.
- Use built-in Webform/Commerce integrations, not duplicate custom event subscribers.
- Use bulk subscriber import for existing users, not `createSubscriber` inside an entity query loop.

## Verification

```bash
drush pm:list --status=enabled | grep bento_sdk
drush config:get bento_sdk.settings
drush cron
php test_autoload.php
```

Test custom module integrations with mocked Drupal services where possible. Cover queue creation, cron processing, config access, and no-email webform submissions.
