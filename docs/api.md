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

All endpoints except `/api/auth/status`, `/api/auth/setup`, `/api/auth/login`, and `/api/auth/config` require a JWT.

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
| `/api/stats` | GET | JWT or `X-API-Key` | Counts, page totals, library size, version |
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
  "indexed_books": 320,
  "total_pages": 45000,
  "total_size_mb": 18240.5
}
```

`/api/stats` is the only endpoint that accepts the `X-API-Key` header, so it's the safe way to surface library counts on an external dashboard. Generate a key as an admin under **Settings → App Settings → Stats API Key** (regenerate or revoke it there at any time). See the [Homepage Widget guide](/guide/homepage) for a full walkthrough.

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
        - field: total_size_mb
          label: Size
          format: float
          scale: 0.001
          suffix: " GB"
```

Homepage shows up to four fields per row; pick the counts you care about from the fields below. Use a `refreshInterval` of 60s or higher, `/api/stats` is rate limited.

| Field | Meaning | Suggested `format` |
|---|---|---|
| `game_systems` | Number of game systems (top-level library folders) | `number` |
| `books` | Total books (PDFs) | `number` |
| `maps` | Total maps | `number` |
| `tokens` | Total tokens | `number` |
| `audio` | Total audio tracks | `number` |
| `indexed_books` | Books with a searchable full-text index | `number` |
| `total_pages` | Sum of all book page counts | `number` |
| `total_size_mb` | Total library size in MB | `float`, add `scale: 0.001` + `suffix: " GB"` to show GB |

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

---

## Tokens

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/tokens` | GET | any | Paginated token list. Query: `limit`, `offset`, `tag` |
| `/api/tokens/:id` | GET | any | Token detail |
| `/api/tokens/:id` | PATCH | gm/admin | Update `description`, `tags`, `is_explicit` |
| `/api/tokens/:id/file` | GET | any | Download the token image |
| `/api/tokens/:id/thumbnail` | GET | any | WebP thumbnail |
| `/api/token-folders` | GET | any | List folder tag assignments |
| `/api/token-folders` | PATCH | gm/admin | Set tags on a folder. Body: `{path, tags}` |

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
| `/api/settings/api-key/generate` | POST | Generate a stats API key |
| `/api/settings/api-key` | DELETE | Revoke the stats API key |

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
