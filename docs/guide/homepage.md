# Homepage Widget

Grimoire exposes a lightweight stats endpoint (`/api/stats`) designed for
external dashboards, authenticated with an [API key](/api#api-keys). The most common use is a
[Homepage](https://gethomepage.dev) [Custom API widget](https://gethomepage.dev/widgets/services/customapi/),
which shows your library counts on your dashboard without logging in.

## 1. Create an API key

1. Sign in to Grimoire. Use an **admin** account if the widget should count the whole library: a key counts what its owner can see. Other users need an admin to grant them API keys first (**Settings → People**).
2. Go to **Settings → Connections**, and under **API Keys** click **Create API key**.
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

Response (book counts and sizes cover only what the key's owner may see; each total is its role-playing and wargame halves added up; item counts leave out variants, which the `*_variants` fields count):

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

## 3. Add the widget to Homepage

In your Homepage `services.yaml`, add a service with a `customapi` widget:

```yaml
- Grimoire:
    icon: grimoire.png
    href: https://grimoire.example.com
    description: Tabletop Library
    widget:
      type: customapi
      url: https://grimoire.example.com/api/stats
      refreshInterval: 60000
      method: GET
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

Homepage shows up to four fields per row. Pick the four counts that matter most
to you, for example swap in `ttrpg_systems`, `wargame_pages`, `audio`, or
`maps_size_mb`. To show both halves of a total side by side, use a pair such as
`ttrpg_books` and `wargame_books`, or `maps` and `map_variants`.

### Available fields

| Field | Meaning | Suggested `format` |
|-------|---------|--------------------|
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

`armory_books` stays for widgets set up before the split, so an older `services.yaml` keeps working unchanged.

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
  under **Settings → Connections → API Keys**), its owner no longer has API key
  access, or API keys are turned off for the instance (`API_KEYS_ENABLED=false`).
- **429 Too Many Requests**: too many *wrong* keys were sent from Homepage's
  address. Fix the key and wait a minute.
