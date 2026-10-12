# Go SDK

## Point it at gupsh

**Runtime status: unobserved.** Install with `go get github.com/bentonow/bento-golang-sdk@f70dbd926f0492e6f0af5d839f1136e2bbe0f7de`. The SDK's base URL is private, but its exported `SetHTTPClient` accepts a request doer. This wrapper rewrites only the Bento API origin and leaves path, query, authorization, and request body intact:

```go
package gupsh

import (
    "net/http"
    "net/url"
    "os"
    "time"

    bento "github.com/bentonow/bento-golang-sdk"
)

type gupshDoer struct {
    inner *http.Client
    base *url.URL
}

func (d gupshDoer) Do(req *http.Request) (*http.Response, error) {
    if req.URL.Hostname() == "app.bentonow.com" {
        cloned := req.Clone(req.Context())
        cloned.URL.Scheme = d.base.Scheme
        cloned.URL.Host = d.base.Host
        cloned.Host = d.base.Host
        return d.inner.Do(cloned)
    }
    return d.inner.Do(req)
}

func New() (*bento.Client, error) {
    base, err := url.Parse(os.Getenv("GUPSH_API_BASE_URL"))
    if err != nil { return nil, err }
    client, err := bento.NewClient(&bento.Config{
        PublishableKey: os.Getenv("GUPSH_PUBLISHABLE_KEY"),
        SecretKey: os.Getenv("GUPSH_SECRET_KEY"),
        SiteUUID: os.Getenv("GUPSH_SITE_UUID"),
        Timeout: 10 * time.Second,
    })
    if err != nil { return nil, err }
    if err := client.SetHTTPClient(gupshDoer{inner: &http.Client{Timeout: 10 * time.Second}, base: base}); err != nil {
        return nil, err
    }
    return client, nil
}
```

Set `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1`; the SDK already builds `/api/v1` paths. The [upstream client](https://github.com/bentonow/bento-golang-sdk/blob/f70dbd926f0492e6f0af5d839f1136e2bbe0f7de/client.go) rejects publishable keys, secret keys, and site UUIDs outside 28–36 characters and accepts 200/201 for requests. `GetBroadcasts` reads a top-level `broadcasts` array, so a `data`-only response will decode empty. `GetSiteStats(ctx)` against gupsh has not been observed.

For bulk sync, use the SDK batch import path and chunk according to the API limits; do not send one request per user. Keep the secret key on the server.
