# Filename Rules

If you name your files carefully, Grimoire can use that. Filename rules teach it what your naming scheme means:

- **Folder naming patterns** describe how the books in a folder are named. Grimoire reads each new book's title (and publisher, version, year, ...) out of its filename, and can rename existing files to match - or move them from an old scheme onto a new one.
- **Variant filename rules** are your library's own markers for variants - `GL_` for a gridless map, `-pf` for a printer-friendly PDF - so duplicate detection pre-fills them the way it already does for `nogrid` or `printer friendly`.

Both live under **Settings → Library → Files & folders** and are admin only. Files are never renamed unless you ask.

## Folder naming patterns

Take a file named:

```text
D&D5e - [Loot Tavern] - adventure L2,7,12 - Crystallica (v0.99, 2026).pdf
```

With this pattern on its folder:

```text
{system} - [{publisher}] - {*} - {title}< (v{version}, {year})>
```

the book shows as **Crystallica**, published by **Loot Tavern**, version **0.99**, from **2026**. The filename itself is unchanged.

### Pattern syntax

Plain text has to appear in the filename as written (case and the width of spaces don't matter). Each `{field}` matches the part of the name in its place.

| Field | When reading a filename | When renaming |
|---|---|---|
| `{title}` | The book's title - required in every pattern | ✓ |
| `{publisher}`, `{version}`, `{isbn}`, `{product_code}` | Fill those fields | ✓ |
| `{authors}` | Authors, split on `,`, `&` and `and` | ✓ |
| `{year}` | A four-digit year | ✓ |
| `{system}`, `{category}`, `{language}`, `{date}` | Must match, but aren't kept | ✓ |
| `{*}` | Anything, thrown away | ✗ |

- **`{*}`** lets a pattern pick out the title without describing the rest of the name. A pattern with `{*}` can read titles but can't rename files.
- **`< ... >`** marks an optional part: `{title}< (v{version})>` matches `Crystallica.pdf` and `Crystallica (v2).pdf`. When renaming, an optional part is dropped if a field inside it is empty.

A pattern holds at most 20 fields, `{*}` included.

### Titles from filenames

New books under a folder with a pattern take their title from the filename when it fits the pattern, along with any publisher, authors, year, version, ISBN and product code the pattern names. A sidecar [`.opf`](/guide/opf-metadata) still wins.

For books already in the library, open the pattern, check the preview and choose **Update titles**. Titles are overwritten; the other fields are only filled where a book has none, so anything you typed in by hand stays.

This works on a [read-only library](/configuration/volumes#read-only-or-writable) - it never touches the files.

### Renaming files to match

A pattern without `{*}` also works in reverse: Grimoire writes each book's filename from its metadata. The preview shows every new name and flags the ones it won't rename - a required field is empty, the name is over 255 bytes, or another file already has it - and warns when a full path would pass Windows' 260-character limit.

Tick the books you want and choose **Rename files**. Each book keeps its tags, favourites, reading progress, campaign links and sidecars, exactly as with a rename in the [file manager](/deployment/file-management). Renaming needs a writable library, and only ever happens when you ask for it.

### Which pattern applies

A pattern covers its folder and every folder beneath it; when two apply, the one nearest the book wins. Renaming or moving a folder in the file manager takes its pattern along, and deleting the folder deletes it.

## Variant filename rules

When duplicate detection pairs two copies of something, it guesses what kind of variant each is from the filenames. Add rules for your own markers:

| Match | Text | Kind | Label |
|---|---|---|---|
| Starts with | `GL_` | Gridless | |
| Whole word | `night` | | Night |
| Ends with | `-pf` | Printer friendly | |

- **Whole word** matches complete words of the name, so `grid` doesn't match `Gridiron Tavern`. **Starts with** / **Ends with** test the name without its extension. Case is ignored.
- A rule can set a kind, a label, or both, and can be limited to one collection.
- A rule only counts when one copy matches and the other doesn't.
- Rules run top to bottom, before the built-in markers, so your convention wins.

Suggestions are only ever a starting point - linking variants is always your call. New rules apply to the next duplicate scan and to any comparison you open.
