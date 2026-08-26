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

1. **Globally** — turn it off for the whole library in **Settings → Application → Folder Category Inference**, or pin it with the [`DISABLE_FOLDER_CATEGORY_INFERENCE`](/configuration/env-vars#library-scanning) environment variable. When the env var is set, the in-app toggle shows the effective value read-only.
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

Some books don't belong to a single game system. Create a folder with one of these names and Grimoire displays its contents in a separate **Special Collections** section:

| Folder name |
|---|
| `System Agnostic` |
| `Generic` |
| `Any` |

Subfolders directly under the agnostic root become custom category headings, and the folder name is used as-is.

### One-page and micro RPG collections

Some "systems" are really a bucket of many tiny games. Folders with one of these names are treated as a **sub-library**: a single tidy entry in the Special Collections strip whose contents are each their own game system.

| Folder name |
|---|
| `One-Page RPGs` |
| `Single-Page RPGs` |
| `One-Shot RPGs` |
| `Micro RPGs` |

Both subfolders **and** loose files at the root become systems:

```
books/
└── One-Page RPGs/
    ├── honey-heist.pdf          → system "Honey Heist" (1 book)
    ├── lasers-and-feelings.pdf  → system "Lasers And Feelings" (1 book)
    └── cbr+pnk/                 → system "Cbr+pnk" (2 books)
        ├── core/
        │   └── core-rules.pdf
        └── character-sheets/
            └── character.pdf
```

A single-file game becomes a system holding that one book; a folder-backed game keeps its internal category structure. Either way, each game gets full system metadata, tags, and system-level filtering — so a pile of one-shot PbtA games can all be found by system without cluttering the main library grid.

::: tip Systems count
Games nested inside a collection count toward your library's game-system total. If you already use a `One-Page RPGs` folder, that number rises after the first rescan as each game inside becomes its own system.
:::

### Parent systems and editions

A folder can also be a shelf for the editions of one game. Drop a `.parent-system-container` marker file at its root (or add a `(parent-system)` suffix to the folder name), and its immediate subfolders become systems:

```
books/
└── Dungeons & Dragons/
    ├── .parent-system-container
    ├── 3e/
    │   └── core/
    │       └── Players Handbook.pdf
    └── 5e/
        └── core/
            └── Players Handbook.pdf
```

This yields the systems "Dungeons & Dragons 3e" and "Dungeons & Dragons 5e", each with **Parent System** set to "Dungeons & Dragons" and **Edition** set to the folder name — both available as library filters. Category folders work normally inside each edition.

### System families

A family groups related but *distinct* systems that share a lineage — unlike a parent system, whose children are editions of one game. Use a `.system-family-container` marker (or a `(system-family)` folder-name suffix):

```
books/
└── d20 System/
    ├── .system-family-container
    ├── Pathfinder/
    │   ├── .parent-system-container
    │   ├── 1e/
    │   └── 2e/
    ├── Mutants & Masterminds/
    └── d20 Modern/
```

Each child indexes as an independent system, and the container's name populates its **System Family** field, so the folder structure lines up with the family filter. Family children keep their own names — no `{Parent} {Child}` prefixing — and get no **Edition** or **Parent System**, because they are not variants of anything.

Containers nest: as shown above, a family can hold a multi-edition system. The inner `.parent-system-container` resolves its editions exactly as it would at the top level, and inherits the family name itself.

### Publisher containers

A `.publisher-container` marker (or a `(publisher)` suffix) groups the systems one company puts out:

```
books/
└── Paizo/
    ├── .publisher-container
    ├── Pathfinder 2e/
    └── Starfinder/
```

Each child is an independent system with the container's name recorded as its **Publisher**.

### Generic containers

If your shelf does not fit any of the named kinds, a bare `.container` marker (or a `(container)` suffix) says only "the folders in here are systems" and claims nothing about how they relate:

```
books/
└── Kickstarter Hauls/
    ├── .container
    ├── Mörk Borg/
    └── Mothership/
```

Unlike the named kinds it propagates no metadata — no family, publisher, edition, or parent system. It groups, and nothing more.

Family and publisher containers only fill in metadata a system does not already have. A family or publisher set by an OPF sidecar, an add-on, or your own edit is never overwritten by a rescan.

### Declaring a container

Any of these declare a folder to be a container of systems, and they combine with `(nsfw)` and sort-order prefixes:

| Method | Example | Kind |
|---|---|---|
| Marker file | `books/D&D/.parent-system-container` | Parent system |
| Marker file | `books/Itch Bundle/.one-page-container` | One-page collection |
| Marker file | `books/d20 System/.system-family-container` | System family |
| Marker file | `books/Paizo/.publisher-container` | Publisher |
| Marker file | `books/Kickstarter Hauls/.container` | Generic |
| Folder-name suffix | `books/Cyberpunk (parent-system)/` | Parent system |
| Folder-name suffix | `books/Jam Games (one-page)/` | One-page collection |
| Folder-name suffix | `books/Powered by the Apocalypse (system-family)/` | System family |
| Folder-name suffix | `books/Chaosium (publisher)/` | Publisher |
| Folder-name suffix | `books/My Shelf (container)/` | Generic |
| Recognized name | `books/One-Page RPGs/` | One-page collection |

If a folder carries more than one declaration, the most specific kind wins, in this order: **parent system → one-page → system family → publisher → generic**. Every recognized suffix is stripped from the stored name either way, so a stray `(publisher)` never reaches the UI.

Child systems get a default name — `{Parent} {Edition}` for parent systems, the prettified file or folder name for one-page games, and their own folder name for family, publisher, and generic children. Only a parent-system container sets its children's **Parent System**: the other kinds hold independent games, not variants of the shelf. Rename one in the UI and it sticks: rescans never overwrite a system you have renamed, so "Dungeons & Dragons 2e" can become "Advanced Dungeons & Dragons".

### Flattening containers in the library

Containers organize the grid; they do not lock systems away. The **Group collections** switch beside the "Your Collection" heading — shown only once you have a container — flattens them: container cards drop out and their child systems take their place, giving a plain list of every real system with the usual sorting and filters applied. Turn it back on to return to the drill-down view. The choice is remembered across sessions.

One-page collections are the deliberate exception and stay grouped either way. Keeping a pile of tiny one-book games out of the main grid is the entire reason that collection exists, so flattening leaves its chip in the Special Collections strip and its games reachable by drilling in.

### Reorganizing an existing library

Moving a flat `books/Dungeons & Dragons 5e/` into `books/Dungeons & Dragons/5e/` produces a child whose generated name matches the system you already have. Grimoire adopts that existing system instead of creating a duplicate — its books, metadata, tags, and cover follow it into the container, and the old entry disappears from the top level.

### System cover art

A system's cover comes from the first of these that exists:

1. a `cover.*` or `folder.*` image at the system's folder root,
2. an image uploaded from the system's page (**Cover image**, GM/admin only),
3. a thumbnail from one of the system's books.

A `cover.*` / `folder.*` image at a system's folder root is artwork only — it is not also indexed as a book. The same name deeper in the tree (in a category folder, say) is an ordinary image book.

Container folders hold no books of their own, so options 1 and 2 are the only ones available to them:

```
books/
└── Dungeons & Dragons/
    ├── .parent-system-container
    ├── cover.jpg           ← folder artwork
    └── 5e/
```

### Explicit content

Append `(nsfw)` to a system folder name to mark all content in that system as explicit:

```
books/
└── Some Adult Game (nsfw)/
    └── core/
        └── rulebook.pdf
```

Users with explicit content disabled will not see this system or its books.

An empty `.nsfw` file at the system root does the same thing, for when parenthesised folder names are awkward for your filesystem or sync tool:

```
books/
└── Some Adult Game/
    ├── .nsfw
    └── core/
        └── rulebook.pdf
```

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
