# Quick Start

> New to Docker? See the [Docker Install Guide](/guide/docker-install) for a step-by-step walkthrough on Windows, macOS, and Linux.

## 1. Organize your library

Create a `library/` folder with this structure:

```
library/
├── books/
│   └── Dungeons and Dragons 5e/
│       ├── core/
│       │   ├── Players Handbook.pdf
│       │   └── Dungeon Masters Guide.pdf
│       ├── supplements/
│       ├── adventures/
│       │   └── Curse of Strahd/
│       │       └── Curse of Strahd.pdf
│       └── homebrew/
├── maps/
│   └── Sunken Temple (22x22)/
│       └── The Sunken Temple.png
└── tokens/
    └── Monsters/
        └── goblin.png
```

See [Library Structure](/guide/library-structure) for the full layout and category rules.

## 2. Get a compose file

Use the **[Compose Generator](/compose-generator)** to build a tailored `docker-compose.yml` for your setup, or copy one of the ready-made examples:

```bash
# Minimal (no extras)
curl -O https://raw.githubusercontent.com/hunter-read/grimoire/main/docs/docker/docker-compose.yml

# With Valkey page cache (recommended for large libraries)
curl -O https://raw.githubusercontent.com/hunter-read/grimoire/main/docs/docker/docker-compose.valkey.yml
mv docker-compose.valkey.yml docker-compose.yml
```

## 3. Set your secret key

Edit the compose file and replace `change-me` with a long random string:

```bash
# Generate one with:
openssl rand -hex 32
```

Also update the two volume paths to point at your library and data folders.

## 4. Start Grimoire

```bash
docker compose up -d
```

Open [http://localhost:9481](http://localhost:9481). On first launch you'll be prompted to create an admin account.

## 5. Trigger a rescan

Grimoire scans your library on startup. For large collections this runs in the background and can take a few minutes. You can browse immediately while it works.

After adding new files later, click **Rescan** at the top of any Server settings page (admins) or go to **Settings → Library → Scanning → Rescan Library**.

## Stopping and updating

```bash
# Stop
docker compose down

# Update to latest
docker compose pull && docker compose up -d
```

Your library and all data (bookmarks, metadata, accounts) are stored in the data volume and are not affected by updates.
