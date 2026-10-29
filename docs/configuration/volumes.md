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
Library → Library tools → Open file manager** and the move/rename/delete actions on a book's
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

## File ownership

Grimoire does not run as root. The container starts as root just long enough to get the
data volume ready, then switches to an ordinary user before Grimoire starts, so a flaw in a
PDF parser or a third-party add-on would not have root access to your files. You don't need
to set anything.

**Which user.** By default, whoever owns your library folder, so files Grimoire creates
there (uploads, exported sidecars) belong to you like the rest of your library. If the
library belongs to root, it uses the owner of the data folder, then the built-in `grimoire`
user (`1000:1000`).

**The data volume** is handed to that user on every start: anything in it with another
owner is changed. So is a [`BACKUP_DIR`](/configuration/backups#storage-location) set in the
container's environment. After upgrading from a version that ran as root, the first start does
this for the old files, which can take a moment with a large thumbnail cache. **Your library
is never changed.** A backup folder chosen in Settings rather than with `BACKUP_DIR` is left as
it is; if that user can't write to it, the log says so, and you can give it access on the host
or set `BACKUP_DIR` instead.

**Nothing breaks on upgrade.** If that user couldn't read your library, couldn't write to
library folders root could write to (a library mounted without `:ro`), or couldn't write to
the data volume (say, a network share that refuses to change owners), Grimoire stays root and
the log says why and how to fix it. Write access is checked on the library folder and up to
500 folders inside it, top levels first, since renaming or moving a book needs write access
to its folder. Symlinks, `lost+found`, and folders that user already owns don't count towards
that check. The data volume is tested with a real write, because some NAS filesystems refuse
writes a plain permission check allows; the library is only ever checked, never written to.

If a folder is still out of reach, renaming, moving or uploading there says Grimoire doesn't
have permission to write to it and names the user it runs as, while a library mounted `:ro`
says it is read-only. See the
[FAQ](/faq#since-updating-renaming-or-moving-a-file-says-the-library-is-read-only) for the
fixes.

### Choosing the user: `PUID` / `PGID`

The convention Unraid and LinuxServer images use. When set, Grimoire always runs as exactly
this user, so check the log after a change. `PUID=0` keeps it running as root.

```yaml
environment:
  - PUID=99    # Unraid's nobody
  - PGID=100   # Unraid's users
```

### Starting as a fixed user: `--user`

`user: "1000:1000"` in compose, `docker run --user`, or `runAsUser` in Kubernetes skips the
switch entirely. Nothing can change owners in this mode, so the data volume must already be
writable by that user. On an install that used to run as root, fix it once on the host:

```bash
sudo chown -R 1000:1000 /path/to/grimoire/data
```

In Kubernetes, set `runAsUser: 1000`, `runAsGroup: 1000` and `fsGroup: 1000` in the pod's
`securityContext`; that also satisfies `runAsNonRoot: true`.

### Rootless Podman and Docker

Under rootless Podman (Quadlets included) or rootless Docker, root inside the container is
already your own unprivileged account on the host, so Grimoire stays as it and the files it
creates belong to you, as before. `PUID`/`PGID` still switch to that user inside the
container, and Podman's `--userns=keep-id` works like `--user`.
