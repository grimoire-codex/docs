# API Reference

This reference follows the current release. A running server reports its own version at `GET /api/about` and in its OpenAPI schema (`info.version`).

## Interactive docs

With the server running, the API is self-documented via OpenAPI:

| URL | Description |
|---|---|
| `http://localhost:9481/api/docs` | Swagger UI: interactive, try-it-out docs |
| `http://localhost:9481/api/redoc` | ReDoc: clean, readable reference |
| `http://localhost:9481/api/openapi.json` | Raw OpenAPI schema |

These need a signed-in session by default (open them in a browser where you're signed in). Set [`API_DOCS`](/configuration/env-vars) to `public` to serve them to anyone, or `off` to turn them off.

An `/api` address that no endpoint takes returns a JSON `404` (`{"detail": "Not Found"}`), and one called with a method it doesn't take returns a `405` with an `Allow` header. Neither returns the web app's page. The same goes for `/mcp` and `/opds` when they are turned off. These prefixes are matched ignoring case and extra leading slashes, so `/API/x` and `//api/x` get the same JSON `404`.

---

## Authentication

All endpoints except `/api/auth/status`, `/api/auth/setup`, `/api/auth/login`, and `/api/auth/config` require a JWT, or, for external integrations, an [API key](#api-keys).

**Header** (preferred):
```
Authorization: Bearer <token>
```

**Session cookie** (browser images and downloads): sign-in also sets an HttpOnly `grimoire_session` cookie, which authenticates `GET` requests. A token in the URL (`?token=`) is not accepted.

Access tokens are returned by `/api/auth/login` and expire after **30 minutes**; the app renews them from an HttpOnly refresh cookie.

### Roles

| Role | Permissions |
|---|---|
| `admin` | Full access including user management and app settings |
| `gm` | Read + edit metadata, rescan a single book, manage campaigns |
| `player` | Read-only access |

### API keys

Scripts and tools, such as a [Homepage widget](/guide/homepage), [grimoire-cli](https://github.com/thomaslazar/grimoire-cli), or a backup script, authenticate with an **API key** instead of a login:

```
X-API-Key: grim_...
```

**Keys are personal.** Every key belongs to a user and acts **as that user**: their role, their campaigns, their favorites. The key's permissions can only narrow that; a key can never do more than its owner. If the owner's role changes, their keys follow. Keys are deleted with their owner.

Create keys under **Settings → Connections → API Keys**. Admins always can. Anyone else can once an admin ticks **API keys** on their row in **Settings → People** (off by default), or your [OIDC provider](/guide/oidc) grants the `apiKeys` permission; taking it away stops their existing keys working. Admins can see and revoke everyone's keys under **Settings → Access & sharing → All API Keys**. Guests never have keys.

Setting `API_KEYS_ENABLED=false` turns API keys off for the whole instance, admins included: keys stop working and the key menus disappear.

Each key has a name, an optional expiry (30, 90 or 365 days, or never), and a level per area:

| Level | Allows |
|---|---|
| **No access** (default) | Nothing |
| **Read** | Viewing: `GET` requests, plus a few read-only `POST`s such as metadata search |
| **Read and write** | Everything in that area your account can do, including changes |

**All permissions** sets one level for every area at once, including areas added in future versions. You can still raise individual areas above it.

| Permission | Covers |
|---|---|
| Statistics | `/api/stats`: library counts and totals |
| Library | Scan status, start or cancel a rescan, clean up missing items, version and changelog |
| Books | Books and their metadata, covers, pages and files |
| Game systems | Game systems, their covers and book folders |
| Search | Full-text search |
| Tags | Create, rename, merge and delete tags, and list tagged items |
| Lookup lists | Genres, system families, parent systems, licenses and dice/materials |
| Maps | Maps and map folders |
| Tokens | Tokens, token folders and token frames |
| Audio | Audio tracks, their covers and folders |
| 3D models | 3D models and model folders |
| Share links | Your public [share links](#share-links): create, list and revoke |
| Campaigns | Your campaigns: members, sessions, wiki, resources and schedule |
| Characters | Character sheets and the characters built on them, the content catalogue, and rulesets |
| Downloads | Zip archive downloads |
| File manager | Browse, upload, move, rename and delete library files |
| Duplicates | Find, compare, link and merge duplicates |
| Add-ons | Community metadata add-ons |
| Maintenance | Metadata sidecar settings and export |
| Backups | Backups and the backup schedule |
| Logs | The application logs |
| Settings | App settings |
| Users | User accounts, and your own preferences |
| Personal | Your favorites, bookmarks, saved filters, pins, saved playlists and soundboards, and themes |
| Armies and leagues | Your army lists, the source books they cite, and the leagues you play in |

You're only offered what your role can use. A player sees game systems as Read at most, since players can't edit them, and never sees admin-only areas like Logs or the file manager.

A few things are **never available to a key**: signing in, managing API keys, and changing a credential or the account itself (password, deleting the account, OPDS and calendar feed tokens). Campaign and league guest invite codes (creating, listing, regenerating and sharing them), the OIDC sign-in button icon, the add-on source settings (`PATCH /api/addons/settings`), private source tokens (`/api/addons/sources/credential`) and the backup settings (`PUT /api/backups/settings`, which choose where on the server backups are written) are session-only too.

**Session-only fields.** Some routes a key can use take a few fields that only a signed-in session may set. Sent with a key they get a `403` naming the field; the rest of the route works as usual:

- `POST /api/users`: `password`, `email` and the `admin` role. A key can still create a player or GM with no password or email.
- `PATCH /api/users/:id`: `password`, a changed `email` and any change of role. Converting a guest to a user: `password` and the `admin` role.
- `PATCH /api/users/me/preferences`: a changed `email` or `allow_explicit`.
- `PATCH /api/settings`: password sign-in, guest access, the login message, and every OIDC setting.
- Approving an add-on's script, on install (`approve_script: true`) or afterwards (`script_approved: true`).

Campaign and league responses never include a guest's invite code when the request uses a key.

**Share links with a key.** Creating or changing a [share link](#share-links) with a key also needs the key to have **Read** on the item's own area - Books, Maps, Tokens, Audio or 3D models - besides **Share links: Read and write**.

**Shown once.** The full key appears only when it's created or regenerated. Grimoire stores just a hash of it and afterwards shows only its first few characters. A lost key can't be recovered, only regenerated.

**Errors.** An unknown or expired key gets `401`. A valid key without the permission a request needs gets `403`, with a message naming the permission and level it needs. A key also gets `403` once its owner loses API key access, or while `API_KEYS_ENABLED=false`. Once an address has sent too many wrong keys, further wrong keys from it get `429`; a working key is never refused (see [Security](/configuration/security#auth-rate-limiting)).

**MCP.** With `MCP_ENABLED=true`, keys also authenticate the [MCP server](/guide/mcp) at `POST /mcp`. Each MCP tool, resource and prompt checks the permission it needs, and a key is only shown the ones it can use.

**OAuth for MCP.** With `MCP_OAUTH_ENABLED=true` as well, a hosted MCP client such as claude.ai can sign each user in instead of being given a key. What the user approves is stored as an API key row - a **grant**, listed with `kind: "app"` - and the client sends `Authorization: Bearer <access token>` to `/mcp`, where it is checked exactly as a key with the same permissions would be. Access tokens work on `/mcp` only: on `/api` a Bearer token is always a session. The OAuth endpoints live at the site root:

| Endpoint | Method | Description |
|---|---|---|
| `/.well-known/oauth-protected-resource/mcp` | GET | Protected resource metadata (RFC 9728). A `401` from `/mcp` points here in `WWW-Authenticate` |
| `/.well-known/oauth-authorization-server` | GET | Authorization server metadata (RFC 8414) |
| `/register` | POST | Dynamic client registration (RFC 7591). A redirect address must be `https` (or `http` on loopback) and, with `MCP_OAUTH_REDIRECT_HOSTS` set, on one of those hosts |
| `/authorize` | GET, POST | Starts a sign-in (PKCE `S256` required) and sends the browser to the consent page |
| `/token` | POST | Exchanges a code or renews tokens. Access tokens last an hour, refresh tokens 30 days from their last use; each renewal replaces both |
| `/revoke` | POST | Revokes a grant by either of its tokens (RFC 7009) |

A refused access token gets `401` with `WWW-Authenticate: Bearer error="invalid_token"`, so the client renews it rather than starting a new sign-in. These endpoints share the sign-in rate limit.

### Calling the API from a browser

Server-side tools like Homepage and grimoire-cli need nothing extra. Code that runs in a web page on another site, such as a Foundry VTT module, a browser extension or a dashboard, is blocked by the browser unless you allow that site with `CORS_ALLOWED_ORIGINS`:

```yaml
environment:
  CORS_ALLOWED_ORIGINS: "https://foundry.example.com"
```

List each origin exactly (`scheme://host[:port]`), separated by commas. It's off by default. Allowed sites can send `Authorization`, `Content-Type` and `X-API-Key`, and can read `X-Token-Expired` and error responses. Cookies are never allowed cross-origin, so another site can't use a signed-in user's session: browser integrations authenticate with an API key, which its permissions already limit.

---

## Auth

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/auth/status` | GET | None | Returns `{initialized, password_min_length, share_link_password_min_length}`: whether first-run setup is done, and the minimum lengths for new account and share-link passwords |
| `/api/auth/config` | GET | None | Public auth config for the login screen |
| `/api/auth/setup` | POST | None | First-run admin account creation. Body: `{username, password}`. The password must meet the [password rules](/configuration/security#password-rules) |
| `/api/auth/login` | POST | None | Authenticate. Body: `{username, password}`. Returns `{token, user}`. Rate-limited per IP and per account; `429` once either is used up (see [Auth rate limiting](/configuration/security#auth-rate-limiting)) |
| `/api/auth/guest-login` | POST | None | Exchange a campaign or league guest's invite code for a token. Body: `{code}`. Returns `{token, user, campaign_id, league_id}`; `user.guest_scope` is `ttrpg` for a campaign's guest and `wargame` for a league's. `403` while guest access is off, `401` for an unknown or expired code |
| `/api/auth/me` | GET | any | Current user details, including `guest_scope` (`ttrpg`, `wargame`, or null for everyone but guests) and `sidebar_stats`, the user's own [sidebar stats layout](#sidebar-stats-layouts) (null when they follow the admin's default) |
| `/api/auth/openid/login` | GET | None | Start OIDC login, redirects to IdP. With no authorization endpoint set and none from discovery, redirects to `/login?oidc_error=authorization%20endpoint%20not%20configured` instead |
| `/api/auth/openid/callback` | GET | None | OIDC callback. A token or JWKS response that isn't a JSON object (an HTML error page, say) fails the sign-in with an error; a userinfo response like that is logged and ignored |
| `/api/auth/openid/discover` | POST | admin | Server-side discovery fetch. Body: `{issuer_url}`. `502` when the issuer can't be reached or doesn't answer with a JSON object |

---

## Users

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/users` | GET | admin | List all users |
| `/api/users` | POST | admin | Create a user. Body: `{username, password, role?, email?}`. The password must meet the [password rules](/configuration/security#password-rules). A username is trimmed and must be 2-100 characters with no control characters (`422` otherwise) - here, on guest conversion and at setup |
| `/api/users/:id` | PATCH | admin | Update `role`, `password`, `allow_explicit`, or `email` |
| `/api/users/:id` | DELETE | admin | Delete a user. The campaigns they own go with them, along with their uploads on disk - banner, files, members' art, tokens and sheets - as do their own art, token and sheet in campaigns they play in |
| `/api/users/me/preferences` | PATCH | any | Update own `display_name`, `allow_explicit`, `email`, or `sidebar_stats`: up to 10 `/api/stats` fields in order for your own [sidebar stats layout](#sidebar-stats-layouts), or `null` to follow the admin's default again. While [OIDC](/guide/oidc) matches existing users by email, only an admin can change an email |
| `/api/users/me/password` | PATCH | any | Change own password. Body: `{current_password, new_password}`. The new password must meet the [password rules](/configuration/security#password-rules). `400` for a wrong `current_password`; wrong ones are counted per account (`ACCOUNT_RATE_LIMIT`), separately from failed sign-ins, and get `429` past the limit. Ends every other session |
| `/api/users/me` | DELETE | any | Delete own account (admins can't); campaigns and uploads go as for `DELETE /api/users/:id` |

---

## Library

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/stats` | GET | JWT, or an API key with Statistics: Read | Counts, page totals and sizes, split into role-playing and wargame where that applies |
| `/api/scan-status` | GET | admin | Current scan state |
| `/api/rescan` | POST | admin | Trigger a background rescan and reindex |
| `/api/cancel-scan` | POST | admin | Gracefully stop the running scan |

**Stats response:** book counts and sizes cover only what the caller may see. A total is always its role-playing and wargame halves added up (`books` = `ttrpg_books` + `wargame_books`, and likewise for systems and pages). Item counts leave out variants, as the library views do, and the `*_variants` fields count them beside them; sizes include them. Sizes are in MB to one decimal place.

```json
{
  "game_systems": 14,
  "ttrpg_systems": 12,
  "wargame_systems": 2,
  "books": 380,
  "ttrpg_books": 340,
  "wargame_books": 40,
  "armory_books": 40,
  "maps": 1500,
  "tokens": 800,
  "audio": 250,
  "models": 640,
  "book_variants": 12,
  "map_variants": 1420,
  "token_variants": 35,
  "model_variants": 4,
  "audio_variants": 0,
  "indexed_books": 360,
  "total_pages": 51000,
  "ttrpg_pages": 45000,
  "wargame_pages": 6000,
  "total_size_mb": 20480.5,
  "ttrpg_size_mb": 18240.5,
  "wargame_size_mb": 2240.0,
  "maps_size_mb": 9120.4,
  "tokens_size_mb": 610.2,
  "models_size_mb": 10211.7,
  "audio_size_mb": 3467.4,
  "library_size_mb": 43890.2
}
```

To show library counts on an external dashboard, create an [API key](#api-keys) with **Statistics: Read** and nothing else. The counts are the key owner's, so use an admin's key to count the whole library. See the [Homepage Widget guide](/guide/homepage) for a full walkthrough.

**Homepage Custom API widget**: add this to your Homepage `services.yaml` ([Custom API widget docs](https://gethomepage.dev/widgets/services/customapi/)):

```yaml
- Grimoire:
    href: https://grimoire.example.com
    icon: grimoire.png
    widget:
      type: customapi
      url: https://grimoire.example.com/api/stats
      refreshInterval: 60000
      headers:
        X-API-Key: your-key-here
      mappings:
        - field: ttrpg_books
          label: RPG books
          format: number
        - field: wargame_books
          label: Wargame books
          format: number
        - field: maps
          label: Maps
          format: number
        - field: library_size_mb
          label: Size
          format: float
          scale: 0.001
          suffix: " GB"
```

Homepage shows up to four fields per row; pick the counts you care about from the fields below. A `refreshInterval` of 60s is plenty, since the counts only change when the library does.

| Field | Meaning | Suggested `format` |
|---|---|---|
| `game_systems` | Every game system: `ttrpg_systems` + `wargame_systems` | `number` |
| `ttrpg_systems` | Role-playing game systems (under `books/`) | `number` |
| `wargame_systems` | Wargame systems, in the [Armory](/guide/library-structure#armory) | `number` |
| `books` | Every book: `ttrpg_books` + `wargame_books` | `number` |
| `ttrpg_books` | Role-playing books | `number` |
| `wargame_books` | Wargame books, from the [Armory](/guide/library-structure#armory) | `number` |
| `armory_books` | The older name for `wargame_books`; always the same number | `number` |
| `maps` | Total maps | `number` |
| `tokens` | Total tokens | `number` |
| `audio` | Total audio tracks | `number` |
| `models` | Total 3D models | `number` |
| `book_variants` | Book variants (a printer-friendly cut, say), which `books` leaves out; `books` + `book_variants` is every book file | `number` |
| `map_variants` | Map variants (a gridless or night version), which `maps` leaves out | `number` |
| `token_variants` | Token variants, which `tokens` leaves out | `number` |
| `model_variants` | 3D model variants, which `models` leaves out | `number` |
| `audio_variants` | Audio variants, which `audio` leaves out | `number` |
| `indexed_books` | Books with a searchable full-text index | `number` |
| `total_pages` | Every book's pages: `ttrpg_pages` + `wargame_pages` | `number` |
| `ttrpg_pages` | Pages of the role-playing books | `number` |
| `wargame_pages` | Pages of the wargame books | `number` |
| `total_size_mb` | Size of the books in MB (books only): `ttrpg_size_mb` + `wargame_size_mb` | `float`, add `scale: 0.001` + `suffix: " GB"` to show GB |
| `ttrpg_size_mb` | Size of the role-playing books in MB | `float`, as above |
| `wargame_size_mb` | Size of the wargame books in MB | `float`, as above |
| `maps_size_mb` | Size of the maps in MB | `float`, as above |
| `tokens_size_mb` | Size of the tokens in MB | `float`, as above |
| `models_size_mb` | Size of the 3D models in MB | `float`, as above |
| `audio_size_mb` | Size of the audio in MB | `float`, as above |
| `library_size_mb` | Size of the whole library in MB: books plus maps, tokens, audio, and models | `float`, add `scale: 0.001` + `suffix: " GB"` to show GB |

### Sidebar stats layouts

The sidebar's footer shows up to 10 of these fields, in an order you choose; **All stats** under it (in the **More** drawer on a phone) opens every one. People choose their own under **Settings → Preferences → Sidebar stats**, and an admin sets everyone's default under **Settings → Sidebar**. A layout is a list of field names, and may name any field above except `armory_books`. The admin's layout (`sidebar_stats` in `PATCH /api/settings`, also returned by `/api/settings/ui`) is everyone's default; each user may save their own with `PATCH /api/users/me/preferences`, and `/api/auth/me` returns it. Unknown fields, repeats and more than 10 are refused with `400`.

The older `show_stat_*` settings still work. Until an admin saves a layout, the default is built from them in their old order, and each one reads as whether its stat is in the default layout. Setting one after a layout is saved adds its stat at the end or removes it.

---

## Game Systems

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/systems` | GET | any | List all systems with book counts. `game_type=ttrpg` lists the role-playing systems in `books/`, `game_type=wargame` the [Armory](/guide/library-structure#armory)'s; both when omitted. Each system carries its `game_type`. A system inside a container the caller cannot see lists at the top level for them, with empty `parent_id`/`parent_name` |
| `/api/systems/:id` | GET | any | System detail + full book list. `include_books=false` returns the summary alone. For a caller who cannot see an enclosing container, `scope_path` and the books' `relative_path` leave out that container's folder |
| `/api/systems/:id/books` | GET | any (not guest) | One page of the system's books matching the shelf filters. Query: filters, `category` + `folder` (one folder of the grouped shelf), `sort`, `order`, `limit`, `offset` |
| `/api/systems/:id/book-groups` | GET | any (not guest) | Every category and subfolder holding matching books, with counts |
| `/api/systems/:id/book-facets` | GET | any (not guest) | The tags, genres, product-code prefixes and categories on the shelf |
| `/api/systems/:id/factions` | GET | any (not guest) | A wargame system's [factions](/guide/library-structure#factions): `{factions: [{id, name, parent_id, depth, book_count}]}`, parents before their sub-factions. Empty for a role-playing system |
| `/api/systems/:id` | PATCH | gm/admin | Update metadata |

**PATCH fields:** `name`, `slug`, `description`, `publishers`, `character_builder_url`, `cover_image`, `cover_book_id`, `tags`, `genre`, `is_explicit`

---

## Books

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/books` | GET | any | Paginated book list. Query: `system_id`, `category`, `limit` (max 500), `offset`, `sort` (`title` or `added_at`), `order` (`asc`/`desc`; `added_at` defaults to newest first), `added_since` (ISO-8601, UTC if no offset; one whose offset carries it past either end of the calendar in UTC is clamped to that end). Each book carries `added_at` - when it first appeared in the library, or `null` for items from the first import of a new install (and older rows with no date). Poll `sort=added_at&added_since=<last poll>` to find new additions. |
| `/api/books/:id` | GET | any | Book detail. For a system the caller can't see, or a visible system inside a container they can't see, `relative_path` leaves out the hidden folders |
| `/api/books/:id` | PATCH | gm/admin | Update book metadata. Changing `category` also moves the file into that category's folder, so a category can't contain `/`, `\`, `..`, a control character (a line break, a tab), a Unicode line or paragraph separator or an invisible format character (a zero-width space, a direction mark), start with a dot, be blank, or run past 100 characters or 255 bytes (`422`). Spaces around it are trimmed first. `language` is one of a fixed list, stored lowercase (`english`): any case, an ISO 639 code (`en`, `pt-BR`) or the language's own name (`Deutsch`) is stored as the language it names, and anything else as `""` (not set) rather than refused; longer than 200 characters is a `422` |
| `/api/books/:id/systems` | GET | any (not guest) | The other systems this book is linked onto, besides the one its folder gives it |
| `/api/books/:id/systems` | POST | gm/admin | Show the book on another system's shelf too. Body: `{system_id}`. 400 for its own system, a version of a book, or across the library/armory divide |
| `/api/books/:id/systems/:system_id` | DELETE | gm/admin | Take the book off that system's shelf. 404 when it was not linked there |
| `/api/books/:id/reindex` | POST | gm/admin | Re-run OCR on a scanned book. Optional query `ocr_dpi` (72–600) re-reads it at a higher resolution than the global `OCR_DPI`. Clears the search index and re-queues the book (OCR runs in the background). 400 for books with an embedded text layer. |
| `/api/books/:id/rescan` | POST | gm/admin | Re-read a single book from disk and rebuild its search index, for a file edited externally. Works for any PDF: a text-layer book is re-extracted; an image-only book is re-queued for OCR. Refreshes page count and thumbnail. Runs in the background; no-ops if a library scan is already running. 400 for non-PDFs. |
| `/api/books/:id/file` | GET | any | Download/stream the file |
| `/api/books/:id/thumbnail` | GET | any | WebP cover thumbnail |
| `/api/books/:id/toc` | GET | any | PDF table of contents |
| `/api/books/:id/page/:num` | GET | any | Render PDF page as WebP. Query: `width` (default 1200, 16-3000; outside that range is 422) |
| `/api/books/:id/page/:num/text` | GET | any | Plain text of a page |
| `/api/books/:id/page/:num/words` | GET | any | Word bounding boxes for text overlay |

---

## Maps

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/maps` | GET | any | Paginated map list. Query: `limit`, `offset`, `map_type`, and the [browse parameters](#browsing-large-collections) |
| `/api/maps/groups` | GET | any (not guest) | Folders holding matching maps, with counts |
| `/api/maps/index` | GET | any (not guest) | Where each letter (or added month) starts in the flat list, for a [jump rail](#browsing-large-collections) |
| `/api/maps/:id` | GET | any | Map detail |
| `/api/maps/:id` | PATCH | gm/admin | Update `description`, `tags`, `map_type`, `grid_size` |
| `/api/maps/:id/file` | GET | any | Download/stream the original map file. A PDF is served inline and may be framed by the app itself, for the viewer's PDF mode; everything else is an attachment |
| `/api/maps/:id/page/:n` | GET | any | Downscaled WebP render. Query: `width` (default 1600, 16-3000; outside that range is 422) |
| `/api/maps/:id/vtt/image` | GET | any | Battlemap image decoded out of a `.uvtt`/`.dd2vtt` file |
| `/api/maps/:id/vtt/data` | GET | any | Grid resolution and wall/portal/light counts from a Universal VTT file |
| `/api/maps/:id/export.uvtt` | GET | any | Builds a Universal VTT file for a raster map: the image, the grid, and anything authored in the editor |
| `/api/maps/:id/otfbm` | GET | may share | How to set the map up as an [OTFBM](/guide/otfbm) background: the whole-cell map size, cell size (`?cell=` 20–100, defaults to a suggestion), and the resized image size |
| `/api/maps/:id/otfbm/image` | GET | may share | The map resized so each cell is `?cell=` pixels, as a JPEG (PNG when transparent), named `<slug>-<W>x<H>c<cell>` |
| `/api/maps/:id/vtt/authoring` | GET | any | Authored walls, portals, lights and environment, with the resolved grid. `data` is null when nothing is authored |
| `/api/maps/:id/vtt/authoring` | PUT | gm/admin | Replaces a map's authored geometry in one atomic write. Body: `{data}` — send `{"data": null}` to clear |
| `/api/maps/:id/thumbnail` | GET | any | WebP thumbnail |
| `/api/map-folders` | GET | any | List folder tag assignments |
| `/api/map-folders` | PATCH | gm/admin | Set tags on a folder. Body: `{path, tags}` |

### Map formats and viewing

Map detail responses carry a `media_kind` field naming the viewer to use:
`image`, `video`, `vtt`, or `archive`.

Raster maps are **not** viewed through `/file`. The viewer requests
`/page/1`, which renders a downscaled WebP — a large battlemap would otherwise
have to be transferred in full before anything appeared. `/file` remains the
download endpoint and serves the untouched original.

Animated battlemaps (`.webm`, `.mp4`) stream from `/file` with a real video MIME
type and no attachment disposition, so they play in place.

Universal VTT files (`.uvtt`, `.dd2vtt`) are JSON envelopes holding the map as
base64 plus wall, portal, and light data
([format reference](https://arkenforge.com/universal-vtt-files/)). The image is
decoded server-side and served by `/vtt/image`; `/vtt/data` returns the grid and
feature counts with the image omitted, so the base64 payload never reaches the
browser.

### Universal VTT authoring

Walls, doors, windows, and lights drawn in the in-app editor are stored against
the map in Grimoire's database, **not** written to the library. Coordinates are
in grid units rather than pixels, which is why recalibrating the grid moves the
geometry with it.

`PUT /vtt/authoring` replaces the whole document atomically — there is no
per-feature endpoint — and the `.uvtt` is assembled on demand by
`/export.uvtt`. The original file on disk is never modified, so authoring works
against a read-only library mount, and editing a map that *is* a `.uvtt` leaves
that file untouched while the export carries its embedded image alongside the
new geometry.

---

## Tokens

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/tokens` | GET | any | Paginated token list. Query: `limit`, `offset`, and the [browse parameters](#browsing-large-collections) |
| `/api/tokens/groups` | GET | any (not guest) | Folders holding matching tokens, with counts |
| `/api/tokens/index` | GET | any (not guest) | Where each letter (or added month) starts in the flat list, for a [jump rail](#browsing-large-collections) |
| `/api/tokens/:id` | GET | any | Token detail |
| `/api/tokens/:id` | PATCH | gm/admin | Update `description`, `tags`, `is_explicit` |
| `/api/tokens/:id/file` | GET | any | Download the token image |
| `/api/tokens/:id/thumbnail` | GET | any | WebP thumbnail |
| `/api/token-folders` | GET | any | List folder tag assignments, plus `frame_folders` — paths (any depth under `tokens/`) holding a `.frames-container` marker |
| `/api/token-folders` | PATCH | gm/admin | Set tags on a folder. Body: `{path, tags}` |
| `/api/token-frames` | GET | any (not guest) | List user-supplied [token editor](/guide/token-editor#custom-frames) frames |
| `/api/token-frames/:id/file` | GET | any (not guest) | Serve one frame image |

Frames are the `.png`/`.webp`/`.svg` files in any folder holding a
`.frames-container` marker file under `tokens/`. The built-in frames ship with
the frontend and are not listed here. A frame `id` is the base64url-encoded
library-relative path, and is revalidated on every request — it must resolve
inside the library, sit in a marked folder reached through non-hidden folders,
and carry an allowed extension. Anything else is a 404.

Each listed frame also carries `token_id` — the id of the `Token` row indexed
from the same file, or `null` when the scanner has not reached it yet. There is
no separate "frame" favourite type: a frame is favourited by starring its token,
and this is the join the editor uses to group favourited frames.

---

## Models

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/models` | GET | any | Paginated 3D model list. Query: `limit`, `offset`, and the [browse parameters](#browsing-large-collections) |
| `/api/models/groups` | GET | any (not guest) | Folders holding matching models, with counts |
| `/api/models/index` | GET | any (not guest) | Where each letter (or added month) starts in the flat list, for a [jump rail](#browsing-large-collections) |
| `/api/models/:id` | GET | any | Model detail, including viewer capability |
| `/api/models/:id` | PATCH | gm/admin | Update `description`, `tags`, `is_explicit`, `is_supported` |
| `/api/models/:id/file` | GET | any | Download the model file |
| `/api/models/:id/thumbnail` | GET | any | WebP thumbnail (rendered preview) |
| `/api/models/bulk` | POST | gm/admin | Bulk update models |
| `/api/models/bulk/tags` | POST | gm/admin | Bulk add tags to models |
| `/api/model-folders` | GET | any | List folder tag assignments |
| `/api/model-folders` | PATCH | gm/admin | Set tags on a folder. Body: `{path, tags}` |
| `/api/model-folders/bulk` | POST | gm/admin | Set tags on many folders |

The detail response carries three viewer fields. `viewer_loader` names the client-side loader for the format (`stl`, `gltf`, `3mf`, `ply`) and is empty when none applies. `viewer_available` says whether the browser viewer should load the file on its own. `viewer_oversized` distinguishes the two reasons it might not: when true, the format is supported and only the size cap held it back, so the client can offer to load it anyway behind a warning; when false alongside a false `viewer_available`, no loader exists for the format and a download is the only option.

`is_supported` is a tri-state — `true` presupported, `false` unsupported, `null` when the name says nothing. See the [3D Models](/guide/models) guide.

---

## Browsing large collections

The gallery list routes (`/api/maps`, `/api/tokens`, `/api/audio`, `/api/models`) and their `/groups` and `/index` companions filter, sort and group on the server, so a client can browse a library of any size a page at a time. They share these parameters:

| Parameter | Meaning |
|---|---|
| `q` | Search text over the name, folder path and tags; accepts `field:` prefixes (`tag:`, `title:`, `artist:`, `album:`) |
| `tags` | Tag filter as JSON: `[{"mode": "include"\|"exclude", "tags": [...]}]` groups (OR within a group, AND between groups), or a flat list. Folder tags count for everything beneath the folder. `__grim:none__` / `__grim:any__` match untagged / tagged items |
| `favorites` | `true` for the caller's favourites only |
| `added_since` | Items added at or after this ISO time; undated items never match. A time whose offset puts it outside the calendar in UTC is clamped to that end |
| `folder` | Only items directly in this folder (`""` for the collection root) |
| `sort`, `order` | `path` (default), `name` (natural order), `size`, `added_at`; audio also `title`, `duration`, `track` (folder, then disc and track number, then name). `asc` or `desc` |

`/groups` returns `{total, groups: [{path, count}]}` - every folder holding matching items.

`/index` (`/api/maps/index`, `/api/tokens/index`, `/api/audio/index`, `/api/models/index`) feeds the gallery's letter rail. It takes the same filters plus `sort` and `order`, and returns `{entries: [{key, label, count, offset}]}`: where each run of the flat list starts. For the `name` sort a run is a first letter (case- and accent-folded, so "Élan" is an `E`; digits and symbols under `#`); for `added_at` it is a month (`label` is `YYYY-MM`, and `""` for undated items). `offset` is where the run starts in the list route's results for the same filters and order, so a client can jump there with `offset=`. Any other sort returns no entries. Books and game systems have no `/index` route.

A tag's items are paged the same way: `/api/tags/:internal/items` returns per-type counts, the tagged folders with counts, and a page of directly-tagged items (`resource_type`, `limit`, `offset`), and `/api/tags/:internal/folder-items` pages through one tagged folder.

---

## Search

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/search?q=` | GET | any | Full-text search (SQLite FTS5, or PostgreSQL full-text search). `q` is 2-500 characters; only its first 64 terms, operators included, reach the full-text index, and a query the database still can't run is a `400`. Optional: `book_id`, `system_id`, `limit` (1-200, default 50) |

**Response:**
```json
{
  "query": "fireball",
  "total": 42,
  "results": [{"id": "uuid", "title": "...", "game_system": "...", "page_number": 42, "snippet": "..."}],
  "maps":    [{"id": "uuid", "filename": "...", "relative_path": "...", "tags": []}],
  "tokens":  [{"id": "uuid", "filename": "...", "relative_path": "...", "tags": []}]
}
```

`maps` and `tokens` are empty when `book_id` or `system_id` is scoped.

---

## Favorites

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/favorites` | GET | any | List current user's favorites |
| `/api/favorites` | POST | any | Add a favorite (idempotent). Body: `{item_type, item_id}` |
| `/api/favorites/:type/:id` | DELETE | any | Remove a favorite |

Item types: `book`, `map`, `token`, `system`

---

## Bookmarks

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/bookmarks?book_id=` | GET | any | List user's bookmarks for a book |
| `/api/bookmarks` | POST | any | Create a bookmark. Body: `{book_id, page_number, label?, notes?, selected_text?}` |
| `/api/bookmarks/:id` | PATCH | any | Update `label` or `notes` |
| `/api/bookmarks/:id` | DELETE | any | Delete a bookmark |

---

## Pins

Folders and game systems a user keeps one click away, shown as a strip at the top of their view. Per user; guests have none.

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/pins` | GET | any (not guest) | The user's pins, oldest first: `{pins: [{id, collection, target}]}`. Optional `?collection=` limits to one view (`400` for an unknown one) |
| `/api/pins` | POST | any (not guest) | Pin. Body: `{collection, target}` (`target` at most 1000 bytes). Pinning something already pinned returns the existing pin. At most 100 pins per view |
| `/api/pins/:id` | DELETE | any (not guest) | Unpin. Someone else's pin is a `404`, the same as a missing one |

`collection` is the view: `systems` (`target` is a game system's id) or `maps`, `tokens`, `audio`, `models` (`target` is a collection-relative folder path). A pin points at the path rather than the items in it, so it survives a rescan.

---

## Saved audio sets

Named, per-user playlists and soundboards. See [Audio Library](/guide/audio#saved-playlists-and-soundboards).

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/audio-sets` | GET | any | List the user's saved sets (names and counts only). Optional `?kind=` limits to one kind |
| `/api/audio-sets` | POST | any | Save a set. Body: `{kind, name, entries: [{audio_id, loop?}], layout?: {cols, rows}}` |
| `/api/audio-sets/:id` | GET | any | Load one set, resolved against the current library |
| `/api/audio-sets/:id` | PATCH | any | Rename or replace contents. Body: `{name?, entries?, layout?}` |
| `/api/audio-sets/:id` | DELETE | any | Delete a saved set |

Kinds: `playlist`, `soundboard`. `layout` is the soundboard grid and is `null` for a
playlist. Re-saving an existing `(kind, name)` overwrites that set rather than creating a
second one.

Entries store audio ids only, so titles come back resolved against the library as it is
now. An entry whose track has been removed is omitted and counted in `missing`, letting a
stale set load with what remains instead of failing.

---

## Campaigns

### Campaign CRUD

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns` | GET | any | List own + invited campaigns |
| `/api/campaigns` | POST | any | Create campaign |
| `/api/campaigns/:id` | GET | owner or member | Campaign detail |
| `/api/campaigns/:id` | PATCH | owner or admin | Update campaign |
| `/api/campaigns/:id` | DELETE | owner or admin | Delete campaign, including its uploads on disk (banner, campaign files, members' art, tokens and sheets) |

### Members

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/invite` | POST | owner or admin | Invite a user. Body: `{user_id}` |
| `/api/campaigns/:id/members/:user_id` | PATCH | member or owner | Accept/decline or set character name |
| `/api/campaigns/:id/members/:user_id` | DELETE | owner, admin, or self | Remove member, and the art, token and sheet they uploaded to it. |

### Sessions

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/sessions` | GET | member or owner | List sessions |
| `/api/campaigns/:id/sessions` | POST | member or owner | Create session. Body: `{session_date, title?}` |
| `/api/campaigns/:id/sessions/search` | GET | member or owner | Search session notes. Query: `q` (1-200 characters). Returns `{results: [{session_id, session_date, session_title, note_type, author_id, author_username, author_display_name, snippet}], query}` by session date. GM internal notes are searched only for the owner |
| `/api/campaigns/:id/sessions/:sid` | PATCH | owner or admin | Update title. Members, guests included, get `403` |
| `/api/campaigns/:id/sessions/:sid` | DELETE | owner or admin | Delete session |
| `/api/campaigns/:id/sessions/:sid/notes/player` | PUT | member or owner | Save player note. Body: `{content}` |
| `/api/campaigns/:id/sessions/:sid/notes/gm` | PUT | owner or admin | Save GM notes. Body: `{internal_content?, external_content?}` |

### Schedule

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/schedule` | GET | member or owner | Schedule + next 10 dates |
| `/api/campaigns/:id/schedule` | PUT | owner or admin | Create or update schedule |
| `/api/campaigns/:id/schedule` | DELETE | owner or admin | Remove schedule |

**Schedule body:**
```json
{
  "frequency": "weekly",
  "days": [5],
  "time_utc": "18:00",
  "biweekly_reference": "2026-01-03",
  "monthly_week": null,
  "custom_dates": null
}
```

`frequency` values: `weekly`, `biweekly`, `monthly`, `custom`. Days are 0 (Monday) – 6 (Sunday); duplicates are dropped, and a monthly schedule uses the first day. A time is `HH:MM` (`00:00`–`23:59`), `timezone` must name an IANA zone the server knows, `monthly_week` is `1`–`4` or `-1` for last, and `custom_dates` are at most 600 dates within ten years of today. A value outside these rules is `400`, naming the field. A schedule stored before these checks still reads: dates it cannot use are left out, and a bad time publishes all-day events.

### Availability

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/availability` | GET | member or owner | Availability for next 10 scheduled sessions |
| `/api/campaigns/:id/availability/:date` | PUT | member or owner | Set own availability. Body: `{status, is_cancelled?, user_id?}`. `user_id` and `is_cancelled` are GM-only: `user_id` sets that person's availability instead, and must name the owner or an accepted member (`404` otherwise) |
| `/api/campaigns/:id/availability/:date/cancel` | PUT | owner | Toggle session cancellation. Anyone else, an admin included, gets `403` |

Statuses: `available`, `tentative`, `unavailable`

### Wiki map pages

A wiki page's title is written into the `[[links]]` that point at it, so it can't contain `|`, `[`, `]` or a line break: creating or renaming a page to one is a `400`. A title already stored with one is kept on a save that doesn't rename the page, and a title that arrives some other way (a link's auto-created stub, a pin's new target, an import, a template's default) has each run of those characters turned into a space. A link's target and label hold no `[[`: a second opener starts a new link, so `[[draft [[Castle]]` links to `Castle`. A page body, and a note template's, holds at most 1,000,000 characters (`422` otherwise, and an import naming the page).

A page with `page_type: "map"` is built around one map: `map_id` (an image map linked to the campaign as a resource the caller can see) or `map_file_id` (one of the campaign's own image files). Page detail returns `map` (`{source, id, name, has_thumbnail}`, `source` one of `library`, `file`, `missing`, `none`, or `hidden` for a picture this caller may not see, sent with no `id` or `name`) and the `pins` the caller may see. A library map is shown to the campaign owner and to whoever its campaign link is visible to, so a map linked as `gm` stays the GM's on a public map page. A pin is `{id, x, y, label, target_page_id, target_title, icon, icon_color, visibility, sort_order, can_edit, ...}`, with `x`/`y` as fractions (0-1) of the picture, and is returned only to someone who can see both the map page and its target. A pin with `visibility: "gm"` is the campaign GM's alone.

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/wiki/:page_id/pins` | POST | anyone who can edit the page | Add a pin to a map page (`400` on any other type). Body: `{x, y, label?, target_page_id?, target_title?, icon?, icon_color?, visibility?}`. A `target_title` naming no page the caller can see creates a stub, as a `[[link]]` does. Only the GM can set `visibility: "gm"` (`403`) |
| `/api/campaigns/:id/wiki/:page_id/pins/:pin_id` | PATCH | anyone who can edit the page and see the pin | Move or edit a pin; every field optional. `target_page_id: ""` unlinks it; `target_title` relinks by title. `404` for a pin the caller can't see |
| `/api/campaigns/:id/wiki/:page_id/pins/:pin_id` | DELETE | anyone who can edit the page and see the pin | Remove a pin |

When a map is deleted from the library through the app, each campaign using it gets its own copy as a campaign file where the upload settings allow, and its map pages, `[[map:ID]]` embeds and map resource are pointed at the copy. The copy is read by the people who could see the map. Otherwise its map pages read as `missing`, with their pins intact. [`GET /api/files/campaign-map-copies`](#file-manager-admin-only) previews this.

### GM screen

A campaign's [GM screen](/guide/gm-screen) is a board of built-in tools on a twelve-column snap grid, for the campaign's owner alone: every route here answers `403` to anyone else, members and admins included. Each panel has a `kind` (`note`, `party`, `counter`, `clocks`, `pools`, `scratch`, `initiative`, `reference`, `table`), a `title`, a cell (`x`, `y`, `w`, `h` in grid units), a `config` and a `state`. An unknown `kind` is a `400`, and an unreadable `config` or `state` value falls back to its default. A board holds at most 60 panels.

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/gm-screen` | GET | owner | `{panels}`, top to bottom |
| `/api/campaigns/:id/gm-screen/panels` | POST | owner | Add a panel. Body: `{kind, title?, x?, y?, w?, h?, config?, state?}`. Without `x`/`y` it takes the first gap it fits |
| `/api/campaigns/:id/gm-screen/panels/:panel_id` | PATCH | owner | Change any of `title`, `x`, `y`, `w`, `h`, `config`, `state`. `state` replaces the panel's state whole |
| `/api/campaigns/:id/gm-screen/panels/:panel_id` | DELETE | owner | Remove a panel. Returns `{panels}` |
| `/api/campaigns/:id/gm-screen/layout` | PUT | owner | Move and resize several panels at once. Body: `{panels: [{id, x, y, w, h}]}` |
| `/api/campaigns/:id/gm-screen/reset` | POST | owner | Body: `{scope: "scene" \| "session" \| "rest"}`. Clears what each panel marks to clear at that point - counters back to their start, pool tokens and countdown ticks to 0, the initiative order emptied |
| `/api/campaigns/:id/gm-screen/sources` | GET | owner | `{campaigns: [{id, name, panel_count}]}`: the caller's other campaigns with a board to copy |
| `/api/campaigns/:id/gm-screen/copy` | POST | owner | Body: `{source_campaign_id, replace?}` (default `true`; `false` adds the board below this one). Copies the layout and each panel's config; every state starts at its default |
| `/api/campaigns/:id/gm-screen/export` | GET | owner | The board as a document: `{format: "grimoire://gm-screen/v1", name, panels}` |
| `/api/campaigns/:id/gm-screen/import` | POST | owner | Body: `{document, replace?}`. Puts a board document on this campaign. `400` for one with no panels or a panel of unknown kind |
| `/api/campaigns/:id/gm-screen/presets` | GET | owner | The community preset catalogue, read from `gm-screens/index.json` beside the configured add-on sources. `403` when community installs are disabled |
| `/api/campaigns/:id/gm-screen/presets/apply` | POST | owner | Body: `{preset_id, index_url?, replace?}`. Downloads the preset, checks it against the catalogue's digest and puts it on the board. `404` for a preset the catalogue doesn't list, `502` for one that fails to download or verify |
| `/api/campaigns/:id/gm-screen/party` | GET | owner | The campaign's characters for the party panel, with each one's scalar fields and computed values (worked out against the GM's copy of the sheet; for a sheet the GM hasn't installed, only its stored fields), and `editable` for those the GM may change (through `PUT /api/characters/:id`) |

Writes follow the campaign's lock like every other campaign write.

---

## Characters

Sheets (schemas) and characters are stored **per user**, so any signed-in user — guests included — may install a sheet. See [Characters](/guide/characters) for how the builder works.

### Sheets

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/characters/schemas` | GET | any | Installed sheets, each with a `character_count` |
| `/api/characters/schemas` | POST | any | Install a pasted sheet. Body: `{text}` (JSON or YAML, without YAML aliases, at most 512 KB and 100 levels deep) or `{document}`, plus optional `layout` and `styles`. 400 names the problem if it does not validate |
| `/api/characters/schemas/browse` | GET | any | The community sheet catalogue, each marked `installed`. `errors` lists sources that could not be read; for anyone but an admin their details, `index_url` and `sources` are left empty, since the sources are server configuration |
| `/api/characters/schemas/install/:sheet_id` | POST | any | Install a sheet from the catalogue |
| `/api/characters/schemas/:schema_id` | GET | any | One sheet with its validated `document` |
| `/api/characters/schemas/:schema_id` | DELETE | any | Uninstall a sheet. Characters built on it are kept |

### Characters

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/characters` | GET | any | Your characters plus your parties', newest first. `?schema_ref=` and `?campaign_id=` filter |
| `/api/characters` | POST | any | Create one. Body: `{schema_ref, name?, data?, campaign_id?}` |
| `/api/characters/:id` | GET | owner or party | One character with `data` and freshly worked-out `computed` |
| `/api/characters/:id` | PUT | owner | Update `name`, `data` (a partial patch), `unset`, `campaign_id` (`""` clears it) and/or `status` |
| `/api/characters/:id` | DELETE | owner | Delete a character |
| `/api/characters/:id/export` | GET | owner or party | A self-contained file, with the sheet and every referenced entry embedded |
| `/api/characters/import` | POST | any | Rebuild a character from an exported file. Body: `{payload, import_entries?}` |
| `/api/characters/importers` | GET | any | `{importers, available, downloads_enabled}`: the importers you can run (`scope` server or user, the `sheet` each builds on, `extensions`, whether it reads files and links, its `inputs`), and catalogue importers you lack with `install_as` - `user`, `server`, `settings` or `admin` |
| `/api/characters/importers/:id/install` | POST | any | Install a file-only YAML importer for yourself, or update your copy |
| `/api/characters/importers/:id` | DELETE | any | Remove an importer you installed |
| `/api/characters/import/external` | POST | any | Build a character from another tool's export. Body: `{text, filename}` for a file or `{url}` for a link, plus optional `importer_id`, `campaign_id` and `inputs` (`{key: value}` answers to the importer's declared inputs, never stored). The importer is recognised automatically unless named; 409 when several claim it. A missing input answers 400 with `detail: {message, importer_id, needs_input}`. Installs the importer's sheet from the catalogue if you lack it. Returns the character plus `importer_id`, `importer_name` and `unmatched` (names kept as custom entries) |
| `/api/characters/:id/portrait` | GET / POST / DELETE | read: owner or party; write: owner | The character's art (PNG, JPEG, WebP or GIF, 5 MB) |
| `/api/characters/:id/images/:field` | GET / POST / DELETE | read: owner or party; write: owner | The picture a sheet's `image` field holds (same types and size). POST returns `{field, value}`; `value` is the stored file name, also the URL's `?v=` cache-buster |

**Character fields:** `id`, `name`, `schema_ref`, `schema_name`, `system`, `schema_missing`, `campaign_id`, `campaign_name`, `status`, `portrait_path`, `portrait_version`, `portrait_crop` (`{x, y, zoom}` - the focus point in percent and a 1-4 zoom; `null` is centred, and `PUT` with `{}` clears it), `owned`, `created_at`, `updated_at`, and on detail `data`, `computed`, `validators`, `entries`.

**Campaigns.** A character can be placed in a campaign you own (a GM campaign or a personal one) or have joined — 403 otherwise, including for an invitation you have not accepted. Everyone at that table — the owner and accepted members — can then read the sheet; only its owner can change it (`owned` is `false` for anyone else). A player may have any number of characters in one campaign.

**Status** is `active` (the default), `retired` or `dead`, so a fallen character can stay beside its replacement. Any other value is a 400.

### Content catalog

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/content/packs` | GET | any | Installed content packs with their licence and credit |
| `/api/content/packs/reload` | POST | admin | Re-read every pack from disk |
| `/api/content/:schema_id/types` | GET | any | The content types a sheet declares, with entry counts |
| `/api/content/:schema_id/:content_type` | GET | any | Browse: `search`, `filter[field]=value`, `sort`, `page`, `page_size`. Each facet in `filters_available` is narrowed by every other active filter but never by itself, so a choice stays reversible; a facet with more than 500 values is left out |
| `/api/content/:schema_id/:content_type/:entry_id` | GET | any | One entry in full |

### Rulesets

A campaign ruleset is read by everyone at that table and edited by the campaign's owner; a server ruleset (`campaign_id` null) is read by everyone and edited by admins; a personal ruleset (`personal: true`) is read and edited by its owner alone. Importing a character creates a personal ruleset for the content embedded in its file.

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/rulesets` | GET | any | Rulesets you can read. `schema_id` and `campaign_id` filter |
| `/api/rulesets` | POST | campaign owner, or admin for a server ruleset | Create one |
| `/api/rulesets/packs/browse` | GET | any | The community content-pack catalogue, with `installed` and `update_available`. Source details are admin-only, as for the sheet catalogue |
| `/api/rulesets/packs/install/:pack_id` | POST | admin | Install or update a content pack |
| `/api/rulesets/packs/:pack_id` | DELETE | admin | Uninstall a content pack |
| `/api/rulesets/:id` | GET / PUT / DELETE | read: reader; write: editor | One ruleset |
| `/api/rulesets/:id/export` | GET | reader | The ruleset as a portable document |
| `/api/rulesets/:id/import` | POST | editor | Import a `pack_id`, a `document`, or `text` (JSON or YAML). `conflict`: `skip` (default), `overwrite`, `rename` |
| `/api/rulesets/:id/fork` | POST | editor | Copy an entry into this ruleset to change it |
| `/api/rulesets/:id/entries` | GET / POST | read: reader; write: editor | List or add entries |
| `/api/rulesets/:id/entries/:entry_id` | GET / PUT / DELETE | read: reader; write: editor | One entry |

---

## Armies & leagues

The [army builder and leagues](/guide/armies) for wargames. Not available to a campaign's guests (`403`); a league's [guests](/guide/armies#league-guests) reach their own armies and their leagues, with the library narrowed to their leagues' games and the books shared with them. An API key needs the **Armies and leagues** permission. Grimoire ships no points, stat lines or rules text - an army holds only what its owner imported or typed.

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/armies` | GET | any (league guests too) | Your own armies, most recently changed first |
| `/api/armies` | POST | any (league guests too) | Build an army by hand: `{name, game_system_id?, faction_id?, points_limit?, notes?, units}` |
| `/api/armies/import` | POST | any (league guests too) | Import a BattleScribe / New Recruit roster: multipart `file` (`.ros` or `.rosz`, up to 20 MB), optional `name`. Matched to a wargame system and faction by name; the file is kept as uploaded |
| `/api/armies/games` | GET | any (league guests too) | The wargames an army can be set to: `[{id, name, parent_id}]`, `parent_id` null when that container is hidden from the caller. A league's guest gets only their leagues' games and those games' editions |
| `/api/armies/games/:system_id/factions` | GET | any (league guests too) | `{factions: [{id, name, parent_id, depth}]}`, parents before their sub-factions. `404` for a game the caller may not choose |
| `/api/armies/:id` | GET / PATCH / DELETE | read: owner, admin, the league's people; write: owner, admin | One army. An imported list's units are read-only |
| `/api/armies/:id/export` | GET | as GET | `?format=text` (default) the list as text; `?format=roster` the uploaded file |
| `/api/armies/:id/units/:unit_id/progress` | PUT | owner, admin | A unit's league progress: `{experience, honours, notes}` |
| `/api/armies/:id/publications` | GET | as GET | Each publication the roster cites, its link, and matching wargame books |
| `/api/armies/publication-links` | PUT / DELETE | gm/admin | Link a cited publication to a library book (with a `page_offset`) for every army that cites it |
| `/api/armies/publication-links/books` | GET | gm/admin | `?q=` (2+ characters): up to 10 wargame books the caller can see whose titles contain it, `[{book_id, title}]` |
| `/api/leagues` | GET | any (league guests too) | Leagues you organise or play in (admins: every league) |
| `/api/leagues` | POST | gm/admin | Start a league: `{name, description?, game_system_id?, points_limit?, points_win?, points_draw?, points_loss?}` |
| `/api/leagues/:id` | GET / PATCH / DELETE | read: the league's people; write: organiser, admin | The league with its members, games and standings |
| `/api/leagues/:id/eligible-members` | GET | organiser, admin | Users who could be added: anyone not a guest and not already in |
| `/api/leagues/:id/members` | POST | organiser, admin | Add a member: `{user_id}`. A guest or unknown user is `404` |
| `/api/leagues/:id/members/:user_id` | DELETE | organiser, admin, or that member | Remove a member, or leave. Their logged games stay; the league's books shared with them go |
| `/api/leagues/:id/members/:user_id/army` | PUT | that member, organiser, admin | Register the army the member plays: `{army_id}` (null clears it). It must be the member's own |
| `/api/leagues/:id/games` | POST | members, organiser | Log a game: `{player_a_id, player_b_id, score_a?, score_b?, result?, mission?, played_on?, notes?}` |
| `/api/leagues/:id/games/:game_id` | PUT / DELETE | its players, whoever logged it, organiser | Edit or delete a game |
| `/api/leagues/:id/books` | GET | the league's people | Books shared with the league that the caller may open. The organiser sees every one, with `shared_user_ids`; a member the `public` ones and the `private` ones shared with them |
| `/api/leagues/:id/books` | POST | organiser, admin | Share a book: `{book_id, visibility?, shared_user_ids?}`. `visibility` is `public` (every member), `private` (the members in `shared_user_ids`) or `organizer` (the default). A book restricted by its access level is kept at `organizer`. `409` for one already shared |
| `/api/leagues/:id/books/search` | GET | organiser, admin | `?q=` (2+ characters): up to 20 library books the caller can open whose titles contain it, the league's own game first; `game_system_name` is blank for a system the caller cannot see |
| `/api/leagues/:id/books/order` | PUT | organiser, admin | Reorder: `{ordered_ids}` |
| `/api/leagues/:id/books/:resource_id` | PATCH / DELETE | organiser, admin | Change a shared book's `visibility` or `shared_user_ids`, or stop sharing it |
| `/api/leagues/:id/guests` | GET | organiser, admin | The league's guests: `[{user_id, nickname, guest_code, army}]` |
| `/api/leagues/:id/guests` | POST | organiser, admin | Invite a guest: `{nickname?}`. Makes a code-only account and adds it to the league. `403` while guest access is off |
| `/api/leagues/:id/guests/:user_id/regenerate` | POST | organiser, admin | A new code; the old one stops working and the guest's sessions end |
| `/api/leagues/:id/guests/:user_id/share` | GET | organiser, admin | Ready-made invite text: `{code, link, message, mailto_url, discord_message}` |
| `/api/leagues/:id/guests/:user_id` | DELETE | organiser, admin | Take the guest out of the league. Their account and armies stay, and their code still signs in |

The guest invite-code routes are session-only: an API key can't create, list, regenerate or share them. A guest also in a league someone else organises shows a null `guest_code`, and regenerating or sharing their code is `403` - admins excepted.

---

## Settings *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/settings` | GET | Get all application settings |
| `/api/settings` | PATCH | Update application settings |
| `/api/settings/ui` | GET | UI visibility flags (any authenticated user) |
| `/api/settings/folder-access` | GET | Every folder restriction: `{rules: [{path, level, exists, books}]}`, by path. `exists` is false once the folder is gone from disk; `books` is how many books the rule covers now |
| `/api/settings/folder-access` | PUT | Restrict every book beneath a folder. Body: `{path, level: "gm" \| "admin"}`, where `path` is a library-relative folder inside `books/` or the armory (`400` otherwise, `404` when it doesn't exist). Replaces any level the folder had, and demotes campaign shares of the books it restricts to GM-only. Returns the rule |
| `/api/settings/folder-access?path=` | DELETE | Lift a folder's restriction. `404` when it has none. Shares it demoted stay GM-only |

A folder restriction only ever adds one: a book with no access level of its own resolves to the stricter of what its system or category gives it and the strictest folder rule above it. Only the book's own level overrides a folder rule. Rules follow a folder renamed or moved in the file manager, and are deleted with it.

---

## API keys

Manage your [API keys](#api-keys), and answer MCP OAuth sign-ins. These need a login: a key can never manage keys.

| Endpoint | Method | Description |
|---|---|---|
| `/api/api-keys` | GET | List your keys (never the key itself). Admins can add `?all=true` for everyone's. Each has a `kind`: `key`, or `app` for an MCP client you approved through OAuth, with its `client_name` and `redirect_host` |
| `/api/api-keys/permissions` | GET | The permissions your role can grant, with descriptions and the levels each offers |
| `/api/api-keys` | POST | Create a key. Body: `{name, permissions?, expires_at?}`, where `permissions` is `{permission: "none" \| "read" \| "write"}` and `"*"` means all permissions. Returns `{api_key, key}`: `key` is the full key, returned only here and by regenerate |
| `/api/api-keys/:id` | PATCH | Rename your key or change its permissions or expiry (`expires_at: null` means never) |
| `/api/api-keys/:id/regenerate` | POST | Issue a new key with the same name and permissions. The old key stops working immediately. `409` for an app: revoke it instead |
| `/api/api-keys/:id` | DELETE | Revoke your key or app. Admins can revoke anyone's. A revoked app can be approved again; to stop that, turn API keys off for the user |
| `/api/oauth/requests/:request_id` | GET | MCP sign-in (`MCP_OAUTH_ENABLED`, else `404`): what a pending request asks for. `404` once it has expired (10 minutes) or been answered |
| `/api/oauth/requests/:request_id/approve` | POST | Approve it as yourself. Body: `{permissions?}` as for a key; omitted is Read on everything your role can read. Returns `{redirect_to}` |
| `/api/oauth/requests/:request_id/deny` | POST | Deny it. Returns `{redirect_to}` with `error=access_denied` |

---

## Share links

Public, revocable links to one book, map, token, audio track or 3D model - see [Share links](/guide/sharing). Managing them needs a login or an API key with the **Share links** permission; guests never can.

| Endpoint | Method | Description |
|---|---|---|
| `/api/share-links` | GET | Your links, newest first, each with its `state` (`active`, `disabled`, `revoked`, `expired`, `suspended`, `unavailable`). Filter with `resource_type` and `resource_id` |
| `/api/share-links` | POST | Create a link. Body: `{resource_type, resource_id, label?, expires_in_days?, password?, variant?, cell?}`, where `resource_type` is `book`, `map`, `token`, `audio` or `model` and `expires_in_days: null` means never. `variant: "otfbm"` (maps only) serves the map resized as an [OTFBM](/guide/otfbm) background at `cell` px per square. Returns the link with `url` (the share page) and `file_url` (the raw file) |
| `/api/share-links/permission` | GET | Whether you may create links, and whether that comes from your role or a per-user setting |
| `/api/share-links/:id` | PATCH | Change the label, expiry or password (`password: null` removes it) |

A new link password must be at least `SHARE_LINK_PASSWORD_MIN_LENGTH` characters (8 by default) and not a commonly used password.
| `/api/share-links/:id/revoke` | POST | Stop a link working for good. It stays listed as revoked |
| `/api/share-links/:id` | DELETE | Remove a link |
| `/api/share-links/all` | GET | Every user's links *(admin only)* |
| `/api/share-links/users/:user_id` | GET | One user's links *(admin only)* |

The public side needs no authentication at all - the token is the credential:

| Endpoint | Method | Description |
|---|---|---|
| `/api/share/:token/meta` | GET | What the share page shows. For a password-protected link, only `password_required` and `unlocked` until it is unlocked |
| `/api/share/:token/unlock` | POST | Body `{password}`. Unlocks the link in this browser for 12 hours. Wrong passwords are limited per IP and per link (`ACCOUNT_RATE_LIMIT`), and get `429` past either |
| `/api/share/:token/:filename` | GET, HEAD | The item itself (`?download=1` for an attachment). The filename is ignored, and is there for tools that read a file's type from the URL |

Every way a link can fail to work - unknown, expired, revoked, suspended, or the item gone - is the same `404`. Responses allow cross-origin reads (`Access-Control-Allow-Origin: *`), so VTTs such as Owlbear Rodeo can load them.

---

## File manager *(admin only)*

Structural file management for the library. Every path is relative to the library root; a path resolving outside it is a `403`. These routes write to the library, so they need it mounted read-write (`409` otherwise). Moves and renames keep each item's record, so its id, tags, favorites, bookmarks and campaign links survive.

| Endpoint | Method | Description |
|---|---|---|
| `/api/files/browse` | GET | List one library folder, with each file's indexing state |
| `/api/files/move` | POST | Move files and folders into a destination folder |
| `/api/files/rename` | POST | Rename a file or folder on disk |
| `/api/files/folder` | POST | Create a folder |
| `/api/files/folder/markers` | PUT | Set or clear a folder's container-kind, NSFW and frame markers |
| `/api/files/folder` | DELETE | Delete a folder, recursively when confirmed by name |
| `/api/files/folder/contents` | GET | Whether a folder holds content |
| `/api/files/campaign-map-copies` | GET | Campaigns using a map at or under `?path=`, and whether deleting it leaves each one a copy: `{campaigns: [{campaign_id, campaign_name, map_id, map_name, outcome}]}`, `outcome` one of `copy`, `too_large`, `uploads_disabled`, `not_image`, `file_missing` (see [Wiki map pages](#wiki-map-pages)) |
| `/api/files/delete` | POST | Remove a file or folder from the index, or also from disk |
| `/api/files/folder/scaffold` | POST | Create the standard category folders in a system folder |
| `/api/files/upload` | POST | Upload one file into a library folder |

**Uploads.** `POST /api/files/upload` takes a multipart `file`, `destination`, an optional `relative_dir`, and `on_conflict`: `rename` (the default), `skip` or `replace`. It returns `{path, name, size, record_id, record_type, replaced}`.

`on_conflict=replace` overwrites the file of an indexed book, map, token, audio file or 3D model and keeps its record, so the same id keeps its tags, metadata, favorites, bookmarks and campaign links. The upload must land on the exact path of an indexed file under `books/`, the armory, `maps/`, `tokens/`, `audio/` or `models/`; anything else is a `409`, so a mistyped name never adds a second copy. It is also a `409` while a library scan or OCR run is going. `record_type` then says what was replaced (`book`, `map`, `token`, `audio` or `model`). The file's size, hash and thumbnail are re-read (and a model's triangle count, a track's duration); what you wrote - description, tags, a map's grid and authored walls and lights - is kept.

---

## Maintenance *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/maintenance/cleanup-missing` | POST | Remove DB records for files no longer present on disk |

---

## Backups *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/backups` | GET | List backups, newest first, with `created_at`, `size_bytes`, and `version`. `includes_database` is `false` on PostgreSQL, where an archive holds the files but not the database |
| `/api/backups` | POST | Create a backup now |
| `/api/backups/settings` | GET | Read backup schedule, retention, and storage location |
| `/api/backups/settings` | PUT | Configure schedule, retention, and storage location |
| `/api/backups/{backup_id}/download` | GET | Download a backup archive (`application/zip`) |
| `/api/backups/{backup_id}` | DELETE | Delete a backup archive (`204`) |

A backup is a timestamped `.zip` holding a consistent snapshot of the database plus the
files uploaded through Grimoire. The **library is never included**. Because the listing
carries timestamps, a client can check how stale the newest backup is and take a fresh one
before running something destructive.

Creating a backup pauses database writes for the length of the snapshot; a second
concurrent create returns `409`.

**There is no restore endpoint, by design.** See [Backups](/configuration/backups) for the
restore procedure and configuration.

---

## Logs *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/logs` | GET | Retrieve recent log entries from the in-memory ring buffer |

**Query parameters:** `level` (default `info`), `limit` (default 200, max 20000), `offset`, `after_seq` (cursor for live polling).

---

## Error responses

```json
{"detail": "Human-readable error message"}
```

| Status | Meaning |
|---|---|
| `400` | Bad request / business rule violated |
| `401` | Not authenticated |
| `403` | Forbidden, insufficient role |
| `404` | Not found |
| `409` | Conflict, duplicate resource |
| `422` | Request body failed schema validation |
