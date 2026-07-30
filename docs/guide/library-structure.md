# Library Structure

Grimoire derives game systems, categories, and groups entirely from your folder structure, with no manual configuration required.

## Top-level layout

```
library/
├── books/    ← PDF collection (and archive files)
├── maps/     ← battle maps and scene images
├── tokens/   ← character tokens and portraits
└── audio/    ← ambient tracks, music, and sound effects
```

## Books

Each top-level folder under `books/` becomes a **game system**. Subfolders are auto-detected as categories.

```
books/
└── Dungeons and Dragons 5e/       ← game system
    ├── core/                      ← category: Core Rulebooks
    │   ├── Players Handbook.pdf
    │   └── monsters/              ← subfolder group "Monsters"
    │       └── Monster Manual.pdf
    ├── supplements/               ← category: Supplements
    ├── adventures/                ← category: Adventures
    │   └── Curse of Strahd/      ← subfolder group "Curse of Strahd"
    │       ├── Curse of Strahd.pdf
    │       └── Strahd DM Screen.pdf
    └── homebrew/                  ← category: Homebrew
```

### Category folder names

Folder name matching is **case-insensitive** and treats hyphens, underscores, and spaces as equivalent.

| Category | Recognized folder names |
|---|---|
| Core Rulebooks | `core`, `rulebooks`, `rules` |
| Starter Set | `starter-set`, `starter kit`, `beginner box`, `boxed set`, `essentials` |
| Supplements | `supplements`, `sourcebooks`, `expansions` |
| Adventures | `adventures`, `modules`, `campaigns` |
| Character Sheets | `character-sheets`, `character sheets`, `charsheets` |
| Handouts | `handouts`, `reference`, `screen` |
| Homebrew | `homebrew`, `custom`, `house-rules` |

::: tip
Files placed directly in a system folder (not in a subfolder) default to the **Core Rulebooks** category.

Any unrecognized subfolder name becomes its own category, slugified from the folder name, so `Bestiary` becomes the `bestiary` category.
:::

### Disabling category inference

If you organize your folders differently and don't want Grimoire deriving categories from folder names, you can turn the behavior off. When inference is disabled, books fall back to the neutral `uncategorized` category (any category you set in the web UI is still respected).

There are two ways to disable it, applied in this order of precedence:

1. **Globally** — turn it off for the whole library in **Settings → Maintenance → Folder Category Inference**, or pin it with the [`DISABLE_FOLDER_CATEGORY_INFERENCE`](/configuration/env-vars#library-scanning) environment variable. When the env var is set, the in-app toggle shows the effective value read-only.
2. **Per game system** — place an empty file named `.no-auto-category` at a system's folder root (e.g. `books/My System/.no-auto-category`) to disable inference for just that system while leaving the rest of the library on the default behavior.

The change takes effect on the next scan of the affected books.

### Subfolder groups

Any category folder can contain named subfolders to group related books. Grimoire detects these automatically and displays them as collapsible groups, with no configuration needed.

Books without a subfolder appear ungrouped at the top of their category, above any groups. Subfolder groups include a download button for the whole group.

### Archive files

Archive files placed anywhere under `books/` are shown alongside your books in their category, handy for bundling related files (a maps pack, a COMP/CON export, loose handouts) next to the book they belong to. Recognized extensions:

| Type | Extensions |
|---|---|
| Zip | `.zip`, `.cbz` |
| RAR | `.rar`, `.cbr` |
| 7-Zip | `.7z`, `.cb7` |
| Tar | `.tar`, `.cbt`, `.tar.gz`, `.tgz`, `.tar.bz2`, `.tbz2` |

Archives are treated as opaque downloads: Grimoire does not extract or read their contents, so clicking one downloads the file rather than opening the reader. They're included when you download a whole system, category, or subfolder as an archive. Comic-book archives (`.cbz`, `.cbr`, `.cb7`, `.cbt`) additionally get a cover thumbnail generated from the first image inside them.

### System-agnostic collections

Some books don't belong to a single game system. Create a folder with one of these names and Grimoire displays its contents in a separate **System-Agnostic** section:

| Folder name |
|---|
| `System Agnostic` |
| `Generic` |
| `Any` |

Subfolders directly under the agnostic root become custom category headings, and the folder name is used as-is.

### Explicit content

Append `(nsfw)` to a system folder name to mark all content in that system as explicit:

```
books/
└── Some Adult Game (nsfw)/
    └── core/
        └── rulebook.pdf
```

Users with explicit content disabled will not see this system or its books.

### Sort-order prefixes

Many file browsers sort folders alphabetically, so it's common to prepend
punctuation to a folder name to pull it to the top of the list. Grimoire ignores
a leading run of `!`, `$`, or `%` characters when deriving a system's name — only
the contiguous run at the very start is stripped, and everything from the first
other character onward is kept as-is:

```
books/
├── !!Dungeons & Dragons/   → shown as "Dungeons & Dragons"
├── !system-agnostic/       → still the System-Agnostic collection
└── $%Pathfinder 2e/        → shown as "Pathfinder 2e"
```

Only `!`, `$`, and `%` are recognized, and only as a leading prefix — internal
occurrences (e.g. `D&D $ Extras`) are left untouched. The prefix also stacks with
`(nsfw)`, so `!!Forbidden Lore (NSFW)` becomes the explicit system "Forbidden Lore".

## Maps

```
maps/
└── Creator Name/      ← shown as a group header in the map gallery
    └── map-file.png
```

## Tokens

```
tokens/
└── Category/          ← shown as a group header in the token browser
    └── token-file.png
```

## Audio

```
audio/
└── Category or Creator/   ← shown as a group header in the audio library
    ├── cover.jpg           ← optional folder artwork (cover.* or folder.*)
    └── track.mp3
```

Supported formats: `.mp3`, `.ogg`, `.opus`, `.flac`, `.wav`, `.m4a`, `.aac`. Duration and embedded title/artist/album tags are read on scan. See the [Audio Library](/guide/audio) guide for playback and the global player.

## Ignoring files

To keep files on disk but out of Grimoire, add a `.grimoireignore` file. It uses the same syntax as `.gitignore` or `.dockerignore`, so anything matched by a rule is skipped by the scanner and never shown in the UI. This is handy when a book ships extra print variants — black-and-white single-page versions, zine-sized layouts — that you want kept alongside the book but hidden from the library.

Place `.grimoireignore` at your **library root** to apply rules everywhere, or in any subfolder to add rules for just that subtree. Rules are cumulative and nested, exactly like git.

```
library/
├── .grimoireignore              ← applies to the whole library
└── books/
    └── Example TTRPG/
        ├── core/
        │   └── Players Handbook.pdf
        └── ignore/               ← ignored: this folder is skipped
            ├── Players Handbook BW Single Pages.pdf
            └── Players Handbook Zine-sized.pdf
```

A `.grimoireignore` with:

```
# Skip an entire folder
ignore/

# Skip print variants anywhere in the library
*BW Single Pages*.pdf
*Zine-sized*.pdf
```

Patterns support the full gitignore dialect, including `!` to re-include a previously excluded file and `**` for arbitrary-depth matching. Rules apply to every collection — `books/`, `maps/`, `tokens/`, and `audio/`.

Changes take effect on the next scan. If you add a rule that matches a file Grimoire already indexed, that item is marked missing and hidden on the next rescan; remove the rule and rescan to bring it back.

## Rescanning

After adding, removing, or reorganizing files, click **Rescan** in the sidebar or go to **Settings → Maintenance → Rescan Library** to pick up the changes. You can also configure a scheduled rescan there.

::: tip Scoped rescan
For large libraries you don't have to rescan everything. Every system, category, subfolder, and map/token/audio group has its own rescan button that re-scans just that folder.
:::

### Metadata refresh modes

OPF and `tags.json` metadata is only applied when an item is **first indexed**: ordinary rescans leave existing records alone so your web-UI edits aren't overwritten. To pick up a sidecar file you added or corrected after the initial scan, choose a mode in the rescan dialog (available on the global Rescan button and every per-folder rescan button):

| Mode | Behaviour |
|---|---|
| **Find new files** | Default: add new files, flag missing ones, leave existing records untouched. |
| **Update missing metadata** | Additionally fill **empty** book fields from sidecar files, without touching anything you've already set (non-destructive). |
| **Replace all metadata** | Overwrite fields with whatever the sidecar files provide (this discards UI edits the sidecar covers). |
