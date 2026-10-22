# Reader

Grimoire renders PDF pages server-side as WebP images, keeping the viewer fast on mobile without loading large files into the browser.

## Features

- **Page-by-page navigation**: swipe or use arrow keys
- **Zoom up to 800%**: toolbar buttons, `+`/`-`/`0`, Ctrl + scroll, or pinch on mobile. Past about 200% the part of the page you are looking at sharpens a moment after you stop moving, so small print stays crisp
- **Zoom carries across pages**: turn the page while zoomed and you stay zoomed, in the same column, at the top of the next page (or the bottom of the previous one). A new book, a switch between Page and Spread, or the reset button returns to 100%
- **Spread mode**: display two pages side-by-side (even/odd offset toggle). **Join pages**, under Spread in the **⋮** menu (or the **Join pages** button in the PDF map viewer), draws the two pages as one image with no gap between them - the way a map or illustration printed across both pages looks in the book. It is remembered per book and per map
- **Table of contents**: jump directly to any chapter
- **Bookmarks**: save page bookmarks or text-selection highlights, with inline highlight overlay
- **Native PDF viewer**: switch to the browser's built-in PDF viewer any time via the toolbar
- **Download**: download the original PDF file
- **Export pages & cut-outs**: save chosen pages, or areas cut out of them, as their own files - see below

## Exporting pages and cut-outs

Open **⋮ → Export pages & cut-outs** (or press `e`). PDF maps have the same **Export** button in the map viewer. The panel has two tabs.

### Whole pages

Type the pages you want - `12-15, 20` - or tap a shortcut: **This page**, **This spread**, the table-of-contents **Section** you are in (handy for a character sheet at the back of a book), or **All**.

- **PDF** keeps the text selectable and prints at full quality.
- **PNG**, **JPEG** and **WebP** save one image per page at the resolution you choose (Screen 96 dpi, Standard 150 dpi, Print 300 dpi). The panel shows the pixel size you will get. One image export takes up to 100 pages and 250 megapixels in all - about 28 pages at Print, or 100 at Standard. For more, save as PDF or export in parts.
- **Join facing pages into one image** turns a map printed across a spread into a single picture, paired the same way the reader pairs spreads.

### Cut out

Switch to **Cut out** and the page becomes a drawing surface. Pick **Rectangle**, **Square** or **Circle** and drag over a map, a table or a token; hold Shift to keep a rectangle square. In spread mode you can draw across both pages.

**Find shapes on this page** outlines every separate picture on the page - each token on a token sheet, round or square, even when the tokens touch. It reads the PDF's structure first and, for a sheet saved as one flat picture, the pixels against the page's plain background. Remove any you do not want; on a page with no plain background (a full-page illustration), draw them by hand.

Every cut-out becomes its own file, named after the book and page unless you type a name. Round cut-outs have transparent corners in PNG and WebP. With a cut-out selected, the arrow keys move it, Alt and the arrow keys resize it, and Delete removes it.

One file downloads as itself; several download as a zip.

### Saving into the library

Admins on a writable library also get **Save to library**, which writes the files into a library folder - beside the original, or into Tokens or Maps - instead of downloading them. Existing files are never overwritten, and Grimoire scans the folder straight away so the new files show up in the library. Tokens only accept images, so a PDF export has to go to Books or Maps.

## Performance

PDF pages are rendered as WebP images at up to 3000px wide. Rendered pages are cached aggressively (`Cache-Control: max-age=31536000, immutable`) so repeat visits are instant.

For large libraries or multi-user setups, add a [Valkey page cache](/deployment/docker-compose#with-valkey-cache) to keep rendered pages in memory rather than disk.
