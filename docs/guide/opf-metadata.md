# OPF Metadata

Grimoire reads [OPF](https://idpf.org/epub/20/spec/OPF_2.0.1_draft.htm) sidecar files to populate book metadata automatically on first scan. OPF is the format used by [Calibre](https://calibre-ebook.com/) and many other library managers.

## Supported fields

| OPF element | Book field |
|---|---|
| `dc:title` | Title |
| `dc:creator` (role=aut) | Authors |
| `dc:publisher` | Publisher |
| `dc:date` | Year (4-digit year extracted) |
| `dc:description` | Description (HTML tags stripped) |
| `dc:subject` | Tags (lowercased) |
| `guide/reference[@type='cover']` | Cover image (excluded from book list) |

`dc:contributor` and `dc:identifier` are intentionally ignored.

## File discovery

The scanner checks two locations per book, in priority order:

1. **`<bookname>.opf`**: a sidecar file with the same stem as the PDF, in the same directory
2. **`metadata.opf`**: a file named `metadata.opf` in the same directory (Calibre's per-book-folder format)

## Calibre layout

A typical Calibre export is fully supported:

```
books/
└── Dungeons & Dragons/
    └── core/
        ├── Players Handbook/
        │   ├── players_handbook.pdf
        │   ├── metadata.opf       ← read by Grimoire
        │   └── cover.jpg          ← used as book cover
        └── Dungeon Masters Guide/
            ├── dungeon_masters_guide.pdf
            ├── metadata.opf
            └── cover.jpg
```

## Re-applying metadata

OPF metadata is applied only when a book is **first indexed**. Edits made via the web UI are not overwritten on subsequent rescans.

To re-apply updated OPF metadata to an already-indexed book, delete the book record in Grimoire (**Settings → Maintenance → Cleanup Missing**) and trigger a rescan.

## Writing metadata back out

Grimoire can also write its metadata *out* as sidecar files next to your content,
so the library folder describes itself. Copy the library elsewhere, or rebuild the
container with a fresh `DATA_PATH`, and the metadata travels with the files
instead of living only in Grimoire's database. Calibre, Jellyfin, Kodi, and a
plain file manager can all read it.

Turn it on in **Settings → Maintenance → Metadata Sidecars** and tick the formats
you want (`.opf`, `.nfo`, `.json`). It is off by default.

Three things trigger a write:

- **Export Metadata To Library** backfills the library, writing only the sidecars
  that are **missing** — safe to re-run, and it will not overwrite one you edited
  by hand.
- Any new book a scan picks up gets its sidecars written automatically.
- Editing a book's metadata in the UI **updates the sidecars it already has**,
  creating none.

Grimoire only overwrites files it wrote: every exported sidecar carries a marker,
and a `.opf` you maintain in Calibre is left alone and reported as skipped unless
you explicitly allow overwriting.

Sidecars are hidden in the [file manager](/deployment/file-management) — they
describe your content rather than being content — but they still travel with the
file they belong to when it is moved or renamed.

::: warning Needs a writable library
Sidecar export writes into your library folder, so it needs the library mounted
writable. On a read-only mount, export reports it clearly and your metadata edits
keep working — only the sidecar write is skipped. See
[Volumes](/configuration/volumes#read-only-or-writable).
:::

## Using Calibre to manage metadata

See the [File Management](/deployment/file-management) guide for running Calibre alongside Grimoire in Docker so it can write `metadata.opf` files that Grimoire reads automatically.
