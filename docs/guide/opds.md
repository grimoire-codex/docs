# OPDS Catalog

Grimoire supports the [OPDS 1.2](https://specs.opds.io/opds-1.2) catalog format, allowing e-reader apps (Panels, Chunky, Kybook, KOReader, etc.) to browse and download books directly from your library.

## Enabling OPDS

Set `OPDS_ENABLED=true` and `BASE_URL` to your instance's public URL:

```yaml
environment:
  - OPDS_ENABLED=true
  - BASE_URL=https://grimoire.example.com
```

## Personal feed URLs

OPDS access is per-user. Each user generates their own opaque feed URL in **Settings → Connections → OPDS Feed**. The URL contains a long random token; no username or password is needed by the OPDS client.

- **Enable**: generates a unique feed URL
- **Copy**: copies the URL to clipboard
- **Regenerate**: issues a new token; the old URL stops working immediately
- **Disable**: revokes the token

## Feed versions

Grimoire serves two versions of the feed. Both use the same token, so regenerating or disabling it affects both. Pick one with the **Feed version** dropdown in **Settings → Connections → OPDS Feed**; the URL and the Copy button follow your choice.

| Version | What it shows |
|---------|---------------|
| **v2** (default) | **All Books**, then one entry per game system with its book count, then **Other Books** for anything with no game system. Systems you cannot see, and explicit systems when explicit content is off, are left out. |
| **v1** (original) | One **All Books** entry. Kept as it was, so apps you connected before v2 existed keep working without any change. |

To switch an app from v1 to v2, copy the v2 URL and replace the feed URL in the app.

## Feed URL structure

v2:

```
https://grimoire.example.com/opds/v2/{token}               ← navigation root
https://grimoire.example.com/opds/v2/{token}/all           ← all books
https://grimoire.example.com/opds/v2/{token}/system/{id}   ← one game system's books
https://grimoire.example.com/opds/v2/{token}/other         ← books with no game system
https://grimoire.example.com/opds/v2/{token}/entry/{id}    ← single book
https://grimoire.example.com/opds/v2/{token}/download/{id} ← file download
https://grimoire.example.com/opds/v2/{token}/cover/{id}    ← cover thumbnail (WebP)
```

v1:

```
https://grimoire.example.com/opds/{token}              ← navigation root
https://grimoire.example.com/opds/{token}/all           ← all books
https://grimoire.example.com/opds/{token}/entry/{id}    ← single book
https://grimoire.example.com/opds/{token}/download/{id} ← file download
https://grimoire.example.com/opds/{token}/cover/{id}    ← cover thumbnail (WebP)
```

## Paging large libraries

By default every list of books (**All Books**, each game system, **Other Books**) arrives as one feed holding every book in it. That suits most readers, but with a large library some apps are slow to load the feed or give up on it.

To split the lists into pages, add `?page_size=` to the end of the feed URL you give the app:

```
https://grimoire.example.com/opds/v2/{token}?page_size=100
```

Every list then holds at most that many books, with **next**, **previous**, **first** and **last** links between the pages, and the app loads more as you scroll or turn the page. Pick a size from 1 to 500; 100 is a good start.

- **Set it once, on the root URL.** Grimoire carries `page_size` onto every link in the catalog, so each game system's feed, the book entries and the links back up are all paged the same way. You can also add it to a single feed's URL, such as `/opds/v2/{token}/all?page_size=100`.
- **Leave it off and nothing changes.** Without `page_size` the feed is exactly as it always was, so apps you have already connected keep working.
- **Your app has to follow the links.** Paging relies on the app reading the standard OPDS `next` link, as most OPDS apps do. If yours shows only the first page of books, remove `page_size` from its URL.
- **It works with both versions.** v1 pages its **All Books** feed the same way: `/opds/{token}?page_size=100`.

A page past the end (say the library shrank while you were browsing) comes back empty, with links to the first and last real pages.

## Content filtering

The OPDS feed respects each user's explicit-content preference. Users with explicit content disabled will not see explicit books in their feed.
