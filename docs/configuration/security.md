# Security

Grimoire ships with two layers of hardening enabled by default: rate limiting on the endpoints that check credentials, and a set of security response headers. Neither requires configuration, but both can be tuned. Cross-origin access for browser-based integrations is off unless you turn it on.

## Auth rate limiting

The credential-checking endpoints are rate-limited **per client IP** to slow online password and invite-code brute-forcing:

- `/api/auth/login`
- `/api/auth/setup`
- `/api/auth/guest-login`
- failed [API key](/api#api-keys) attempts, on any endpoint

The default is **`10/minute`** per IP; exceeding it returns HTTP `429`.

Only *failed* API key attempts count, so a dashboard or script with a working key is never throttled, however often it polls. Once an IP is over the limit, every key it sends is refused until the window passes, even a valid one, so a guess can't be confirmed.

| Variable | Default | Description |
|---|---|---|
| `AUTH_RATE_LIMIT` | `10/minute` | A [`limits`](https://limits.readthedocs.io/en/stable/quickstart.html#rate-limit-string-notation) string such as `20/minute` or `100/hour`. |
| `RATE_LIMIT_ENABLED` | `true` | Set to `false` to turn off auth rate limiting entirely. |
| `TRUST_FORWARDED_FOR` | `true` | Whether to key on the `X-Forwarded-For` address. |

### Behind a reverse proxy

By default (`TRUST_FORWARDED_FOR=true`) the limiter keys on the left-most `X-Forwarded-For` address, so each real client, not the proxy, gets its own bucket. Make sure your proxy sets `X-Forwarded-For`.

::: warning Direct exposure
If Grimoire is exposed directly with no trusted proxy in front, set `TRUST_FORWARDED_FOR=false`. Otherwise a client could spoof `X-Forwarded-For` to sidestep the limit.
:::

### Multiple replicas

When [`VALKEY_URL`](/configuration/env-vars#optional-features) is set, the limit counters are shared through Valkey, so the limit is enforced consistently across all workers and replicas. Without it, each process keeps its own in-memory counters. The limiter also falls back to in-memory automatically if Valkey becomes unreachable.

## Security headers

Every response carries:

| Header | Value |
|---|---|
| `Content-Security-Policy` | Scoped to what the SPA actually loads: own scripts, inline styles used by React, Google Fonts, and `data:`/`blob:` images for rendered pages. Includes `frame-ancestors 'none'`. |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` (matching the CSP `frame-ancestors 'none'`) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Strict-Transport-Security` | Emitted **only when the request is HTTPS**: directly or via `X-Forwarded-Proto: https` from your TLS-terminating proxy, so it is never sent over plain HTTP. |

These are on by default and require no configuration. If you terminate TLS at a reverse proxy, make sure it forwards `X-Forwarded-Proto: https` so HSTS is emitted correctly. See [Reverse Proxy](/deployment/reverse-proxy).

## Cross-origin requests (CORS)

By default Grimoire sends no CORS headers, so browsers block web pages on other sites from reading its API. To let a browser-based integration such as a Foundry VTT module call the API, list its origin in `CORS_ALLOWED_ORIGINS`:

```yaml
environment:
  CORS_ALLOWED_ORIGINS: "https://foundry.example.com"
```

Only the exact origins listed are allowed; a wildcard or a path is ignored with a warning. Grimoire never sends `Access-Control-Allow-Credentials`, so browsers won't send its cookies cross-origin, and another site can't act through a signed-in user's session. Allowed sites authenticate with an [API key](/api#api-keys), limited by that key's permissions. See [API → Calling the API from a browser](/api#calling-the-api-from-a-browser).
