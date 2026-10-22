# API Reference

**Version:** 0.1.0

## Interactive docs

With the server running, the API is self-documented via OpenAPI:

| URL | Description |
|---|---|
| `http://localhost:9481/api/docs` | Swagger UI: interactive, try-it-out docs |
| `http://localhost:9481/api/redoc` | ReDoc: clean, readable reference |
| `http://localhost:9481/api/openapi.json` | Raw OpenAPI schema |

These need a signed-in session by default (open them in a browser where you're signed in). Set [`API_DOCS`](/configuration/env-vars) to `public` to serve them to anyone, or `off` to turn them off.

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

A few things are **never available to a key**: signing in, managing API keys, and changing a credential or the account itself (password, deleting the account, OPDS and calendar feed tokens). Guest invite codes (creating, listing, regenerating and sharing them), the OIDC sign-in button icon, the add-on source settings (`PATCH /api/addons/settings`) and private source tokens (`/api/addons/sources/credential`) are session-only too.

**Session-only fields.** Some routes a key can use take a few fields that only a signed-in session may set. Sent with a key they get a `403` naming the field; the rest of the route works as usual:

- `POST /api/users` and `PATCH /api/users/:id`: `password`, the `admin` role, and (on `PATCH`) a changed `email`. Converting a guest to a user likewise.
- `PATCH /api/users/me/preferences`: a changed `email`.
- `PATCH /api/settings`: password sign-in, guest access, the login message, and every OIDC setting.
- Approving an add-on's script, on install (`approve_script: true`) or afterwards (`script_approved: true`).

Campaign responses never include a guest's invite code when the request uses a key.

**Share links with a key.** Creating or changing a [share link](#share-links) with a key also needs the key to have **Read** on the item's own area - Books, Maps, Tokens, Audio or 3D models - besides **Share links: Read and write**.

**Shown once.** The full key appears only when it's created or regenerated. Grimoire stores just a hash of it and afterwards shows only its first few characters. A lost key can't be recovered, only regenerated.

**Errors.** An unknown or expired key gets `401`. A valid key without the permission a request needs gets `403`, with a message naming the permission and level it needs. A key also gets `403` once its owner loses API key access, or while `API_KEYS_ENABLED=false`. Once an address has sent too many wrong keys, further wrong keys from it get `429`; a working key is never refused (see [Security](/configuration/security#auth-rate-limiting)).

**MCP.** With `MCP_ENABLED=true`, keys also authenticate the [MCP server](/guide/mcp) at `POST /mcp`. Each MCP tool, resource and prompt checks the permission it needs, and a key is only shown the ones it can use.

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
| `/api/auth/me` | GET | any | Current user details |
| `/api/auth/openid/login` | GET | None | Start OIDC login, redirects to IdP |
| `/api/auth/openid/callback` | GET | None | OIDC callback |
| `/api/auth/openid/discover` | POST | admin | Server-side discovery fetch. Body: `{issuer_url}` |

---

## Users

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/users` | GET | admin | List all users |
| `/api/users` | POST | admin | Create a user. Body: `{username, password, role?, email?}`. The password must meet the [password rules](/configuration/security#password-rules) |
| `/api/users/:id` | PATCH | admin | Update `role`, `password`, `allow_explicit`, or `email` |
| `/api/users/:id` | DELETE | admin | Delete a user |
| `/api/users/me/preferences` | PATCH | any | Update own `display_name`, `allow_explicit`, or `email`. While [OIDC](/guide/oidc) matches existing users by email, only an admin can change an email |
| `/api/users/me/password` | PATCH | any | Change own password. Body: `{current_password, new_password}`. The new password must meet the [password rules](/configuration/security#password-rules) |
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
| `/api/systems/:id` | GET | any | System detail + full book list. `include_books=false` returns the summary alone |
| `/api/systems/:id/books` | GET | any (not guest) | One page of the system's books matching the shelf filters. Query: filters, `category` + `folder` (one folder of the grouped shelf), `sort`, `order`, `limit`, `offset` |
| `/api/systems/:id/book-groups` | GET | any (not guest) | Every category and subfolder holding matching books, with counts |
| `/api/systems/:id/book-facets` | GET | any (not guest) | The tags, genres, product-code prefixes and categories on the shelf |
| `/api/systems/:id` | PATCH | gm/admin | Update metadata |

**PATCH fields:** `name`, `slug`, `description`, `publishers`, `character_builder_url`, `cover_image`, `cover_book_id`, `tags`, `genre`, `is_explicit`

---

## Books

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/books` | GET | any | Paginated book list. Query: `system_id`, `category`, `limit` (max 500), `offset`, `sort` (`title` or `added_at`), `order` (`asc`/`desc`; `added_at` defaults to newest first), `added_since` (ISO-8601, UTC if no offset). Each book carries `added_at` - when it first appeared in the library, or `null` for items from the first import of a new install (and older rows with no date). Poll `sort=added_at&added_since=<last poll>` to find new additions. |
| `/api/books/:id` | GET | any | Book detail |
| `/api/books/:id` | PATCH | gm/admin | Update book metadata |
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
| `/api/maps/:id` | GET | any | Map detail |
| `/api/maps/:id` | PATCH | gm/admin | Update `description`, `tags`, `map_type`, `grid_size` |
| `/api/maps/:id/file` | GET | any | Download/stream the original map file |
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

The gallery list routes (`/api/maps`, `/api/tokens`, `/api/audio`, `/api/models`) and their `/groups` companions filter, sort and group on the server, so a client can browse a library of any size a page at a time. They share these parameters:

| Parameter | Meaning |
|---|---|
| `q` | Search text over the name, folder path and tags; accepts `field:` prefixes (`tag:`, `title:`, `artist:`, `album:`) |
| `tags` | Tag filter as JSON: `[{"mode": "include"\|"exclude", "tags": [...]}]` groups (OR within a group, AND between groups), or a flat list. Folder tags count for everything beneath the folder. `__grim:none__` / `__grim:any__` match untagged / tagged items |
| `favorites` | `true` for the caller's favourites only |
| `added_since` | Items added at or after this ISO time |
| `folder` | Only items directly in this folder (`""` for the collection root) |
| `sort`, `order` | `path` (default), `name` (natural order), `size`, `added_at`; audio also `title`, `duration`, `track` (folder, then disc and track number, then name). `asc` or `desc` |

`/groups` returns `{total, groups: [{path, count}]}` - every folder holding matching items. A tag's items are paged the same way: `/api/tags/:internal/items` returns per-type counts, the tagged folders with counts, and a page of directly-tagged items (`resource_type`, `limit`, `offset`), and `/api/tags/:internal/folder-items` pages through one tagged folder.

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

## Characters

Sheets (schemas) and characters are stored **per user**, so any signed-in user — guests included — may install a sheet. See [Characters](/guide/characters) for how the builder works.

### Sheets

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/characters/schemas` | GET | any | Installed sheets, each with a `character_count` |
| `/api/characters/schemas` | POST | any | Install a pasted sheet. Body: `{text}` (JSON or YAML, without YAML aliases, at most 512 KB and 100 levels deep) or `{document}`, plus optional `layout` and `styles`. 400 names the problem if it does not validate |
| `/api/characters/schemas/browse` | GET | any | The community sheet catalogue, each marked `installed` |
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
| `/api/content/:schema_id/:content_type` | GET | any | Browse: `search`, `filter[field]=value`, `sort`, `page`, `page_size` |
| `/api/content/:schema_id/:content_type/:entry_id` | GET | any | One entry in full |

### Rulesets

A campaign ruleset is read by everyone at that table and edited by the campaign's owner; a server ruleset (`campaign_id` null) is read by everyone and edited by admins; a personal ruleset (`personal: true`) is read and edited by its owner alone. Importing a character creates a personal ruleset for the content embedded in its file.

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/rulesets` | GET | any | Rulesets you can read. `schema_id` and `campaign_id` filter |
| `/api/rulesets` | POST | campaign owner, or admin for a server ruleset | Create one |
| `/api/rulesets/packs/browse` | GET | any | The community content-pack catalogue, with `installed` and `update_available` |
| `/api/rulesets/packs/install/:pack_id` | POST | admin | Install or update a content pack |
| `/api/rulesets/packs/:pack_id` | DELETE | admin | Uninstall a content pack |
| `/api/rulesets/:id` | GET / PUT / DELETE | read: reader; write: editor | One ruleset |
| `/api/rulesets/:id/export` | GET | reader | The ruleset as a portable document |
| `/api/rulesets/:id/import` | POST | editor | Import a `pack_id`, a `document`, or `text` (JSON or YAML). `conflict`: `skip` (default), `overwrite`, `rename` |
| `/api/rulesets/:id/fork` | POST | editor | Copy an entry into this ruleset to change it |
| `/api/rulesets/:id/entries` | GET / POST | read: reader; write: editor | List or add entries |
| `/api/rulesets/:id/entries/:entry_id` | GET / PUT / DELETE | read: reader; write: editor | One entry |

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
