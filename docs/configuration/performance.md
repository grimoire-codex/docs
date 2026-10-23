# Performance

## Indexing

On first startup Grimoire scans the library and indexes every PDF page for full-text search. This can take several minutes for large collections. The index is stored in the data volume and subsequent startups are fast.

New files are picked up by themselves while the library is [watched for changes](/guide/library-structure#watching-for-changes), the default. Use the **Rescan** button in the sidebar to pick them up by hand, or configure a scheduled rescan in **Settings → Library → Scanning** - on a slow server you can turn watching off and scan at a quiet hour instead. For large libraries you can also rescan a single corner: every system, category, subfolder, and map/token/audio group has its own rescan button that re-scans just that folder.

### Very large images

Very large images, usually high-resolution scanned battlemaps, can be indexed without a thumbnail. The image library Grimoire uses (Pillow) refuses to open anything over about 179 megapixels, treating it as a possible decompression bomb, which is a limit sized for untrusted web uploads. A 60x60in map scanned at 300 DPI is 324 megapixels. When this happens the container log says so:

```
Thumbnail skipped for /app/library/…/map.jpg: Image size (324000000 pixels) exceeds limit of 178956970 pixels…
This is the decompression-bomb guard, not a damaged file.
Raise MAX_IMAGE_PIXELS (currently Pillow's default) to index images this large.
```

To thumbnail them, set `MAX_IMAGE_PIXELS` and restart. Pillow warns above this many pixels and refuses above twice it, so `MAX_IMAGE_PIXELS=400000000` covers images up to 800 megapixels without log warnings below 400. Then rescan the folder the maps are in to backfill their thumbnails.

Raising it costs less than it sounds for JPEGs, which are decoded at a reduced scale for the thumbnail: a 324-megapixel map takes about 2 seconds and 42 MB. PNGs get no such shortcut and are decoded in full, and uploaded files go through the same path, so only raise it if you trust where your images come from. Leave it unset and nothing changes.

## OCR

Some PDFs contain only scanned page images with no embedded text layer (common with older, scanned books). These can't be full-text searched from their text layer alone and show an **Image Only** badge.

The default Grimoire image bundles the [Tesseract](https://github.com/tesseract-ocr/tesseract) OCR engine (English), so scanned image-only PDFs are run through OCR and their recognised text is added to the search index. Books indexed this way show an **OCR** badge. No extra container or service is required.

OCR runs quietly in the background, so it never holds up the rest of your library. A scan indexes all your regular (text-based) books, maps, tokens, and audio first — those are searchable right away — and then works through the scanned books afterward. Even if you add 100+ scanned books at once, the rest of your library stays available while they're being processed.

Scanned books are processed **page by page**, and progress is saved as it goes. If the server restarts (or you stop and start a scan), OCR simply picks up where it left off instead of starting the book over, so even a very large scanned book will finish however long it takes. You can watch the queue drain under an **OCR** phase in the scan status (**Settings → Library → Scanning**).

### Options

- **Disable OCR**: set `OCR_ENABLED=false` (or `OCR_CONCURRENCY=0`). Image-only PDFs are then left unindexed, the same as on the slim image.
- **Slim image**: the `-slim` tags (e.g. `hunterreadca/grimoire:slim`, `:1.5.0-slim`) omit Tesseract for a smaller image. OCR is automatically disabled there and Grimoire degrades gracefully.
- **Upgrading to OCR**: when OCR becomes available (upgrading from a slim image, or enabling it), previously image-only books are automatically queued for OCR on the next startup scan.
- **Additional languages**: set `OCR_LANGUAGES` to a `+`-joined list of Tesseract language codes (e.g. `eng+deu+fra`). The extra languages' `.traineddata` files must be present in the image's tessdata directory. Mount a directory of `.traineddata` files over it (or point `TESSDATA_PREFIX` at a mounted directory) to add languages without rebuilding.

### Speeding it up

If you have a large collection of scanned books and want the queue to finish faster, two settings help:

- **`OCR_CONCURRENCY`** (default `1`) — how many scanned books to work on at once. On a machine with several CPU cores and memory to spare, raising this (e.g. `2`–`8`) processes books in parallel. Each parallel worker typically adds on the order of 50–250 MB of RAM (it varies with the scanned page's dimensions and quality), so leave headroom for that per unit, and don't set it above your CPU core count (virtual/hyper-threaded cores count) or the workers just contend for the same processors. On a small device like a Raspberry Pi, leave it at `1`. Setting it to `0` turns OCR off — handy if OCR keeps failing or exhausting memory and you just want it to stop, without switching to the slim image.
- **`OCR_DPI`** (default `150`) — how sharp scanned pages are rendered before reading them. Lowering it (e.g. `120`) makes OCR faster and lighter on memory; raising it (e.g. `200`–`300`) can improve results on faint or low-quality scans at the cost of speed.

A rough guide: a small always-on device (like a Pi) is happiest at the defaults; a typical NAS can handle `OCR_CONCURRENCY=2`; a powerful desktop or server can go higher. It's safe to start low and raise it later — the queue simply continues faster.

### Giving slow pages more time

Each page gets a time budget — `OCR_PAGE_TIMEOUT`, default `120` seconds. A page that runs over it is abandoned and the book moves on, so one pathological page can't stall a book forever.

The catch is that how long a page takes depends on how **dense and noisy** the scan is far more than on its dimensions. On a low-power CPU an ordinary page might read in 13 seconds while a cramped, speckled one from the same book needs four minutes. Those slow pages are skipped, and their text never becomes searchable.

If that's happening, raise the budget (e.g. `OCR_PAGE_TIMEOUT=600`) and re-read the affected books. Raising it costs nothing when pages finish quickly — it's a ceiling, not a delay. Set it to `0` for no limit at all, if you'd rather wait indefinitely than lose text.

### When a book is only partly read

A scanned book that OCR'd cleanly is badged **OCR**. If some of its pages were skipped — they ran over `OCR_PAGE_TIMEOUT`, or reading them failed — it's badged **OCR 14/206** instead: amber, showing how many pages were actually read. Those pages aren't in the search index, so searching that book will quietly miss their text.

To recover them: raise `OCR_PAGE_TIMEOUT` (see above), restart, then re-read the book from its actions menu (**⋮** → **Re-OCR…**). Skipped pages are also logged as warnings when a book finishes, so `docker logs grimoire` will name them.

## Page rendering

PDFs are rendered page-by-page server-side as WebP images. Rendered pages are cached aggressively with `Cache-Control: max-age=31536000, immutable`, so repeat visits load instantly.

Rendering happens on first access for each page. For large books, the first open may take a moment while the first few pages render.

## Large libraries

The galleries (Maps, Tokens, Audio, Models), a game system's shelf, and the Tags page are built for libraries of a few hundred thousand files.

- **Only what you look at is loaded.** Searching, sorting, filtering, and folder grouping happen on the server. The browser fetches one folder at a time as you open it, and a page at a time as you scroll - there is no "page 2", the list just keeps going. Opening a gallery of 180,000 tokens costs a few small requests, not the whole library.
- **Folder counts are exact before anything opens.** Each folder shows how many items it holds under the current filters, so you can see where things are without expanding everything.
- **Folders start collapsed**, including ones a new search or filter brings into view - expand just the ones you need.
- **Search understands field prefixes.** A gallery's search box, and a system page's, accept the same `field:value` syntax as the main [search](/guide/search#narrowing-a-gallery-or-system) - `tag:dungeon` or `title:goblin` in a gallery, `author:gygax`, `year:1985` or `code:PZO` on a system's page.

The database work behind these views is indexed: on a test library of 187,000 tokens, 18,000 maps, and one system holding 30,000 books, every browse request answers in well under half a second. The first startup after upgrading builds those indexes, which takes a few seconds per hundred thousand files.

## Valkey / Redis page cache

By default, rendered pages are cached to disk under `DATA_PATH/page_cache/`. For large libraries or multi-user setups, providing a `VALKEY_URL` moves the cache to memory for faster repeat loads.

```yaml
environment:
  - VALKEY_URL=redis://:<valkey password>@valkey:6379/0
```

Give Valkey a password (`--requirepass`) and put it in the URL as shown: besides pages it holds sign-in counters and single sign-on state. See [Docker Compose examples](/deployment/docker-compose) for a complete compose setup with Valkey.

## Memory use

Out of the box Grimoire runs one worker process and settles at roughly 250 MB once your library is scanned (earlier versions used about 550 MB). Reading adds to that for a while: rendering an illustrated page decodes its images at full size, which can take a few hundred MB while people are turning pages. Grimoire hands that memory back on its own, every `PAGE_RECLAIM_INTERVAL` page renders and again once nobody has viewed a page for `PAGE_IDLE_RELEASE` seconds (5 minutes by default), so it returns to its idle size after a session.

- **`PAGE_RECLAIM_INTERVAL`** (default `10`) - lower it (e.g. `3`) to keep peaks down when several people read at once. It costs a few percent of render speed.
- **`PAGE_IDLE_RELEASE`** (default `300`) - how long reading has to stop before open books are closed and their memory released. The first page after that reopens the book, which is quick.
- **`OCR_CONCURRENCY`** - each parallel OCR job adds 50-250 MB while it runs (see [Speeding it up](#speeding-it-up)).

## Workers

`WORKERS` sets how many worker processes serve requests. The default is `1`, and it is the right choice for most servers: one worker serves many people at once on separate threads, and a single reader turns pages just as fast as with two. Each extra worker is a full second copy of the app and adds roughly 125-190 MB.

Where a second worker pays off is several people turning pages that have never been viewed before, at the same moment. In testing with four people doing that on a 4-core host, one worker served 5.5 pages a second and two served 7.6 (1.9.0's two workers managed 4.5). On a 2-core host the gain was only about 10%. Pages someone has already viewed come from the cache and cost almost nothing either way.

Raise it to `2` when **all** of these are true:

- **Three or four people or more regularly read at the same time**, such as a whole table following along in the reader during a session.
- **Page turns feel slow when they do**, and `docker stats` shows Grimoire at about 100% CPU (one full core) while the host has more cores idle. That is the sign one process is the bottleneck.
- **The host has 4 or more CPU cores and about 200 MB of RAM to spare.**

If page turns are slow but the CPU is not pinned at one core, another worker will not help: look at disk speed, or add [Valkey](#valkey-redis-page-cache) so pages people have already viewed load from memory. More workers than CPU cores never helps.

When you run more than one worker, also set `VALKEY_URL`. Without it each worker keeps its own rendered-page cache, sign-in rate limits and library-scan progress, so the scan status in Settings can look stuck while a scan runs in the other worker.

```yaml
environment:
  - WORKERS=2
  - VALKEY_URL=redis://:<valkey password>@valkey:6379/0
```

Grimoire does not add or remove workers on its own. Starting a worker takes a few seconds and 125-190 MB, so it would arrive after a burst of page turns had already passed, and the extra copy would split those per-worker caches mid-session.
