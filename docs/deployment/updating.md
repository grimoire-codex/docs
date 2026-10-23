# Updating

## Docker Compose

```bash
docker compose pull
docker compose up -d
```

Your data volume (database, thumbnails, search index, user accounts) is stored separately and is not affected by updates.

## Pinning to a version

Replace `latest` with a specific tag to pin to a release:

```yaml
image: hunterreadca/grimoire:1.5.0
```

Check [Docker Hub](https://hub.docker.com/r/hunterreadca/grimoire/tags) for available tags.

## Docker Hub or GitHub Container Registry

Every release and `nightly` tag is published to both registries from the same build, so the images are identical:

| Registry | Image |
| --- | --- |
| Docker Hub | `hunterreadca/grimoire:<tag>` |
| GitHub Container Registry | `ghcr.io/hunter-read/grimoire:<tag>` |

To switch, change the `image:` line in your compose file and run `docker compose pull && docker compose up -d`. Your data volume is unaffected.

## Image variants

The default tags (`latest`, `1.5.0`, …) bundle the Tesseract OCR engine so image-only PDFs are searchable. If you don't need [OCR](/configuration/performance#ocr) and prefer a smaller image, use the matching `-slim` tag (e.g. `:slim`, `:1.5.0-slim`), which omits Tesseract. OCR is automatically disabled on slim images and Grimoire degrades gracefully.

## Database migrations

Schema changes are applied automatically on startup via [Alembic](https://alembic.sqlalchemy.org/): **no manual action is required** when upgrading, including from versions that predate Alembic. On first run under the new system, an existing database is detected and stamped at the correct baseline, so only genuinely new migrations run thereafter.

Your data volume (database, thumbnails, search index, user accounts) is stored separately and is not affected by pulling a new image. Back up `DATA_PATH` before upgrading, as always.

## After updating

- If the update includes new library scanning logic, trigger a **Rescan** from **Settings → Library** to pick up any new metadata.
- If you moved from a `-slim` image to the OCR-capable image (or set `OCR_ENABLED=true`), previously image-only books are automatically re-queued for OCR on the next startup scan.
