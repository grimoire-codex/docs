# Community Add-ons

Add-ons are optional extensions you install from a community catalogue. Today
they are **metadata scrapers**: they look a game system or a book up on an
external source and offer to fill in its details, so you don't have to type
publisher, year, licence, authors, and genre by hand.

Definitions live in the separate
[community-add-ons](https://github.com/grimoire-codex/community-add-ons)
repository rather than inside Grimoire, so when a source changes its layout the
fix is a community pull request — not a wait for the next Grimoire release.

## Installing an add-on

Go to **Settings → Add-ons** (admin only).

1. Click **Refresh**. Grimoire fetches the catalogue from the index URL — by
   default the community repo.
2. Find the add-on you want and click **Install**.

Installed add-ons appear under **Installed**, where you can disable one
temporarily or remove it entirely.

## Keeping add-ons up to date

Add-ons change — that's rather the point of them living outside Grimoire. When a
source redesigns its site or moves its data, the fix ships as a new version of
the add-on rather than waiting for a Grimoire release.

When a newer version is available, the add-on is badged with it and an
**Update** button appears. **Update all** does the lot in one go.

::: warning Updates that change a script
If an update changes an add-on's Python script, its approval is reset and you'll
be asked to confirm again — even via *Update all*. You approved the old code,
not the new code. Add-ons without scripts update silently.
:::

An add-on you installed by hand isn't in the catalogue, so Grimoire can't tell
when it changes; update it the same way you installed it.

## Filling in metadata

1. Open a game system or a book and click **Edit**.
2. Click **Fetch metadata**.
3. Choose a source and confirm the search term (it starts as the system's name
   or the book's title).
4. Pick the right match from the results.
5. Review the changes and tick the ones you want, then **Apply**.

Each add-on works on one kind of thing, so the button only offers sources that
suit what you're editing.

::: tip Links are added, never replaced
Fetching adds the source's own page to your links rather than replacing them, so
anything you'd added yourself stays put. The dialog shows only the links being
added.
:::

::: tip Nothing is overwritten behind your back
Fields you have already filled in are **not** ticked by default. Only genuinely
empty fields are pre-selected, and anything that conflicts is shown with your
current value struck through so you can decide. Nothing is written until you
press Apply.
:::

Each row is labelled:

| Label | Meaning |
|---|---|
| **new** | You have nothing here yet — ticked by default |
| **differs** | You already have a different value — left unticked |
| **already set** | Yours already matches — nothing to do |

Because editions are usually separate entries upstream, searching a bare system
name often returns several results. Pick the edition you actually own.

## When you already know which one

Searching a big catalogue can be fiddly — DriveThruRPG in particular returns a
lot of near-misses for a well-known title. If you already have the item's page
open on the source, you can skip searching entirely.

Click **"Know the exact one? Paste a link or ID"**, paste the page's URL, and
press Look up. It goes straight to the review step.

It accepts:

- a full page URL — `https://www.drivethrurpg.com/en/product/170689/blades-in-the-dark`
- a shorter one — `drivethrurpg.com/product/170689`
- just the ID — `170689`

Any tracking or affiliate parameters on the end of a pasted link are ignored.
If the text isn't a link for that source, you'll be told so rather than left
waiting on a request that can't succeed.

Not every source offers this — the option only appears when the add-on knows how
to read an ID out of that source's URLs.

## Available sources

### For game systems

| Add-on | Source | Fills in |
|---|---|---|
| TTRPG Wiki | [ttrpgwiki.com](https://ttrpgwiki.com) | Description, publisher, year, licence, system family, edition, genres, dice, tags, links |

### For books

| Add-on | Source | Fills in |
|---|---|---|
| DriveThruRPG | [drivethrurpg.com](https://www.drivethrurpg.com) | Title, description, authors, artists, publisher, genres, ISBN, year, links |

DriveThruRPG's catalogue is enormous and full of third-party supplements, stock
art, and translations, so searching a well-known title returns a lot of
near-misses. **Check the publisher shown next to each result** before applying
it — that's why it's there. A book sold only through its publisher's own store
won't be found at all.

More are welcome — see [contributing](#writing-your-own) below.

## Add-ons that run code

Most add-ons are plain configuration files. Grimoire reads them itself, and no
third-party code ever runs on your server.

A few sources are too awkward for that, so an add-on may include a Python
script. **Those run code on your machine**, and Grimoire treats them
accordingly. A scripted add-on runs only if you have:

1. Turned on **Allow add-on scripts** (off by default), **and**
2. Approved that specific add-on when installing it, in a dialog that names the
   script and shows its checksum.

Add-ons that include a script are labelled **Runs code** in the list.

When one does run, it runs in a separate short-lived process with a time limit
and no access to your database — so a buggy add-on can't take Grimoire down with
it. That contains accidents, but it is not a full sandbox: a script can still
reach the network and read files your server can read.

::: warning Treat it like any other program
Installing a scripted add-on is like running software someone sent you. Only
install ones you trust. Every add-on's source is public in the community repo
precisely so it can be read before you install it — and downloads are checked
against a checksum, so a tampered file is refused.
:::

If an add-on updates and its script changed, approval resets and you'll be asked
again.

## Using a private add-on

You don't have to publish an add-on to use it. Put its folder in
`DATA_PATH/add-ons/<id>/` and restart Grimoire; it appears alongside the rest.

To test one you're developing, serve the repo locally:

```bash
python3 -m http.server 8000
```

Then set the index URL to `http://localhost:8000/index.json` and hit Refresh.

## Writing your own

The [add-ons repo](https://github.com/grimoire-codex/community-add-ons) has the
authoring reference. In short, a scraper is a small YAML file describing where
the data lives, how to match a search against it, and how its fields map onto
Grimoire's:

```yaml
id: my-source
name: My Source
version: 1.0.0
kind: scraper
target: game-system

source:
  url: https://example.com/data.json
  format: json
  cache_ttl: 86400

search:
  fields:
    - { field: name, weight: 1.0, strategy: fuzzy }
  identity: { from: id }
  label: { template: "{name} ({edition})" }

map:
  description: { from: summary }
  year: { from: released }
  publishers: { from: publisher, as: link_list }
```

Contributions are welcome. Please respect the source you're scraping: check its
terms, set a generous `cache_ttl`, prefer a bulk endpoint over hammering
individual pages, and credit it with an `attribution` line.

## Frequently asked

**Does fetching metadata overwrite my edits?**
No. Fetching only shows you a comparison. Nothing changes until you tick fields
and press Apply, and fields you've already filled in start unticked.

**Will it change my page count or file size?**
No. Those are read from your actual file when the library is scanned, which is
more trustworthy than a store listing, so no scraper maps them.

**How often does it hit the source?**
As little as it can. Where a source publishes its whole catalogue in one file
(TTRPG Wiki), Grimoire downloads it once and caches it — typically a single
request per day covers every lookup, and the cached copy keeps working if the
source goes down. Where it has to search (DriveThruRPG), each search is one
request, cached for an hour, plus one more when you pick a result.

**Can players use this?**
No. Fetching metadata requires the GM or admin role, and managing add-ons is
admin-only.

**An add-on stopped working.**
The source probably changed. Open an issue on the
[add-ons repo](https://github.com/grimoire-codex/community-add-ons/issues) — the
definition can be fixed there without updating Grimoire.
