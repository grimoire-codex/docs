# Environment Variables

## Signing key

| Variable | Description |
|---|---|
| `SECRET_KEY` | JWT signing secret. Leave it unset and Grimoire generates one on first start and keeps it in the data volume; set it yourself (at least 32 characters, e.g. `openssl rand -hex 32`) if you run more than one replica without a shared data volume. Grimoire refuses to start on a value published in its docs or example files, and warns about one shorter than 32 characters - see [Secret key](/configuration/security#secret-key). It also keys the stored OPDS, calendar and share-link tokens: changing it signs everyone out and ends every feed URL and share link, which then have to be made again. |

## Core

| Variable | Default | Description |
|---|---|---|
| `WORKERS` | `1` | Number of worker processes. One serves a typical gaming table; each extra worker is another full copy of the app (roughly 125-190 MB). See [Performance → Workers](/configuration/performance#workers) for when to raise it to `2`. |
| `LIBRARY_PATH` | `/app/library` | Path to the library directory inside the container. |
| `DATA_PATH` | `/app/data` | Path for the database, thumbnails, and search cache inside the container. |
| `BASE_URL` | `http://localhost:9481` | Public base URL of this instance. Set to your external URL when running behind a reverse proxy. An `https://` value marks every sign-in cookie HTTPS-only (without it, a cookie is marked so only when its request arrived over HTTPS or through a trusted proxy reporting `X-Forwarded-Proto: https`). Also used for OPDS feed links, calendar links and OIDC redirect URIs. |
| `LOG_LEVEL` | `info` | Console log verbosity: `debug`, `info`, `warning`, `error`, `critical`. The in-app log (**Settings → Logs**) always captures `debug`-level entries regardless of this setting. |
| `TZ` | `UTC` | Timezone for all log timestamps (console output and the in-app log under **Settings → Logs**). Use an IANA zone name such as `America/Toronto` or `Europe/Berlin`. Defaults to UTC when unset; an unknown zone name logs a warning and uses UTC. |
| `PUID` / `PGID` | _owner of the library_ | User and group ids Grimoire runs as inside the container. Unset, it runs as the owner of the mounted library, then of the data folder, then `1000:1000`, and stays root only if that user couldn't reach something root could. Set them (e.g. `99`/`100` on Unraid) to choose; `PUID=0` keeps root. Ignored when the container is started with `--user`. See [File ownership](/configuration/volumes#file-ownership). |
| `UMASK` | _inherited_ | Octal umask applied to everything Grimoire writes: uploaded files, exported sidecars, thumbnails, and the database. `022` gives `rw-r--r--`, `002` group-writable, `000` `rw-rw-rw-` (what Unraid setups usually want). Useful where you cannot set the process umask yourself, such as Kubernetes. `GRIMOIRE_UMASK` is accepted as an alias. Left unset, Grimoire keeps whatever umask it inherited; an unparseable value is ignored with a warning rather than guessed at. |

## Optional features

| Variable | Default | Description |
|---|---|---|
| `VALKEY_URL` | _none_ | Redis-compatible cache URL (e.g. `redis://:<password>@valkey:6379/0`). Holds rendered page and map images, sign-in rate-limit and lockout counters, OIDC sign-in state and scan status; falls back to the disk cache and per-worker memory when unset. Give Valkey a password (`--requirepass`) and put it in the URL; it is redacted from the logs. See [Security](/configuration/security). |
| `OPDS_ENABLED` | `false` | Set to `true` to enable the OPDS catalog. |
| `MCP_ENABLED` | `false` | Set to `true` to serve the MCP endpoint at `/mcp`, so AI assistants such as Claude Code can use your library. It authenticates with API keys, so `API_KEYS_ENABLED` must stay on. See [MCP](/guide/mcp). |
| `MCP_ALLOWED_HOSTS` | _none_ | Extra host names `/mcp` answers under, comma-separated (`nas.lan`, `192.168.1.20:9481`; a bare name allows any port). Loopback and `BASE_URL`'s host are always allowed. Set it when an assistant reaches Grimoire at another address, such as a LAN IP. |
| `API_DOCS` | `signed-in` | Who may open `/api/docs`, `/api/redoc` and `/api/openapi.json`: `signed-in`, `public`, or `off`. |
| `DISABLE_VERSION_CHECKING` | `false` | Set to `true` to disable the "update available" check. When off, Grimoire proxies GitHub's releases API server-side (cached ~1h) so the sidebar can show when a newer release exists; when on, no outbound request to GitHub is ever made and no update banner appears. |

## Grimoire Codex DB

Grimoire Codex DB is off and hidden unless `GRIMOIRE_CODEX_DB_API_TOKEN` is set. The other variables pin a setting from **Settings → Metadata → Grimoire Codex DB** and show it read-only there. See [Grimoire Codex DB](/guide/codex).

| Variable | Description |
|---|---|
| `GRIMOIRE_CODEX_DB_API_TOKEN` | A `cdx_…` token from a Grimoire Codex DB account (**Account → API tokens**). Turns Grimoire Codex DB on; without it nothing about Grimoire Codex DB is shown. Every request uses it, and records sent are credited to its account. Never stored or shown by Grimoire. |
| `GRIMOIRE_CODEX_DB_URL` | The Grimoire Codex DB address, for a self-hosted Grimoire Codex DB. Defaults to `https://db.grimoirecodex.org`. |
| `GRIMOIRE_CODEX_DB_SEND_HASHES` | `true` or `false`. Whether book lookups and submissions include the file's SHA-256. Off by default; `true` is recommended for exact matches. |

## Add-ons

| Variable | Description |
|---|---|
| `ADDON_INDEX_CREDENTIALS` | Access tokens for [private add-on sources](/guide/addons#using-a-private-repository), as JSON mapping an index URL to a token, or to `{"token": "...", "header": "PRIVATE-TOKEN"}` for a host that wants another header. A token is sent only to files inside its source. A source set here shows its token read-only in **Settings → Add-ons**; others can still be given one there. Example: `ADDON_INDEX_CREDENTIALS='{"https://raw.githubusercontent.com/our-table/packs/main/index.json": "github_pat_..."}'` |

## Backups

All four are configurable in the UI under **Settings → Backups**. Setting one here pins it: the value wins and the field is read-only in Settings. See [Backups](/configuration/backups).

| Variable | Default | Description |
|---|---|---|
| `BACKUP_DIR` | `DATA_PATH/backups` | Where backup archives are written. Point at another mounted volume to keep backups off the main disk. Set here, the container hands the folder to the user Grimoire runs as on each start; a folder chosen in Settings instead is only checked, and the log says if Grimoire can't write to it. |
| `BACKUP_SCHEDULE` | `off` | `off`, `hourly`, `daily`, or `weekly`. |
| `BACKUP_RETENTION_COUNT` | `0` | Keep at most this many backups, deleting oldest-first. `0` = unlimited. |
| `BACKUP_RETENTION_GB` | `0` | Keep at most this many gigabytes of backups in total, deleting oldest-first. `0` = unlimited. |

Backups cover the database and the files you uploaded through Grimoire, but **not your library**, which you should back up separately.

## Library scanning

| Variable | Description |
|---|---|
| `DISABLE_FOLDER_CATEGORY_INFERENCE` | `true` or `false`. When set, pins folder-name category inference on or off, overriding the in-app toggle (which is shown read-only). When `true`, books are not auto-assigned a category from their folder names and fall back to `uncategorized`. Leave unset to control it from **Settings → Library → Files & folders**. To disable inference for a single game system only, drop an empty `.no-auto-category` file at that system's folder root. |
| `LIBRARY_WATCH` | `true` or `false`. Pins whether Grimoire [watches the library for changes](/guide/library-structure#watching-for-changes) and rescans by itself, overriding the toggle in **Settings → Library → Scanning** (shown read-only). Unset, the toggle applies, and it is on by default. |
| `LIBRARY_WATCH_MODE` | `auto`, `events`, or `poll`: how changes are noticed. `events` uses the operating system's file notifications - instant, but they only cover changes made on the machine Grimoire runs on. `poll` looks through the library every `LIBRARY_WATCH_POLL_INTERVAL` seconds and works on any network share. `auto` (the default) polls on NFS, SMB/CIFS and FUSE mounts and uses events elsewhere. |
| `LIBRARY_WATCH_DELAY` | Default `10`. Seconds the library has to stay quiet after a change before it is scanned, so a folder still being copied is scanned once, when it is done. A library that never goes quiet is still scanned five minutes after the first change. |
| `LIBRARY_WATCH_POLL_INTERVAL` | Default `120`, minimum `5`. In poll mode, seconds between checks. Each check lists every folder and reads every file's size and date, so raise it for a large library on a slow share. |
| `MAX_IMAGE_PIXELS` | Unset by default. Raises the size limit above which an image is refused as a possible decompression bomb. Pillow warns above this many pixels and refuses above twice it. Its own default (warn at ~89 MP, refuse at ~179 MP) is sized for untrusted uploads, so a very large scanned battlemap - 60x60in at 300 DPI is 324 MP - is indexed with no thumbnail. Set it only if you have images that large and trust where they came from, e.g. `400000000`. Values below `1000000` are raised to it; `0`, a negative number, or anything unparseable counts as unset. See [Performance → Very large images](/configuration/performance#very-large-images). |

## Memory

Grimoire returns the memory it used to render pages once reading slows down. These tune when. See [Performance → Memory use](/configuration/performance#memory-use).

| Variable | Default | Description |
|---|---|---|
| `PAGE_RECLAIM_INTERVAL` | `10` | After this many page renders, the memory used to decode page images is handed back to the operating system. Lower (e.g. `3`) keeps peaks down while several people read at once, at a few percent of render speed. `0` turns it off. |
| `PAGE_IDLE_RELEASE` | `300` | Seconds with no page views after which open books are closed and render memory is handed back, so the server drops back to its idle size after a session. Books reopen on the next page view. `0` turns it off. |

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
| `API_KEYS_ENABLED` | Set to `false` to turn [API keys](/api#api-keys) off for the whole instance, admins included: existing keys stop working and the key menus disappear. On by default. Which users may hold keys is set per user in Settings → People. |
| `CORS_ALLOWED_ORIGINS` | Comma-separated origins, e.g. `https://foundry.example.com`, whose web pages may call the API from the browser: a Foundry VTT module, a browser extension, a dashboard. Unset by default, which sends no CORS headers. Entries must be exact origins (`scheme://host[:port]`); a wildcard or a path is ignored with a warning. Server-side tools like Homepage don't need it. See [API → Calling the API from a browser](/api#calling-the-api-from-a-browser). |
| `GUEST_ACCESS_ENABLED` | `true` or `false`. Pins guest invite codes on or off, overriding the in-app toggle (shown read-only). Off by default. See [Guest Access](/guide/guest-access). |
| `SHARE_LINKS_ENABLED` | Set to `false` to turn [share links](/guide/sharing) off for the whole instance: every existing link stops working and the Share buttons disappear. The links are kept, so turning it back on restores them. On by default. Who may create links is set by role in Settings → Access & sharing and per user in Settings → People. |

## Security

Rate limiting on the credential-checking endpoints (per IP, per account and per share link), password rules, and security headers. See [Security](/configuration/security) for the full explanation.

| Variable | Default | Description |
|---|---|---|
| `AUTH_RATE_LIMIT` | `10/minute` | Per-IP throttle on `/api/auth/login`, `/api/auth/setup`, `/api/auth/guest-login`, and on failed API key attempts. Exceeding it returns `429`. Uses a [`limits`](https://limits.readthedocs.io/en/stable/quickstart.html#rate-limit-string-notation) string like `20/minute` or `100/hour`. |
| `ACCOUNT_RATE_LIMIT` | `20/hour` | Failed sign-ins counted per account (by the username typed) and wrong share-link passwords counted per link, whatever address they come from. A browser that has signed in to the account before keeps its own count, so failures elsewhere don't lock its owner out. Same format as `AUTH_RATE_LIMIT`. |
| `RATE_LIMIT_ENABLED` | `true` | Set to `false` to disable auth rate limiting entirely. |
| `TRUST_FORWARDED_FOR` | `true` | When `true`, `X-Forwarded-For` is read from connections that come from a trusted proxy (`TRUSTED_PROXIES`), so each client behind the proxy gets its own rate-limit bucket and shows its own address under Active Sessions. Set to `false` to ignore the header - and `X-Forwarded-Proto` - from everyone; recommended on Docker Desktop and rootless Podman or Docker exposed without a proxy, where every client arrives from a private address. Behind a TLS proxy, set `BASE_URL` to your `https://` address if you turn it off. |
| `TRUSTED_PROXIES` | loopback and private ranges | Comma-separated addresses or CIDR ranges whose `X-Forwarded-For` is read. The default (`127.0.0.0/8,::1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,fc00::/7`) covers a proxy or tunnel on the same host, Docker network or LAN - nginx, Nginx Proxy Manager, SWAG, Traefik, Caddy, Cloudflare Tunnel, Tailscale - with nothing to set. Add your proxy's address if it reaches Grimoire from a public IP. Any client connecting from these ranges is believed too, so a LAN device can name its own address. See [Behind a reverse proxy](/configuration/security#behind-a-reverse-proxy). |
| `PASSWORD_MIN_LENGTH` | `12` | Minimum length for new account passwords (first-run setup, new users, resets, password changes). Passwords set earlier keep working. Commonly used passwords are refused at any length. Lowering it below the default logs a warning at start-up. See [Password rules](/configuration/security#password-rules). |
| `SHARE_LINK_PASSWORD_MIN_LENGTH` | `8` | Minimum length for new share-link passwords. Commonly used passwords are refused at any length. |

## OIDC

Each OIDC setting can be pinned via environment variable. Pinned values are shown read-only in **Settings → Sign-in**.

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
| `OIDC_SIGNING_ALG` | One of `RS256`/`RS384`/`RS512`/`ES256`/`ES384`/`ES512`/`PS256`/`PS384`/`PS512`. Default `RS256`. ID tokens signed with any other algorithm are refused. |
| `OIDC_BUTTON_TEXT` | Label for the SSO button on the login page. |
| `OIDC_BUTTON_BG_COLOR` | Button background, a CSS hex color such as `#131314`. Invalid values are ignored with a log warning. |
| `OIDC_BUTTON_TEXT_COLOR` | Button text color, CSS hex. |
| `OIDC_BUTTON_BORDER_COLOR` | Button border color, CSS hex. |
| `OIDC_BUTTON_RADIUS` | Button corner radius in px, an integer from 0 to 40. |
| `OIDC_BUTTON_ICON` | Path to an image file inside the container (PNG, JPEG, WebP, GIF or SVG, up to 1 MB) used as the button icon. It is normalized to a PNG like an upload. Set it empty to pin "no icon". See [Style the sign-in button](/guide/oidc#style-the-sign-in-button). |
| `OIDC_GROUPS_CLAIM` | Name of the claim containing group memberships. |
| `OIDC_ADMIN_GROUP` | The group that grants the admin role, matched by exact name (ignoring case). Default `admin`. |
| `OIDC_GM_GROUP` | The group that grants the GM role. Default `gm`. |
| `OIDC_PLAYER_GROUP` | The group that grants the player role. Default `player`. |
| `OIDC_PERMISSIONS_CLAIM` | Name of the claim containing a permissions object. |
| `OIDC_MATCH_BY` | `none`, `email`, or `username`: how to link existing accounts on first OIDC login. |
| `OIDC_AUTO_LAUNCH` | Automatically redirect to the IdP when visiting `/login`. |
| `OIDC_AUTO_REGISTER` | Automatically create local accounts on first OIDC login. |
