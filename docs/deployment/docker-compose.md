# Docker Compose

The fastest way to get a working compose file is the **[Compose Generator](/compose-generator)**: it fills in your paths, generates a secret key, and lets you download the file directly.

The examples below pull from Docker Hub (`hunterreadca/grimoire`). Every release and `nightly` tag is also published to the GitHub Container Registry, so you can swap in `ghcr.io/hunter-read/grimoire` with the same tag if you prefer.

## Minimal setup

```yaml
services:
  grimoire:
    image: hunterreadca/grimoire:latest
    container_name: grimoire
    restart: unless-stopped
    ports:
      - "9481:9481"
    environment:
      - LIBRARY_PATH=/library
      - DATA_PATH=/data
      - WORKERS=1
      - SECRET_KEY=<paste the generated value here>   # openssl rand -hex 32, or delete this line
    volumes:
      - /path/to/your/library:/library:ro
      - /path/to/grimoire/data:/data
```

## With Valkey cache

Recommended for large libraries or multi-user setups. Keeps rendered PDF page images in memory for faster repeat loads.

```yaml
services:
  grimoire:
    image: hunterreadca/grimoire:latest
    container_name: grimoire
    restart: unless-stopped
    ports:
      - "9481:9481"
    environment:
      - LIBRARY_PATH=/library
      - DATA_PATH=/data
      - WORKERS=1
      - SECRET_KEY=<paste the generated value here>
      # Valkey also holds sign-in counters, so it has a password: generate one
      # (openssl rand -hex 24) and use the same value on the valkey service.
      - VALKEY_URL=redis://:<valkey password>@valkey:6379/0
    volumes:
      - /path/to/your/library:/library:ro
      - /path/to/grimoire/data:/data
    depends_on:
      - valkey
    networks:
      - grimoire

  valkey:
    image: valkey/valkey:8-alpine
    container_name: grimoire-valkey
    restart: unless-stopped
    command: ["valkey-server", "--requirepass", "<valkey password>"]
    volumes:
      - valkey_data:/data
    networks:
      - grimoire

volumes:
  valkey_data:

networks:
  grimoire:
```

## Build from source

```bash
docker build --build-arg APP_VERSION=1.0.0 -t grimoire:1.0.0 .
docker compose up -d
```

## Updating

```bash
docker compose pull
docker compose up -d
```

Your data volume is unaffected by updates.

## Pre-made example files

Ready-to-copy compose files are in the [grimoire repository](https://github.com/hunter-read/grimoire/tree/main/docs/docker):

| File | What it includes |
|---|---|
| `docker-compose.yml` | Grimoire only (minimal) |
| `docker-compose.valkey.yml` | + Valkey page cache |
| `docker-compose.calibre.yml` | + Calibre full desktop |
| `docker-compose.calibre-web.yml` | + Calibre-Web |
