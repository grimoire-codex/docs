# Backups

Grimoire can take a snapshot of its own database and the files you have uploaded through
it, on demand or on a schedule. You'll find it under **Settings → Maintenance → Backups**.

Each backup is a single timestamped `.zip`:

```
grimoire-backup-20260821T140355Z.zip
├── details.json        manifest: app version, timestamp, what is inside
├── grimoire.db         the SQLite database
├── campaign_uploads/   banners, character art, sheets, campaign files
├── system_covers/      custom game-system cover images
└── audio_covers/       custom audio cover art
```

The database is copied using SQLite's online backup API rather than a plain file copy, so
the snapshot is internally consistent even while Grimoire is running. Database writes are
paused for the length of the snapshot. That is brief for a typical library, but not
instant.

::: danger Your library is not backed up
Backups do **not** include your PDFs, maps, tokens, or audio files. The library is mounted
read-only and is usually far too large to copy on a schedule, so Grimoire never touches
it. **Back your library up separately.**
:::

Thumbnails and rendered pages are excluded too, since both regenerate on demand. After a
restore, the first view of a book or map is a little slower while they rebuild. Nothing is
lost.

## Please do not rely on these alone

Backups are written to the same machine Grimoire runs on. A failed disk, a ransomware
event, or a mistaken `rm -rf` takes the backups along with the original.

They protect you against **application-level** accidents: a bad rescan, a cleanup that
removed more than you meant, a metadata overwrite. That is genuinely worth having. It is
not disaster recovery.

::: tip Follow 3-2-1
- **3** copies of your data (the live one plus two backups)
- **2** different kinds of storage (internal disk and an external drive, or a NAS)
- **1** copy off-site (another building, or a cloud sync target)
:::

In practice: point `BACKUP_DIR` at a mounted volume on a different disk, and sync that
directory somewhere off-site on a schedule. [`rclone`](https://rclone.org/),
[`restic`](https://restic.net/), [Syncthing](https://syncthing.net/), or your NAS's own
backup job all work. Do the same for your library directory.

## Configuration

Every setting below is configurable in the UI. Setting the matching environment variable
**pins** it: the value wins, and the field becomes read-only in Settings.

| Variable | Default | Description |
|---|---|---|
| `BACKUP_DIR` | `DATA_PATH/backups` | Where archives are written. Point at another volume to keep backups off the main disk. |
| `BACKUP_SCHEDULE` | `off` | `off`, `hourly`, `daily`, or `weekly`. |
| `BACKUP_RETENTION_COUNT` | `0` | Keep at most this many backups. `0` = unlimited. |
| `BACKUP_RETENTION_GB` | `0` | Keep at most this many GB in total. `0` = unlimited. |

### Scheduling

Pick `off`, `hourly`, `daily`, or `weekly`. Daily and weekly schedules also take a time of
day (and a weekday), entered in **your local timezone** and stored as UTC, so a backup
runs at the same wall-clock time regardless of when the server was last restarted.

Backups run in a background thread. A failed run is logged and does not stop the next one
from happening.

### Retention

The two limits apply **independently**. A backup is removed once *either* is passed:

- `BACKUP_RETENTION_COUNT`: keep at most N backups
- `BACKUP_RETENTION_GB`: keep at most N gigabytes in total

Old backups are deleted oldest-first. Both default to `0`, meaning unlimited.

::: warning Two things to know about pruning
**At least one backup is always kept**, even if it exceeds the size limit on its own. A
single oversized archive is better than none.

**Pruning happens *after* a new backup is written**, not before, so the configured ceiling
can be briefly exceeded while a backup runs. Leave disk headroom for one extra archive.
:::

### Storage location

By default backups go to `DATA_PATH/backups`. Backups are never nested inside other
backups, so this is safe.

Setting `BACKUP_DIR` (or the field in the UI) points them anywhere else, ideally a volume
on a different physical disk. If you set it in the UI, Grimoire checks the directory is
writable when you save, rather than letting the first scheduled run fail silently at 3am.

With Docker, remember to mount the volume:

```yaml
services:
  grimoire:
    volumes:
      - ./library:/library:ro
      - ./data:/app/data
      - /mnt/backups/grimoire:/backups   # [!code ++]
    environment:
      - BACKUP_DIR=/backups              # [!code ++]
      - BACKUP_SCHEDULE=daily            # [!code ++]
      - BACKUP_RETENTION_COUNT=7         # [!code ++]
```

## Restoring

Restoring is deliberately **not** something Grimoire does for you. It means replacing the
live database underneath a running application. That is safe when a human does it with the
server stopped, and genuinely dangerous when a web request can trigger it.

The full procedure is in
[restore-from-backup.md](https://github.com/hunter-read/grimoire/blob/main/restore-from-backup.md)
in the repository. In short:

1. **Stop Grimoire.** Do not skip this.
2. Move the current `data/` aside (do not delete it, it is your fallback).
3. Unpack the archive into a fresh `data/`.
4. Check it: `sqlite3 data/grimoire.db "PRAGMA integrity_check;"` should print `ok`.
5. Fix ownership to match `PUID`/`PGID` if you run Docker.
6. Start Grimoire and watch the logs. Migrations run here.
7. Trigger a rescan to rebuild thumbnails.

Restoring into the **same** Grimoire version is always safe. Restoring into a **newer**
one is normally fine, since migrations bring the schema forward on startup. Restoring into an
**older** version is not supported. The `details.json` manifest records which version wrote
the archive:

```bash
unzip -p grimoire-backup-20260821T140355Z.zip details.json
```

## From the API

The endpoints are admin-only and need a bearer token. `GET /api/backups` lists every
backup newest-first with its `created_at`, `size_bytes`, and `version`; `POST /api/backups`
takes one now.

```bash
# List backups, newest first
curl -s http://localhost:9481/api/backups \
  -H "Authorization: Bearer $TOKEN"

# Take one now
curl -s -X POST http://localhost:9481/api/backups \
  -H "Authorization: Bearer $TOKEN"
```

Because the listing carries timestamps, you can check how stale the newest backup is and
take a fresh one before doing something destructive:

```bash
NEWEST=$(curl -s http://localhost:9481/api/backups \
  -H "Authorization: Bearer $TOKEN" | jq -r '.backups[0].created_at // empty')

if [ -z "$NEWEST" ] || [ "$(date -d "$NEWEST" +%s)" -lt "$(date -d '24 hours ago' +%s)" ]; then
  curl -s -X POST http://localhost:9481/api/backups -H "Authorization: Bearer $TOKEN"
fi
```

`GET /api/backups/{id}/download` retrieves an archive and `DELETE /api/backups/{id}`
removes one. There is no restore endpoint, by design. See the
[API reference](/api#backups-admin-only) for the full list.

## Taking a snapshot by hand

If you'd rather script it outside Grimoire, use SQLite's backup command:

```bash
sqlite3 data/grimoire.db ".backup '/path/to/grimoire-$(date +%F).db'"
```

::: danger Do not use `cp`
Grimoire runs SQLite in WAL mode, where recent commits may still live in a separate `-wal`
file. A plain `cp` of `grimoire.db` can capture a torn state: a database missing its most
recent writes, or inconsistent between them. `.backup` reads under a lock and folds the WAL
in, producing a single self-consistent file. It is safe on a running server; `cp` is not.
:::

Remember that a database-only snapshot leaves out `campaign_uploads/`, `system_covers/`,
and `audio_covers/`, which Grimoire's own backups include.
