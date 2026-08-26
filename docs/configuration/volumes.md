# Volumes

Grimoire needs two volume mounts:

```yaml
volumes:
  # Your library
  - /path/to/your/library:/app/library

  # Persistent data (database, thumbnails, page cache)
  - /path/to/grimoire/data:/app/data
```

## Library volume

Mount your TTRPG library folder here. The path inside the container defaults to
`/app/library`. If you change it, also set `LIBRARY_PATH` to match:

```yaml
environment:
  - LIBRARY_PATH=/my/custom/path
volumes:
  - /host/library:/my/custom/path
```

## Read-only or writable?

Grimoire only writes to your library when you ask it to. Browsing, searching,
reading, and metadata editing never touch the files, and there is no background
job that rewrites your library. The two features that do write are opt-in and
admin-only:

| Feature | Needs | What it writes |
|---|---|---|
| [File management](/deployment/file-management) (upload, move, rename, delete, new folders) | Writable mount | The files and folders you act on |
| [Metadata sidecars](/guide/opf-metadata#sidecar-export) (export as `.opf` / `.nfo` / `.json`) | Writable mount | Sidecar files next to your content |
| Everything else (browse, search, read, tags, favorites, campaigns, metadata edits) | Either | Nothing in the library folder |

### Writable (default)

Leave the `:ro` suffix off and Grimoire can manage files in place:

```yaml
volumes:
  - /path/to/your/library:/app/library
```

This is the recommended setup for most people. It is what makes **Settings →
Maintenance → Open file manager** and the move/rename/delete actions on a book's
⋮ menu work, and it means uploading a new PDF is a drag-and-drop into the
browser rather than a trip to the shell.

Writes stay confined to the library root, and the destructive ones are guarded:
deleting a folder that still holds files makes you type its name first.

### Read-only (opt-in hardening)

Add `:ro` if you would rather the container could not modify the library at all:

```yaml
volumes:
  - /path/to/your/library:/app/library:ro
```

Everything except the two write features above works exactly the same. Nothing
errors out or half-fails:

- The file manager opens and browses normally, and tells you the library is
  read-only when you try to change something.
- File actions on a book's ⋮ menu are not shown at all.
- Changing a book's **category** saves the category without moving the file.
- Sidecar export reports the read-only mount and skips the write; your metadata
  edits are still saved in Grimoire's database.

With a read-only mount you add and organize files with whatever tool you like
from outside the container: your OS file manager, `scp`, a network share, or
[Calibre](/deployment/file-management#calibre). Trigger a **Rescan** afterwards
so Grimoire picks the changes up.

### Switching later

The mount is not a one-way decision and nothing in Grimoire's database depends
on it. Edit the volume line, `docker compose up -d`, and the write features
appear or disappear accordingly.

## Data volume

Grimoire stores all persistent data here:

- `grimoire.db`: SQLite database (users, metadata, bookmarks, campaigns)
- `grimoire.fts.db`: full-text search index
- `thumbnails/`: generated book cover thumbnails
- `page_cache/`: rendered PDF page images (when Valkey is not used)

**Back this directory up** to preserve your metadata, user accounts, and bookmarks.

The path inside the container defaults to `/app/data`. If you change it, also set `DATA_PATH`:

```yaml
environment:
  - DATA_PATH=/my/custom/data
volumes:
  - /host/data:/my/custom/data
```
