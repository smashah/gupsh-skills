# Ruby SDK and ActionMailer

## Point it at gupsh

**Runtime status: unobserved for all three recipes.** Set `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1` and inject the site UUID and keys through environment variables. Install the modern and legacy clients with `gem 'bento-sdk'`; add `gem 'bento-actionmailer'` only when using that delivery method.

The modern `Bento::Client` creates a new Faraday connection in private `conn`. Put this in an initializer loaded after the gem and before the first Bento request:

```ruby
require "uri"

module GupshBentoConnection
  private

  def conn
    base = URI(ENV.fetch("GUPSH_API_BASE_URL"))
    super.tap { |connection| connection.url_prefix = "#{base.scheme}://#{base.host}#{base.port == base.default_port ? '' : ":#{base.port}"}" }
  end
end

Bento::Client.prepend(GupshBentoConnection)
Bento.configure do |config|
  config.site_uuid = ENV.fetch("GUPSH_SITE_UUID")
  config.publishable_key = ENV.fetch("GUPSH_PUBLISHABLE_KEY")
  config.secret_key = ENV.fetch("GUPSH_SECRET_KEY")
end
```

The [upstream connection](https://github.com/bentonow/bento-ruby-sdk/blob/40147a64c6d6d195a4fe21666da9d0f7811b9672/lib/bento/core/client.rb) hardcodes `app.bentonow.com`; `dev_mode` only switches it to localhost. Faraday's `url_prefix=` is the connection's supported setter. `Bento::Subscribers.find_by(email: ...)` against gupsh is unobserved. The SDK raises on a missing subscriber; its purchase event validator also requires `product_sku`, `product_name`, and `quantity` in cart items.

The same gem's legacy `Bento::Analytics` posts to `/tracking/events/` using its own constant. Before creating an Analytics instance:

```ruby
legacy_host = URI(ENV.fetch("GUPSH_API_BASE_URL")).host
request_defaults = Bento::Analytics::Defaults::Request
request_defaults.send(:remove_const, :HOST)
request_defaults.const_set(:HOST, legacy_host)
```

That path sends `Basic write_key:` and an index-keyed `events` hash; the gupsh route is designed to accept it. The constant is read by the [legacy transport defaults](https://github.com/bentonow/bento-ruby-sdk/blob/40147a64c6d6d195a4fe21666da9d0f7811b9672/lib/bento/sdk/defaults.rb). It fixes the destination only; legacy event delivery remains unobserved.

For `bento-actionmailer`, set its endpoint before sending mail:

```ruby
BentoActionMailer::DeliveryMethod.send(:remove_const, :BENTO_ENDPOINT)
BentoActionMailer::DeliveryMethod.const_set(
  :BENTO_ENDPOINT,
  URI.join(ENV.fetch("GUPSH_API_BASE_URL").sub(%r{/+$}, "") + "/", "batch/emails")
)
```

Supply its normal settings from `GUPSH_SITE_UUID`, `GUPSH_PUBLISHABLE_KEY`, and `GUPSH_SECRET_KEY`. The [delivery method](https://github.com/bentonow/bento-actionmailer/blob/234ce9ead61c968ae7d82f311f26b9aecc816e37/lib/bento_actionmailer.rb) uses the endpoint constant for both its request URI and connection host; it sends a `site_uuid` JSON body and requires an HTML message. Its response handling raises on non-2xx responses. Mail delivery through gupsh is unobserved.

For bulk sync, use the SDK batch import path and chunk according to the API limits; do not send one request per user. Keep the secret key on the server.
