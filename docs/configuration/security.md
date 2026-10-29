# Security

Grimoire ships with two layers of hardening enabled by default: rate limiting on the endpoints that check credentials, and a set of security response headers. Neither requires configuration, but both can be tuned. Cross-origin access for browser-based integrations is off unless you turn it on.

## Non-root container

The image does not run Grimoire as root. It starts as root only to hand the data volume to
an unprivileged user, then drops to it before Grimoire starts, so add-on scripts, the PDF,
image and archive parsers and `ffmpeg` all run without root. The app's own code stays owned
by root, so the running app can't modify it. See [File ownership](/configuration/volumes#file-ownership)
for which user it picks and how to choose one.

## Auth rate limiting

The credential-checking endpoints are rate-limited **per client IP** to slow online password and invite-code guessing:

- `/api/auth/login`
- `/api/auth/setup`
- `/api/auth/guest-login`
- failed [API key](/api#api-keys) attempts, on any endpoint

The default is **`10/minute`** per IP; exceeding it returns HTTP `429`. An IPv6 client is counted by its `/64` network rather than its single address, since one host usually holds a whole `/64` and could otherwise rotate through fresh buckets. Private and link-local IPv6 addresses are still counted one by one, as every device on a LAN segment shares one `/64`.

Only *failed* API key attempts count, and a working key is never refused, so a dashboard or script keeps working however often it polls, even from an address it shares with others (a VPN exit, a household router).

### Per account and per link

Failed sign-ins are also counted against the **account** being signed in to (by the username typed, whether or not it exists), and wrong share-link passwords against the **link**, however many addresses the attempts come from. The default is **`20/hour`**; past it, sign-in to that account returns `429` until the window passes.

A browser that has signed in to an account before keeps its own count (it holds an HttpOnly `grimoire_device` cookie for that account), so failed attempts elsewhere don't stop the owner signing in from their usual browser. Single sign-on and existing sessions are not affected.

| Variable | Default | Description |
|---|---|---|
| `AUTH_RATE_LIMIT` | `10/minute` | Per-IP limit. A [`limits`](https://limits.readthedocs.io/en/stable/quickstart.html#rate-limit-string-notation) string such as `20/minute` or `100/hour`. |
| `ACCOUNT_RATE_LIMIT` | `20/hour` | Per-account and per-share-link limit. Same format. |
| `RATE_LIMIT_ENABLED` | `true` | Set to `false` to turn off auth rate limiting entirely. |
| `TRUST_FORWARDED_FOR` | `true` | Whether to read `X-Forwarded-For` (and `X-Forwarded-Proto`) from a trusted proxy. `false` ignores both headers from everyone. Recommended when Grimoire is exposed without a proxy and every client arrives from an address in the trust list; see below. |
| `TRUSTED_PROXIES` | loopback and Docker networks | Addresses or CIDR ranges whose `X-Forwarded-For` is read. Default `127.0.0.0/8,::1,172.16.0.0/12`. See below. |

### Behind a reverse proxy

`X-Forwarded-For` is only read from connections that come from a **trusted proxy**, and it is read from the right: each proxy appends the address it received the request from, so the right-most address that isn't itself a trusted proxy is the client. A direct connection from anywhere else is keyed on its own address, whatever header it sends, so a spoofed header can't sidestep the limit.

The default trust list - loopback plus `172.16.0.0/12`, the range Docker gives its bridge networks - covers a proxy or tunnel on the same host or in the same Docker Compose project. nginx, Nginx Proxy Manager, SWAG, Traefik, Caddy, Cloudflare Tunnel and `tailscale serve` need no setting. The same address is what **Active Sessions** shows for each device.

The list is matched on the connecting address, so **any** client connecting from those ranges is treated as a proxy and its `X-Forwarded-For` is believed: it can name any address in the header and is rate-limited and listed in Active Sessions under it. That is why the default stops at loopback and Docker's own networks rather than taking in your whole LAN. To narrow it further, set `TRUSTED_PROXIES` to just your proxy's address, or with no proxy set `TRUST_FORWARDED_FOR=false`.

- **Proxy on another machine** - another box on your LAN, a Kubernetes ingress, a Podman network (`10.88.0.0/16`, `10.89.0.0/16`, ...), or a VPS reaching your server over the internet: add its address (or its network's range) to `TRUSTED_PROXIES`. Until you do, every client behind it shares one bucket and shows up in Active Sessions under the proxy's address, and Grimoire logs a one-time line naming the private address whose `X-Forwarded-For` it ignored.
- **Cloudflare's proxy (orange cloud) in front of your own reverse proxy**, without Cloudflare Tunnel: requests reach your proxy from Cloudflare's addresses, so add [Cloudflare's IP ranges](https://www.cloudflare.com/ips/) to `TRUSTED_PROXIES`, keeping the default ranges. Until you do, clients share Cloudflare's addresses for rate limiting and Active Sessions shows them. Cloudflare Tunnel needs nothing.
- **No proxy at all:** nothing has to be set. Set `TRUST_FORWARDED_FOR=false` so no client can name its own address.
- **A port forwarder that hides client addresses** - Docker Desktop, Docker's userland proxy, or rootless Podman or Docker using rootlesskit - exposed directly: the forwarder replaces each client's address with an internal one. If that address falls in the trust list (every client then shows up in Active Sessions under the same `172.x` address), every client - including one reaching the published port from the internet - counts as a trusted proxy and can claim a fresh address on each request. **Set `TRUST_FORWARDED_FOR=false`** on these setups. All clients then share one per-IP bucket and Active Sessions shows the internal address; the per-account and per-link limits above still apply. To see real addresses instead, put a reverse proxy or tunnel in front and keep the header trusted (ideally with `TRUSTED_PROXIES` narrowed to the proxy), or on recent Podman set `rootless_port_forwarder = "pasta"` in `containers.conf`, which keeps the client's own address.

```yaml
environment:
  # Keep the defaults and add a proxy on another machine
  - TRUSTED_PROXIES=127.0.0.0/8,::1,172.16.0.0/12,203.0.113.10
```

### Multiple replicas

When [`VALKEY_URL`](/configuration/env-vars#optional-features) is set, the limit counters are shared through Valkey, so the limit is enforced consistently across all workers and replicas. Without it, each process keeps its own in-memory counters. The limiter also falls back to in-memory automatically if Valkey becomes unreachable.

## Password rules

New passwords have to meet two rules, wherever they are set: first-run setup, creating a user, an admin reset, changing your own password, converting a guest to a user, and creating or editing a password-protected [share link](/guide/sharing).

- **A minimum length:** **12** characters for account passwords and **8** for share-link passwords.
- **Not a commonly used password**, at any length. The list ships with Grimoire and is checked on your server, so nothing about the password leaves it.

Signing in never checks these rules, so passwords set before they existed keep working until their owner changes them. The forms in the app show the minimum your instance uses.

| Variable | Default | Description |
|---|---|---|
| `PASSWORD_MIN_LENGTH` | `12` | Minimum length for new account passwords. Lowering it below the default logs a warning at start-up; only do that on a server the internet can't reach. |
| `SHARE_LINK_PASSWORD_MIN_LENGTH` | `8` | Minimum length for new share-link passwords. |

## Sessions

Signing out a device under **Active Sessions**, an admin changing your role or resetting your password, and deleting an account all end sessions. A session's refresh token stops working at once, but an access token already issued for an ordinary request stays valid until it expires - up to `ACCESS_TOKEN_EXPIRE_MINUTES` (30 by default).

Requests from **GM and admin** accounts are the exception: they check the account's current role, and whether their session has ended, on every request, so a role change, a deleted account or a signed-out device takes effect for those roles straight away.

A [guest's](/guide/guest-access) session ends once guest access is turned off or the guest is removed, and deleting a campaign removes its guest accounts.

## Secret key

`SECRET_KEY` signs every login session. Leave it unset and Grimoire generates a random one on first start and keeps it in the data volume. If you set it yourself, use a private random value of at least 32 characters, such as the output of `openssl rand -hex 32`.

Grimoire **refuses to start** with a key that has appeared in its own documentation or example files (`change-me` and the like), because anyone who knows the key can sign in as any user. A key shorter than 32 characters still starts, with a warning in the log. See the [FAQ](/faq#grimoire-won-t-start-and-the-log-mentions-secret-key) if you hit the error.

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
