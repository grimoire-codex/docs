# Homepage Widget

Grimoire exposes a lightweight stats endpoint (`/api/stats`) designed for
external dashboards, authenticated with an [API key](/api#api-keys). The most common use is a
[Homepage](https://gethomepage.dev) [Custom API widget](https://gethomepage.dev/widgets/services/customapi/),
which shows your library counts on your dashboard without logging in.

## 1. Create an API key

1. Sign in to Grimoire. Use an **admin** account if the widget should count the whole library: a key counts what its owner can see. Other users need an admin to grant them API keys first (**Settings → Users**).
2. Go to **Settings → Account**, open **Security**, and under **API Keys** click **Create API key**.
3. Give it a name, such as `Homepage widget`, and pick an expiration.
4. Set **Statistics** to **Read**. Leave everything else at **No access**.
5. Click **Create key**, then **copy the key**.

::: warning Copy the key now
Grimoire shows the full key **once**, right after you create it. Only a hash of it
is stored, so it can't be shown again. If you lose it, click **Regenerate** on the
key to issue a new one with the same name and permissions (the old one stops
working immediately).
:::

With only **Statistics: Read**, the key can read `/api/stats` and nothing else. It
can't reach books, files, users, or build details. You can **Edit**, **Regenerate**,
or **Revoke** it at any time, and the key list shows when it was last used.

::: tip
The key is sent in the `X-API-Key` request header. Anyone with the key can do
whatever its permissions allow, so treat it like a password.
:::

::: info Upgrading from an older version?
The single "Stats API key" from earlier versions is migrated automatically. **Your
existing key keeps working**, so there's nothing to change in Homepage. It appears in
the first admin's key list as **Stats API key (migrated)**, with Statistics: Read only. The difference is
that Grimoire no longer shows the key itself; if you need it again, regenerate it and
update Homepage.
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
- **Refresh interval**: requests with a working key are never rate limited, but
  a `refreshInterval` of `60000` (60s) is plenty, since the counts only change
  when the library does.
- **401 Unauthorized**: the key is wrong, has expired, was revoked or
  regenerated, or the header name isn't exactly `X-API-Key`.
- **403 Forbidden**: the key is valid but lacks **Statistics: Read** (edit it
  under **Settings → Account → API Keys**), its owner no longer has API key
  access, or API keys are turned off for the instance (`API_KEYS_ENABLED=false`).
- **429 Too Many Requests**: too many *wrong* keys were sent from Homepage's
  address. Fix the key and wait a minute.
