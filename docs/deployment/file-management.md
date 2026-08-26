# File Management

Grimoire manages your library files itself. Admins can upload, move, rename, and
delete content from inside the app, and the metadata attached to a file follows
it wherever it goes.

This needs the library mounted **writable** — that is the default, and it is
simply the absence of a `:ro` suffix on the volume:

```yaml
volumes:
  - /path/to/your/library:/app/library
  - /path/to/grimoire/data:/app/data
```

See [Volumes](/configuration/volumes#read-only-or-writable) for what each mount
allows and what changes if you prefer a read-only library.

## The file manager

Open it at **Settings → Maintenance → Open file manager**. It is a folder tree
built for bulk reorganization rather than a plain file listing.

- **Expand folders in place** so a file and its destination are visible at once.
- **Move** files and folders by dragging them onto any folder. Collapsed folders
  spring open when you hold a drag over them, and the list auto-scrolls near its
  edges. Ctrl/Cmd-click to select several at once.
- **Pin a second pane** to the right, left, above, or below when the two ends of
  a move are far apart. Either pane can be closed to return to one.
- **Rename** a file or folder on disk. The extension is held aside and reattached
  on save, since Grimoire infers a file's type from its suffix. This is distinct
  from editing an item's display title, which only changes what Grimoire shows.
- **Delete** a file or folder. This cannot be undone — the file is removed from
  disk, and its tags, favorites, bookmarks, reading progress, and campaign links
  go with it. A folder that still holds files makes you type its name first.
- **Upload files and folders** by dragging them in from your desktop, or via
  right-click → **Upload files… / Upload a folder…**. A progress panel names any
  failures and lets you retry them individually or all at once.
- **Preview an item** in place with right-click → **Preview…**: a book's rendered
  pages, a map or token image, or an audio player.
- **Edit an item's metadata** with the same editor the library views use.
- **Create folders**, including system, category, and container folders. Choosing
  a container type writes the right marker file for you. **Create standard
  category folders** sets up a whole system in one step — Core, Supplements,
  Adventures, Character Sheets, Maps, Handouts, Homebrew, and Starter Sets, named
  so the scanner classifies them correctly.
- **Mark a folder NSFW or SFW**, or change its container type, without recreating
  it.
- **Rescan** from here: the **Rescan** button re-indexes the whole library, and
  right-click → **Rescan this…** re-indexes just that folder or file. *Refresh*
  only re-reads the folder listing.

The file manager is admin-only, and every destination is confined to the library
root.

## File actions from anywhere

Move, rename, and delete also sit on the **⋮ menu of a book itself**, in the
library views and in the reader, so a single file does not need a trip to the
file manager. They appear at the bottom of the menu behind a divider and behave
exactly as they do in the file manager. Moving from here opens a folder picker
rather than asking you to drag.

These actions are shown only to **admins on a writable library**.

## Moves keep your metadata

Grimoire relinks the existing record rather than treating a moved file as new, so
tags, favorites, reading progress, bookmarks, campaign links, and the search
index all follow the file. A book moved into a different system or category
folder is re-filed automatically, and its system, edition, and category are
re-derived from where it now lives.

Changing a book's **category** in the metadata editor moves the file into the
matching folder, creating it if needed. An existing folder wins over a new one:
if your core books live in a folder called *Rulebooks*, a book re-categorised as
*core* joins them rather than a second *Core* folder appearing beside it.

Changes made in the file manager apply immediately — no rescan needed.

## Managing files outside Grimoire

Nothing stops you adding files by other means: your OS file manager, `scp`, a
network share, or a companion container. This is also how you work when the
library is mounted read-only.

After adding files with an external tool, trigger a **Rescan** in Grimoire
(sidebar or **Settings → Maintenance**) to pick up the new content.

## Calibre

[Calibre](https://calibre-ebook.com/) remains a genuine companion for ebook
management: format conversion, bulk metadata editing across a large collection,
and its own reader ecosystem. It writes `.opf` sidecar files that Grimoire reads
on the next scan to populate titles, authors, publishers, and tags.

See [OPF Metadata](/guide/opf-metadata) for the fields Grimoire reads. If you
want the traffic to go the other way — Grimoire's metadata written out as
sidecars for Calibre and other tools — see
[sidecar export](/guide/opf-metadata#writing-metadata-back-out).

### Docker Compose (full desktop via noVNC)

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
      - WORKERS=2
      - SECRET_KEY=change-me
    volumes:
      - /path/to/your/library:/library
      - /path/to/grimoire/data:/data
    networks:
      - grimoire

  calibre:
    image: lscr.io/linuxserver/calibre:latest
    container_name: grimoire-calibre
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    ports:
      - "8080:8080"   # Calibre desktop (noVNC)
      - "8081:8081"   # Calibre Content Server
    volumes:
      - /path/to/your/library:/library
      - calibre_config:/config
    networks:
      - grimoire

volumes:
  calibre_config:

networks:
  grimoire:
```

Access the Calibre desktop at `http://localhost:8080`. On first run, point the
Calibre library at `/library/books`.

::: tip Use the Compose Generator
The **[Compose Generator](/compose-generator)** can build this file with your
paths pre-filled.
:::

### Calibre's folder layout

Calibre writes a subfolder per book with `metadata.opf` and `cover.jpg`:

```
books/
└── Dungeons & Dragons/
    └── core/
        ├── Players Handbook/
        │   ├── players_handbook.pdf
        │   ├── metadata.opf        ← read by Grimoire
        │   └── cover.jpg
        └── Dungeon Masters Guide/
            ├── dungeon_masters_guide.pdf
            ├── metadata.opf
            └── cover.jpg
```

## Calibre-Web

[Calibre-Web](https://github.com/janeczku/calibre-web) is a lightweight browser
UI for an existing Calibre library. Use it instead of the full Calibre desktop if
you don't need the desktop application.

```yaml
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: grimoire-calibre-web
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
      - DOCKER_MODS=linuxserver/mods:universal-calibre  # enables metadata writing
    ports:
      - "8083:8083"
    volumes:
      - /path/to/your/library:/library
      - calibre_web_config:/config
```

Point Calibre-Web at `/library/books` as its library path on first setup. Default
credentials: `admin` / `admin123` — change them immediately.
