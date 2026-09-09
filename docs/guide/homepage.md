# Homepage Widget

Grimoire exposes a lightweight, API-key-gated stats endpoint (`/api/stats`) that
is designed for external dashboards. The most common use is a
[Homepage](https://gethomepage.dev) [Custom API widget](https://gethomepage.dev/widgets/services/customapi/),
which shows your library counts on your dashboard without logging in.

## 1. Generate a stats API key

1. Sign in to Grimoire as an **admin**.
2. Go to **Settings → App Settings → Stats API Key**.
3. Click **Generate API Key** and copy the key.

This key grants read-only access to `/api/stats` only. It does **not** expose
books, files, users, or build details. You can **Regenerate** (invalidates the
old key) or **Revoke** it at any time.

::: tip
The key is passed in the `X-API-Key` request header. Any client with the key can
read your library counts, so treat it like a password.
:::

## 2. The stats endpoint

```
GET https://grimoire.example.com/api/stats
X-API-Key: <your-key>
```

Response:

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

## 3. Add the widget to Homepage

In your Homepage `services.yaml`, add a service with a `customapi` widget:

```yaml
- Grimoire:
    icon: grimoire.png
    href: https://grimoire.example.com
    description: TTRPG Library
    widget:
      type: customapi
      url: https://grimoire.example.com/api/stats
      refreshInterval: 60000
      method: GET
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

Homepage shows up to four fields per row. Pick the four counts that matter most
to you, for example swap in `game_systems`, `audio`, `indexed_books`, or
`total_pages`.

### Available fields

| Field | Meaning | Suggested `format` |
|-------|---------|--------------------|
| `game_systems` | Number of game systems (top-level library folders) | `number` |
| `books` | Total books (PDFs) | `number` |
| `maps` | Total maps | `number` |
| `tokens` | Total tokens | `number` |
| `audio` | Total audio tracks | `number` |
| `models` | Total 3D models | `number` |
| `indexed_books` | Books with a searchable full-text index | `number` |
| `total_pages` | Sum of all book page counts | `number` |
| `total_size_mb` | Size of the books in MB (books only) | `float` (`scale: 0.001`, `suffix: " GB"` for GB) |
| `library_size_mb` | Size of the whole library in MB — books plus maps, tokens, audio, and models | `float` (`scale: 0.001`, `suffix: " GB"` for GB) |

## Notes

- **HTTPS / reverse proxy**: Homepage must be able to reach Grimoire's URL. If
  Grimoire is behind a reverse proxy, use its public URL and make sure the
  `X-API-Key` header is forwarded (most proxies pass all headers by default).
- **Rate limiting**: `/api/stats` is rate limited. A `refreshInterval` of
  `60000` (60s) or higher is plenty for a dashboard and stays well within limits.
- **401 Unauthorized**: the key is wrong, was revoked/regenerated, or the
  header name isn't exactly `X-API-Key`.
