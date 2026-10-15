# Environment Variables

## Required

| Variable | Description |
|---|---|
| `SECRET_KEY` | **Required.** JWT signing secret. Generate with `openssl rand -hex 32`. |

## Core

| Variable | Default | Description |
|---|---|---|
| `WORKERS` | `2` | Number of uvicorn worker processes. 2–4 is typical; more workers = more memory. |
| `LIBRARY_PATH` | `/app/library` | Path to the library directory inside the container. |
| `DATA_PATH` | `/app/data` | Path for the database, thumbnails, and search cache inside the container. |
| `BASE_URL` | `http://localhost:9481` | Public base URL of this instance. Set to your external URL when running behind a reverse proxy; used for OPDS feed links and OIDC redirect URIs. |
| `LOG_LEVEL` | `info` | Console log verbosity: `debug`, `info`, `warning`, `error`, `critical`. The in-app Logs tab always captures `debug`-level entries regardless of this setting. |
| `TZ` | `UTC` | Timezone for all log timestamps (console output and the in-app Logs tab). Use an IANA zone name such as `America/Toronto` or `Europe/Berlin`. Defaults to UTC when unset; an unknown zone name logs a warning and uses UTC. |
| `UMASK` | _inherited_ | Octal umask applied to everything Grimoire writes: uploaded files, exported sidecars, thumbnails, and the database. `022` gives `rw-r--r--`, `002` group-writable, `000` `rw-rw-rw-` (what Unraid setups usually want). Useful where you cannot set the process umask yourself, such as Kubernetes. `GRIMOIRE_UMASK` is accepted as an alias. Left unset, Grimoire keeps whatever umask it inherited; an unparseable value is ignored with a warning rather than guessed at. |

## Optional features

| Variable | Default | Description |
|---|---|---|
| `VALKEY_URL` | _none_ | Redis-compatible cache URL for rendered page images (e.g. `redis://valkey:6379/0`). Falls back to disk cache when unset. Also shares auth rate-limit counters across replicas; see [Security](/configuration/security). |
| `OPDS_ENABLED` | `false` | Set to `true` to enable the OPDS catalog. |
| `MCP_ENABLED` | `false` | Set to `true` to serve the MCP endpoint at `/mcp`, so AI assistants such as Claude Code can use your library. It authenticates with API keys, so `API_KEYS_ENABLED` must stay on. See [MCP](/guide/mcp). |
| `DISABLE_VERSION_CHECKING` | `false` | Set to `true` to disable the "update available" check. When off, Grimoire proxies GitHub's releases API server-side (cached ~1h) so the sidebar can show when a newer release exists; when on, no outbound request to GitHub is ever made and no update banner appears. |

## Grimoire Codex DB

Grimoire Codex DB is off and hidden unless `GRIMOIRE_CODEX_DB_API_TOKEN` is set. The other variables pin a setting from **Settings → Add-ons → Grimoire Codex DB** and show it read-only there. See [Grimoire Codex DB](/guide/codex).

| Variable | Description |
|---|---|
| `GRIMOIRE_CODEX_DB_API_TOKEN` | A `cdx_…` token from a Grimoire Codex DB account (**Account → API tokens**). Turns Grimoire Codex DB on; without it nothing about Grimoire Codex DB is shown. Every request uses it, and records sent are credited to its account. Never stored or shown by Grimoire. |
| `GRIMOIRE_CODEX_DB_URL` | The Grimoire Codex DB address, for a self-hosted Grimoire Codex DB. Defaults to `https://db.grimoirecodex.org`. |
| `GRIMOIRE_CODEX_DB_SEND_HASHES` | `true` or `false`. Whether book lookups and submissions include the file's SHA-256. Off by default; `true` is recommended for exact matches. |

## Backups

All four are configurable in the UI under **Settings → Maintenance → Backups**. Setting one here pins it: the value wins and the field is read-only in Settings. See [Backups](/configuration/backups).

| Variable | Default | Description |
|---|---|---|
| `BACKUP_DIR` | `DATA_PATH/backups` | Where backup archives are written. Point at another mounted volume to keep backups off the main disk. |
| `BACKUP_SCHEDULE` | `off` | `off`, `hourly`, `daily`, or `weekly`. |
| `BACKUP_RETENTION_COUNT` | `0` | Keep at most this many backups, deleting oldest-first. `0` = unlimited. |
| `BACKUP_RETENTION_GB` | `0` | Keep at most this many gigabytes of backups in total, deleting oldest-first. `0` = unlimited. |

Backups cover the database and the files you uploaded through Grimoire, but **not your library**, which you should back up separately.

## Library scanning

| Variable | Description |
|---|---|
| `DISABLE_FOLDER_CATEGORY_INFERENCE` | `true` or `false`. When set, pins folder-name category inference on or off, overriding the in-app toggle (which is shown read-only). When `true`, books are not auto-assigned a category from their folder names and fall back to `uncategorized`. Leave unset to control it from **Settings → Maintenance**. To disable inference for a single game system only, drop an empty `.no-auto-category` file at that system's folder root. |
| `MAX_IMAGE_PIXELS` | Unset by default. Raises the size limit above which an image is refused as a possible decompression bomb. Pillow warns above this many pixels and refuses above twice it. Its own default (warn at ~89 MP, refuse at ~179 MP) is sized for untrusted uploads, so a very large scanned battlemap - 60x60in at 300 DPI is 324 MP - is indexed with no thumbnail. Set it only if you have images that large and trust where they came from, e.g. `400000000`. Values below `1000000` are raised to it; `0`, a negative number, or anything unparseable counts as unset. See [Performance → Very large images](/configuration/performance#very-large-images). |

## OCR

Image-only PDFs (scanned books with no embedded text) can be run through the bundled Tesseract engine so their text is added to the search index. See [Performance → OCR](/configuration/performance#ocr).

| Variable | Default | Description |
|---|---|---|
| `OCR_ENABLED` | `true` | Set to `false` to disable OCR of image-only PDFs even on the OCR-capable image. Automatically off on `-slim` images (which omit Tesseract). |
| `OCR_LANGUAGES` | `eng` | `+`-joined Tesseract language codes, e.g. `eng` or `eng+deu+fra`. Extra languages require their `.traineddata` files to be present in the image's tessdata directory. |
| `OCR_CONCURRENCY` | `1` | Number of scanned books OCR'd in parallel by the background OCR worker. Raise on multi-core hosts with spare CPU/RAM; keep at `1` on small devices. Set to `0` to turn OCR off entirely (same effect as `OCR_ENABLED=false`) — a runtime off switch for hosts hitting repeated OCR errors or out-of-memory kills. |
| `OCR_DPI` | `150` | Resolution (clamped 72–600) scanned pages are rasterized at before OCR. Higher improves recognition on faint scans but is slower and uses more memory per page; lower is faster and lighter. |
| `OCR_PAGE_TIMEOUT` | `120` | Seconds a single page may take to OCR before it is skipped and the book continues to the next page. Raise it on low-power hardware, where a dense or noisy scan can legitimately need several minutes per page; `0` means no limit. Skipped pages are not searchable — see [Performance → When a book is only partly read](/configuration/performance#when-a-book-is-only-partly-read). |

## Authentication

| Variable | Description |
|---|---|
| `ALLOW_PASSWORD_AUTHENTICATION` | `true` or `false`. Pins password authentication on or off, overriding the in-app toggle (which is shown read-only). First-run admin setup always requires a password regardless of this value. |
| `API_KEYS_ENABLED` | Set to `false` to turn [API keys](/api#api-keys) off for the whole instance, admins included: existing keys stop working and the key menus disappear. On by default. Which users may hold keys is set per user in Settings → Users. |
| `CORS_ALLOWED_ORIGINS` | Comma-separated origins, e.g. `https://foundry.example.com`, whose web pages may call the API from the browser: a Foundry VTT module, a browser extension, a dashboard. Unset by default, which sends no CORS headers. Entries must be exact origins (`scheme://host[:port]`); a wildcard or a path is ignored with a warning. Server-side tools like Homepage don't need it. See [API → Calling the API from a browser](/api#calling-the-api-from-a-browser). |
| `GUEST_ACCESS_ENABLED` | `true` or `false`. Pins guest invite codes on or off, overriding the in-app toggle (shown read-only). Off by default. See [Guest Access](/guide/guest-access). |
| `SHARE_LINKS_ENABLED` | Set to `false` to turn [share links](/guide/sharing) off for the whole instance: every existing link stops working and the Share buttons disappear. The links are kept, so turning it back on restores them. On by default. Who may create links is set in Settings → Users. |

## Security

Per-IP rate limiting on the credential-checking endpoints, plus security headers. See [Security](/configuration/security) for the full explanation.

| Variable | Default | Description |
|---|---|---|
| `AUTH_RATE_LIMIT` | `10/minute` | Per-IP throttle on `/api/auth/login`, `/api/auth/setup`, `/api/auth/guest-login`, and on failed API key attempts. Exceeding it returns `429`. Uses a [`limits`](https://limits.readthedocs.io/en/stable/quickstart.html#rate-limit-string-notation) string like `20/minute` or `100/hour`. |
| `RATE_LIMIT_ENABLED` | `true` | Set to `false` to disable auth rate limiting entirely. |
| `TRUST_FORWARDED_FOR` | `true` | When `true`, the limiter keys on the left-most `X-Forwarded-For` address so each client gets its own bucket behind a reverse proxy. Set to `false` only if Grimoire is exposed directly, so a spoofed header can't sidestep the limit. |

## OIDC

Each OIDC setting can be pinned via environment variable. Pinned values are shown read-only in **Settings → Authentication**.

| Variable | Description |
|---|---|
| `OIDC_ENABLED` | Master toggle for OIDC sign-in. |
| `OIDC_ISSUER_URL` | Base URL of the IdP. |
| `OIDC_TOKEN_ISSUER` | Exact `iss` value in tokens. Leave blank to auto-detect. |
| `OIDC_AUTHORIZATION_ENDPOINT` | Authorization endpoint URL. |
| `OIDC_TOKEN_ENDPOINT` | Token endpoint URL. |
| `OIDC_USERINFO_ENDPOINT` | Userinfo endpoint URL. |
| `OIDC_JWKS_URI` | JWKS URL for ID token signature validation. |
| `OIDC_END_SESSION_ENDPOINT` | Optional RP-initiated logout endpoint. |
| `OIDC_CLIENT_ID` | Client ID issued by the IdP. |
| `OIDC_CLIENT_SECRET` | Client secret issued by the IdP. |
| `OIDC_SIGNING_ALG` | One of `RS256`/`RS384`/`RS512`/`ES256`/`ES384`/`ES512`/`PS256`/`PS384`/`PS512`/`HS256`. Default `RS256`. |
| `OIDC_BUTTON_TEXT` | Label for the SSO button on the login page. |
| `OIDC_BUTTON_BG_COLOR` | Button background, a CSS hex color such as `#131314`. Invalid values are ignored with a log warning. |
| `OIDC_BUTTON_TEXT_COLOR` | Button text color, CSS hex. |
| `OIDC_BUTTON_BORDER_COLOR` | Button border color, CSS hex. |
| `OIDC_BUTTON_RADIUS` | Button corner radius in px, an integer from 0 to 40. |
| `OIDC_BUTTON_ICON` | Path to an image file inside the container (PNG, JPEG, WebP, GIF or SVG, up to 1 MB) used as the button icon. It is normalized to a PNG like an upload. Set it empty to pin "no icon". See [Style the sign-in button](/guide/oidc#style-the-sign-in-button). |
| `OIDC_GROUPS_CLAIM` | Name of the claim containing group memberships. |
| `OIDC_PERMISSIONS_CLAIM` | Name of the claim containing a permissions object. |
| `OIDC_MATCH_BY` | `none`, `email`, or `username`: how to link existing accounts on first OIDC login. |
| `OIDC_AUTO_LAUNCH` | Automatically redirect to the IdP when visiting `/login`. |
| `OIDC_AUTO_REGISTER` | Automatically create local accounts on first OIDC login. |
