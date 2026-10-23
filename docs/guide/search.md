# Search

Grimoire indexes every page of every PDF using SQLite FTS5 for fast full-text search across your entire library.

## What gets indexed

- **Books**: every page, indexed at scan time. The search index is stored in the data volume.
- **Maps**: matched by filename, folder path, and tag
- **Tokens**: matched by filename, folder path, and tag
- **Audio**: matched by filename, folder path, and tag

## Search behaviour

- Minimum query length: **2 characters**
- Results include a snippet showing the matching text in context
- Results can be scoped to a single book or game system
- Maps, tokens, and audio are only returned in global (unscoped) searches
- A book's [product code](/guide/library-structure#product-codes) is matched as
  well as its title, so searching `PZO9001` finds that book. To search codes
  only, use `code:PZO9001` (or `sku:`)

## Narrowing a gallery or system

The search box at the top of a gallery (Maps, Tokens, Audio, Models) filters what you are browsing. Plain text matches an item's name, its folder path, and its tags (including tags set on a folder above it); `tag:`, `title:` (or `filename:`), and on audio `artist:` and `album:` narrow to one field. It combines with the Filters menu (tags, favourites, recently added), and the results stay grouped by folder unless grouping is turned off.

The search box on a game system's page searches inside that system: books whose title, filename, or product code match are listed above the matching pages, honouring the shelf's filters. The book prefixes work there too - `author:`, `publisher:`, `year:` (`year:1985`, `year:>2000`, `year:1990-1999`), `tag:`, `code:`, `isbn:`, `language:`.

Both run on the server, so they cover the whole collection even when most of it has not been loaded.

## Image-only PDFs (OCR)

Scanned books with no embedded text layer show an **Image Only** badge and can't be searched from their text layer alone. The default image bundles the Tesseract OCR engine, so these are run through OCR in the background and their recognised text is added to the index, after which such books show an **OCR** badge. See the [OCR guide](/guide/ocr) for how it works, adding languages, and using the slim (no-OCR) image, or [Performance → OCR](/configuration/performance#ocr) for tuning.

## Indexing

On first startup, Grimoire scans and indexes every PDF page. For large collections this can take several minutes. The index is stored in the data volume and subsequent startups are fast.

Progress is shown in **Settings → Library → Scanning**. You can also cancel a running index job from there.

## API

See the [API reference](/api#search) for the search endpoint parameters and response format.
