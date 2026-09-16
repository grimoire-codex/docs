# OCR (Scanned Books)

Some PDFs are just scanned page images with no embedded text layer. This is common with older, out-of-print, or home-scanned books. Because there's no text to read, they can't be full-text searched on their own and show an **Image Only** badge in the library.

Grimoire's default image bundles the [Tesseract](https://github.com/tesseract-ocr/tesseract) OCR engine (English), which "reads" the scanned pages and adds the recognised text to the search index. Once a book has been through OCR it shows an **OCR** badge and its pages become searchable like any other book. No extra container or service is required.

## How it works

OCR runs quietly in the background, so it never holds up the rest of your library:

1. A scan first indexes all your regular (text-based) books, maps, tokens, and audio. These are searchable right away.
2. Any scanned book with no text layer is queued for OCR and processed afterward, one page at a time.
3. Progress is saved as each page is read. If the server restarts, or you stop and restart a scan, OCR picks up exactly where it left off instead of starting the book over. Even a very large scanned book will finish, however long it takes.

Each page has a time budget (`OCR_PAGE_TIMEOUT`, default 120 seconds). A page that runs over it is skipped so one bad page can't stall a book forever — the book still finishes, but the skipped pages aren't searchable and it's badged **OCR 14/206** rather than **OCR**. On slow hardware this can affect a lot of pages; see [Pages that get skipped](#pages-that-get-skipped).

You can watch the OCR queue drain under an **OCR** phase in the scan status (**Settings → Maintenance**), which shows a progress bar and the book currently being processed.

Even with a big collection of scanned books, only the OCR queue is affected: the rest of your library stays fully available and searchable while OCR works through the backlog.

## Enabling and disabling

OCR is on by default on the standard image. There are two ways to turn it off:

- Set `OCR_ENABLED=false`, or
- Set `OCR_CONCURRENCY=0` (a convenient off switch if OCR keeps erroring or running your machine out of memory and you just want it to stop).

Either way, scanned PDFs are simply left unindexed, exactly as they were before OCR existed. Any books already queued for OCR are left untouched and will resume if you re-enable it later.

## Slim image (no OCR)

If you don't want the Tesseract engine and its language data in your image, use the **`-slim`** tags (e.g. `hunterreadca/grimoire:slim`, `hunterreadca/grimoire:1.5.0-slim`). These omit Tesseract for a smaller image. OCR is automatically disabled there and Grimoire degrades gracefully: scanned PDFs keep the **Image Only** badge and everything else works normally.

The [Compose Generator](/compose-generator) has a **Grimoire image** option to select the slim variant, which pins the correct `-slim` tag for you.

If you later switch from a slim image to the standard one (or enable OCR), any books previously skipped as image-only are automatically queued for OCR on the next startup scan.

## Adding languages

The default image ships with **English** only. To OCR books in other languages, you need two things: the extra language's Tesseract data file, and `OCR_LANGUAGES` set to include it.

### 1. Set the languages

`OCR_LANGUAGES` takes a `+`-joined list of [Tesseract language codes](https://tesseract-ocr.github.io/tessdoc/Data-Files-in-different-versions.html):

```yaml
environment:
  - OCR_LANGUAGES=eng+deu+fra   # English, German, French
```

Listing multiple languages lets a single page be recognised in any of them, which is useful for mixed-language books. Adding languages you don't need slows OCR down slightly, so keep the list to what you actually have.

### 2. Provide the language data files

The extra languages' `.traineddata` files must be present in the image. You don't need to rebuild — mount them at runtime. Download the files you want from the [tessdata repository](https://github.com/tesseract-ocr/tessdata) (or `tessdata_fast` for smaller, faster files) into a folder on your host, then mount that folder and point Tesseract at it with `TESSDATA_PREFIX`:

```yaml
services:
  grimoire:
    image: hunterreadca/grimoire:latest
    environment:
      - OCR_LANGUAGES=eng+deu+fra
      - TESSDATA_PREFIX=/tessdata
    volumes:
      - /path/to/your/library:/library:ro
      - /path/to/grimoire/data:/data
      - /path/to/your/tessdata:/tessdata:ro   # holds eng.traineddata, deu.traineddata, fra.traineddata
```

::: warning Include English
When you set `TESSDATA_PREFIX`, Tesseract looks **only** in that directory, so include `eng.traineddata` alongside your extra languages even though the image ships with English by default.
:::

After changing languages or adding data files, trigger a rescan (or restart) so image-only books are re-processed with the new configuration.

## Re-OCR a single book at a higher DPI

`OCR_DPI` sets the render resolution for the whole library, and the default (`150`) is enough for most scans. Occasionally one faint or low-quality book reads better at a higher resolution. Instead of raising the global setting and re-processing everything, you can re-OCR just that one book:

1. Find the scanned book in its game system (it shows an **OCR** or **Image Only** badge).
2. On the book, open the **actions menu** (the ⋮ button) and choose **Re-OCR…** (it appears only for OCR'd books, for GM/admin users).
3. Optionally enter a DPI (for example `300`) and run it. Leave the DPI blank to re-OCR at the global default.

The book's existing search text is cleared and it's re-read in the background at the resolution you chose — the rest of your library is untouched and stays searchable throughout. Progress shows under the OCR phase in the scan status, and the chosen DPI is remembered for that book so it resumes at the same resolution if the server restarts mid-run.

This requires the **GM** or **admin** role, and only applies to scanned/OCR'd books. A book that already has an embedded text layer has nothing to re-OCR — but if you edited such a PDF externally and need Grimoire to pick up the new content, use **Re-scan & re-index** from the same actions menu instead (see below).

## Re-scan &amp; re-index a single book

When you edit a PDF in place — embedding encounter notes, adding errata, or otherwise changing its text — Grimoire's search index for that book goes stale until the next full library rescan. Instead of rescanning everything, you can re-read just that one file:

1. Find the book in its game system.
2. Open the **actions menu** (the ⋮ button) and choose **Re-scan &amp; re-index**.

The file is re-read from disk in the background: its page count and cover thumbnail are refreshed if they changed, its old search rows are cleared, and its text is re-extracted and re-indexed. A book that has become image-only is automatically handed to the OCR queue. Progress shows in the scan status.

This requires the **GM** or **admin** role and applies to any PDF. (For scanned/OCR'd books the menu offers **Re-OCR…** instead, described above.)

## Performance and tuning

For large scanned collections, two settings control how fast the OCR queue drains. Full details are on the [Performance](/configuration/performance#ocr) page, but in short:

- **`OCR_CONCURRENCY`** (default `1`) — how many scanned books to OCR at once. Raise it on multi-core hosts with spare memory (each parallel worker typically adds on the order of 50–250 MB of RAM, depending on the scanned page's size and quality, and don't exceed your CPU core count); keep it at `1` on small devices like a Raspberry Pi.
- **`OCR_DPI`** (default `150`) — how sharp pages are rendered before reading. Lower is faster and lighter; higher can improve results on faint or low-quality scans.

See [Environment Variables → OCR](/configuration/env-vars#ocr) for the full list of OCR settings.

## Pages that get skipped

Each page gets `OCR_PAGE_TIMEOUT` seconds (default `120`) to be read. Pages that run over are skipped, and the book carries on to the next one.

How long a page takes is driven by how **dense and noisy** the scan is far more than by its dimensions: on a low-power CPU, an ordinary page might read in 13 seconds while a cramped, speckled one from the very same book needs four minutes. So on slower hardware this can affect many pages of a single awkward book, while the rest of the library OCRs fine.

A book with skipped pages is still indexed and still searchable — but only for the pages that were read. You can spot it two ways:

- **In the library**, it's badged **OCR 14/206** in amber instead of the plain green **OCR**. The numbers are pages read out of total; hover for a tooltip with the details.
- **In the logs**, a warning when the book finishes names how many pages were skipped (`docker logs grimoire`).

To recover the missing text:

1. Raise the budget, e.g. `OCR_PAGE_TIMEOUT=600`, and restart. Raising it costs nothing when pages finish quickly — it's a ceiling, not a delay. `0` means no limit at all.
2. Re-read the book from its actions menu (**⋮** → **Re-OCR…**).

If you'd rather trade accuracy for speed, lowering `OCR_DPI` also brings slow pages under the budget — though on the worst scans, dropping from 150 to 100 DPI often still isn't enough.
