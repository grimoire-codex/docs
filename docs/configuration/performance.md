# Performance

## Indexing

On first startup Grimoire scans the library and indexes every PDF page for full-text search. This can take several minutes for large collections. The index is stored in the data volume and subsequent startups are fast.

Use the **Rescan** button in the sidebar to pick up newly added files, or configure a scheduled rescan in **Settings → Maintenance**. For large libraries you can also rescan a single corner: every system, category, subfolder, and map/token/audio group has its own rescan button that re-scans just that folder.

## OCR

Some PDFs contain only scanned page images with no embedded text layer (common with older, scanned books). These can't be full-text searched from their text layer alone and show an **Image Only** badge.

The default Grimoire image bundles the [Tesseract](https://github.com/tesseract-ocr/tesseract) OCR engine (English), so scanned image-only PDFs are run through OCR and their recognised text is added to the search index. Books indexed this way show an **OCR** badge. No extra container or service is required.

OCR runs quietly in the background, so it never holds up the rest of your library. A scan indexes all your regular (text-based) books, maps, tokens, and audio first — those are searchable right away — and then works through the scanned books afterward. Even if you add 100+ scanned books at once, the rest of your library stays available while they're being processed.

Scanned books are processed **page by page**, and progress is saved as it goes. If the server restarts (or you stop and start a scan), OCR simply picks up where it left off instead of starting the book over, so even a very large scanned book will finish however long it takes. You can watch the queue drain under an **OCR** phase in the scan status (**Settings → Maintenance**).

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

## Valkey / Redis page cache

By default, rendered pages are cached to disk under `DATA_PATH/page_cache/`. For large libraries or multi-user setups, providing a `VALKEY_URL` moves the cache to memory for faster repeat loads.

```yaml
environment:
  - VALKEY_URL=redis://valkey:6379/0
```

See [Docker Compose examples](/deployment/docker-compose) for a complete compose setup with Valkey.

## Workers

The `WORKERS` environment variable controls how many uvicorn worker processes run. More workers allow more concurrent requests but use more memory. 2–4 is typical for home use.

```yaml
environment:
  - WORKERS=4
```
