# Python SDK

## Point it at gupsh

**Runtime status: unobserved.** Install the upstream package with `pip install 'git+https://github.com/bentonow/bento-python-sdk.git@fbd4b9412d27d79d13e9ae60bd8ef4e3296b2143'`. `bento_api.api` reads a module-level `BASE_URL` at each request, so set it once before constructing clients. For Python 3.12/3.13, the upstream source also has four undefined annotation names at import time:

```python
import builtins
import os

for name in ("SubscriberAttributes", "FieldAttributes", "TagAttributes", "BroadcastAttributes"):
    if not hasattr(builtins, name):
        setattr(builtins, name, dict)  # upstream annotations only

import bento_api.api as api

api.BASE_URL = os.environ["GUPSH_API_BASE_URL"].rstrip("/")
bento = api.BentoAPI(
    os.environ["GUPSH_SITE_UUID"],
    os.environ["GUPSH_PUBLISHABLE_KEY"],
    os.environ["GUPSH_SECRET_KEY"],
)
stats = bento.get_site_stats()
```

Use `GUPSH_API_BASE_URL=https://api.gupsh.app/api/v1`. The import workaround alters process-global builtins and should run before importing this SDK; it is needed only while the pinned upstream version has those annotations. The [upstream implementation](https://github.com/bentonow/bento-python-sdk/blob/fbd4b9412d27d79d13e9ae60bd8ef4e3296b2143/bento_api/api.py) also indexes `response['data']['attributes']` in `get_subscriber`, so a missing subscriber (`data: null`) raises rather than returning `None`. Its `execute_commands` sends a plural `commands` key, which gupsh's compatibility route accepts. The local stats call and three-count result have not been observed.

For bulk sync, use the SDK batch import path and chunk according to the API limits; do not send one request per user. Keep the secret key on the server.
