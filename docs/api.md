# API Reference

**Version:** 0.1.0

## Interactive docs

With the server running, the API is self-documented via OpenAPI:

| URL | Description |
|---|---|
| `http://localhost:9481/api/docs` | Swagger UI: interactive, try-it-out docs |
| `http://localhost:9481/api/redoc` | ReDoc: clean, readable reference |
| `http://localhost:9481/api/openapi.json` | Raw OpenAPI schema |

---

## Authentication

All endpoints except `/api/auth/status`, `/api/auth/setup`, `/api/auth/login`, and `/api/auth/config` require a JWT, or, for external integrations, an [API key](#api-keys).

**Header** (preferred):
```
Authorization: Bearer <token>
```

**Query parameter** (required for browser-embedded images and file downloads):
```
?token=<token>
```

Tokens are returned by `/api/auth/login` and expire after **30 days**.

### Roles

| Role | Permissions |
|---|---|
| `admin` | Full access including user management and app settings |
| `gm` | Read + edit metadata, rescan library, manage campaigns |
| `player` | Read-only access |

### API keys

Scripts and tools, such as a [Homepage widget](/guide/homepage), [grimoire-cli](https://github.com/thomaslazar/grimoire-cli), or a backup script, authenticate with an **API key** instead of a login:

```
X-API-Key: grim_...
```

**Keys are personal.** Every key belongs to a user and acts **as that user**: their role, their campaigns, their favorites. The key's permissions can only narrow that; a key can never do more than its owner. If the owner's role changes, their keys follow. Keys are deleted with their owner.

Create keys under **Settings → Account → Security → API Keys**. Admins always can. Anyone else can once an admin ticks **API keys** on their row in **Settings → Users** (off by default), or your [OIDC provider](/guide/oidc) grants the `apiKeys` permission; taking it away stops their existing keys working. Admins can see and revoke everyone's keys under **Settings → App Settings**. Guests never have keys.

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
| Campaigns | Your campaigns: members, sessions, wiki, resources and schedule |
| Downloads | Zip archive downloads |
| File manager | Browse, upload, move, rename and delete library files |
| Duplicates | Find, compare, link and merge duplicates |
| Add-ons | Community metadata add-ons |
| Maintenance | Metadata sidecar settings and export |
| Backups | Backups and the backup schedule |
| Logs | The application logs |
| Settings | App settings |
| Users | User accounts, and your own preferences |
| Personal | Your favorites, bookmarks, saved filters, saved playlists and soundboards, and themes |

You're only offered what your role can use. A player sees game systems as Read at most, since players can't edit them, and never sees admin-only areas like Logs or the file manager.

A few things are **never available to a key**: signing in, managing API keys, and changing a credential or the account itself (password, deleting the account, OPDS and calendar feed tokens).

**Shown once.** The full key appears only when it's created or regenerated. Grimoire stores just a hash of it and afterwards shows only its first few characters. A lost key can't be recovered, only regenerated.

**Errors.** An unknown or expired key gets `401`. A valid key without the permission a request needs gets `403`, with a message naming the permission and level it needs. A key also gets `403` once its owner loses API key access, or while `API_KEYS_ENABLED=false`. Too many wrong keys from one address get `429` (see [Security](/configuration/security)).

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
| `/api/auth/status` | GET | None | Returns `{"initialized": bool}` |
| `/api/auth/config` | GET | None | Public auth config for the login screen |
| `/api/auth/setup` | POST | None | First-run admin account creation. Body: `{username, password}` |
| `/api/auth/login` | POST | None | Authenticate. Body: `{username, password}`. Returns `{token, user}` |
| `/api/auth/me` | GET | any | Current user details |
| `/api/auth/openid/login` | GET | None | Start OIDC login, redirects to IdP |
| `/api/auth/openid/callback` | GET | None | OIDC callback |
| `/api/auth/openid/discover` | POST | admin | Server-side discovery fetch. Body: `{issuer_url}` |

---

## Users

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/users` | GET | admin | List all users |
| `/api/users` | POST | admin | Create a user. Body: `{username, password, role?, email?}` |
| `/api/users/:id` | PATCH | admin | Update `role`, `password`, `allow_explicit`, or `email` |
| `/api/users/:id` | DELETE | admin | Delete a user |
| `/api/users/me/preferences` | PATCH | any | Update own `display_name`, `allow_explicit`, or `email` |
| `/api/users/me/password` | PATCH | any | Change own password. Body: `{current_password, new_password}` |
| `/api/users/me` | DELETE | any | Delete own account |

---

## Library

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/stats` | GET | JWT, or an API key with Statistics: Read | Counts, page totals, library size |
| `/api/scan-status` | GET | admin | Current scan state |
| `/api/rescan` | POST | admin | Trigger a background rescan and reindex |
| `/api/cancel-scan` | POST | admin | Gracefully stop the running scan |

**Stats response:**
```json
{
  "game_systems": 12,
  "books": 340,
  "maps": 1500,
  "tokens": 800,
  "audio": 250,
  "models": 640,
  "indexed_books": 320,
  "total_pages": 45000,
  "total_size_mb": 18240.5,
  "library_size_mb": 41890.2
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
        - field: books
          label: Books
          format: number
        - field: maps
          label: Maps
          format: number
        - field: tokens
          label: Tokens
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
| `game_systems` | Number of game systems (top-level library folders) | `number` |
| `books` | Total books (PDFs) | `number` |
| `maps` | Total maps | `number` |
| `tokens` | Total tokens | `number` |
| `audio` | Total audio tracks | `number` |
| `models` | Total 3D models | `number` |
| `indexed_books` | Books with a searchable full-text index | `number` |
| `total_pages` | Sum of all book page counts | `number` |
| `total_size_mb` | Size of the books in MB (books only) | `float`, add `scale: 0.001` + `suffix: " GB"` to show GB |
| `library_size_mb` | Size of the whole library in MB — books plus maps, tokens, audio, and models | `float`, add `scale: 0.001` + `suffix: " GB"` to show GB |

---

## Game Systems

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/systems` | GET | any | List all systems with book counts |
| `/api/systems/:id` | GET | any | System detail + full book list |
| `/api/systems/:id` | PATCH | gm/admin | Update metadata |

**PATCH fields:** `name`, `slug`, `description`, `publishers`, `character_builder_url`, `cover_image`, `cover_book_id`, `tags`, `genre`, `is_explicit`

---

## Books

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/books` | GET | any | Paginated book list. Query: `system_id`, `category`, `limit` (max 500), `offset` |
| `/api/books/:id` | GET | any | Book detail |
| `/api/books/:id` | PATCH | gm/admin | Update book metadata |
| `/api/books/:id/reindex` | POST | gm/admin | Re-run OCR on a scanned book. Optional query `ocr_dpi` (72–600) re-reads it at a higher resolution than the global `OCR_DPI`. Clears the search index and re-queues the book (OCR runs in the background). 400 for books with an embedded text layer. |
| `/api/books/:id/rescan` | POST | gm/admin | Re-read a single book from disk and rebuild its search index, for a file edited externally. Works for any PDF: a text-layer book is re-extracted; an image-only book is re-queued for OCR. Refreshes page count and thumbnail. Runs in the background; no-ops if a library scan is already running. 400 for non-PDFs. |
| `/api/books/:id/file` | GET | any | Download/stream the file |
| `/api/books/:id/thumbnail` | GET | any | WebP cover thumbnail |
| `/api/books/:id/toc` | GET | any | PDF table of contents |
| `/api/books/:id/page/:num` | GET | any | Render PDF page as WebP. Query: `width` (default 1200, max 3000) |
| `/api/books/:id/page/:num/text` | GET | any | Plain text of a page |
| `/api/books/:id/page/:num/words` | GET | any | Word bounding boxes for text overlay |

---

## Maps

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/maps` | GET | any | Paginated map list. Query: `limit`, `offset`, `tag` |
| `/api/maps/:id` | GET | any | Map detail |
| `/api/maps/:id` | PATCH | gm/admin | Update `description`, `tags`, `map_type`, `grid_size` |
| `/api/maps/:id/file` | GET | any | Download/stream the original map file |
| `/api/maps/:id/page/:n` | GET | any | Downscaled WebP render. Query: `width` (default 1600, max 3000) |
| `/api/maps/:id/vtt/image` | GET | any | Battlemap image decoded out of a `.uvtt`/`.dd2vtt` file |
| `/api/maps/:id/vtt/data` | GET | any | Grid resolution and wall/portal/light counts from a Universal VTT file |
| `/api/maps/:id/export.uvtt` | GET | any | Builds a Universal VTT file for a raster map: the image, the grid, and anything authored in the editor |
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
| `/api/tokens` | GET | any | Paginated token list. Query: `limit`, `offset`, `tag` |
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
| `/api/models` | GET | any | Paginated 3D model list. Query: `limit`, `offset` |
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

## Search

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/search?q=` | GET | any | FTS5 full-text search. Min 2 chars. Optional: `book_id`, `system_id`, `limit` |

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
| `/api/campaigns/:id` | DELETE | owner or admin | Delete campaign |

### Members

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/invite` | POST | owner or admin | Invite a user. Body: `{user_id}` |
| `/api/campaigns/:id/members/:user_id` | PATCH | member or owner | Accept/decline or set character name |
| `/api/campaigns/:id/members/:user_id` | DELETE | owner, admin, or self | Remove member |

### Sessions

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/sessions` | GET | member or owner | List sessions |
| `/api/campaigns/:id/sessions` | POST | member or owner | Create session. Body: `{session_date, title?}` |
| `/api/campaigns/:id/sessions/:sid` | PATCH | owner or admin | Update title |
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

`frequency` values: `weekly`, `biweekly`, `monthly`, `custom`. Days are 0 (Monday) – 6 (Sunday).

### Availability

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/campaigns/:id/availability` | GET | member or owner | Availability for next 10 scheduled sessions |
| `/api/campaigns/:id/availability/:date` | PUT | member or owner | Set own availability. Body: `{status}` |
| `/api/campaigns/:id/availability/:date/cancel` | PUT | owner or admin | Toggle session cancellation |

Statuses: `available`, `tentative`, `unavailable`

---

## Settings *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/settings` | GET | Get all application settings |
| `/api/settings` | PATCH | Update application settings |
| `/api/settings/ui` | GET | UI visibility flags (any authenticated user) |

---

## API keys

Manage your [API keys](#api-keys). These need a login: a key can never manage keys.

| Endpoint | Method | Description |
|---|---|---|
| `/api/api-keys` | GET | List your keys (never the key itself). Admins can add `?all=true` for everyone's |
| `/api/api-keys/permissions` | GET | The permissions your role can grant, with descriptions and the levels each offers |
| `/api/api-keys` | POST | Create a key. Body: `{name, permissions?, expires_at?}`, where `permissions` is `{permission: "none" \| "read" \| "write"}` and `"*"` means all permissions. Returns `{api_key, key}`: `key` is the full key, returned only here and by regenerate |
| `/api/api-keys/:id` | PATCH | Rename your key or change its permissions or expiry (`expires_at: null` means never) |
| `/api/api-keys/:id/regenerate` | POST | Issue a new key with the same name and permissions. The old key stops working immediately |
| `/api/api-keys/:id` | DELETE | Revoke your key. Admins can revoke anyone's |

---

## Maintenance *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/maintenance/cleanup-missing` | POST | Remove DB records for files no longer present on disk |

---

## Backups *(admin only)*

| Endpoint | Method | Description |
|---|---|---|
| `/api/backups` | GET | List backups, newest first, with `created_at`, `size_bytes`, and `version` |
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
