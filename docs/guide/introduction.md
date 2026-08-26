# Introduction

Grimoire is a self-hosted web application for managing your tabletop RPG PDF collection. It runs as a Docker container, mounts your existing library folder, and gives you a clean browser-based UI to browse, search, and read your books from any device on your network.

## What it does

- **Organizes** your PDFs by game system and category, derived automatically from your folder structure, with no manual tagging required to get started
- **Indexes** every page of every PDF for full-text search using SQLite FTS5, including OCR of image-only scanned PDFs
- **Renders** PDF pages server-side as WebP images so the reader is fast even on mobile
- **Tracks** maps, tokens, audio, and archive files alongside books in the same library, with a persistent global audio player
- **Supports** campaigns (with a notes wiki, character sheets, and guest invites), bookmarks, favorites, OPDS feeds, and OpenID Connect authentication

## What it writes

Grimoire reads from your library folder and only writes to it when you ask it
to. Browsing, searching, reading, and metadata editing never touch your files.
Two admin-only features do write: [file management](/deployment/file-management)
(upload, move, rename, delete) and
[sidecar export](/guide/opf-metadata#writing-metadata-back-out). Both need the
library mounted writable, which is the default; mount it `:ro` and the rest of
Grimoire works unchanged. See [Volumes](/configuration/volumes#read-only-or-writable).

If you add or reorganize files from outside Grimoire, trigger a **Rescan** in the
UI to pick the changes up.

## Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (async Python 3.12) |
| Database | SQLite with FTS5 full-text search |
| PDF rendering | PyMuPDF → WebP |
| Frontend | React 18, served by the FastAPI backend |
| Optional cache | Valkey / Redis (page image cache) |
| Auth | JWT + optional OpenID Connect |

## License

GNU General Public License v3.0. Source on [GitHub](https://github.com/hunter-read/grimoire).
