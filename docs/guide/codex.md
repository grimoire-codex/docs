# Grimoire Codex DB

[Grimoire Codex DB](https://db.grimoirecodex.org) is a community catalogue of TTRPG
systems and books, in the spirit of MusicBrainz or TMDB: anyone can add or
correct a record, and edits from newer contributors wait for review. Grimoire has
it built in, so there is nothing to install.

It stays **off and hidden until an admin sets `GRIMOIRE_CODEX_DB_API_TOKEN`** (see
[Turning it on](#turning-it-on)). Until then Grimoire Codex DB does not appear in
Fetch metadata, the editors or Settings.

You can use it to:

- **Fill in details.** Grimoire Codex DB is the first source in **Fetch metadata** on every
  book and game system. You see a field-by-field comparison and choose what to
  apply. Nothing changes until you do.
- **Link your records.** Applying a Grimoire Codex DB result also links the book or system to
  its Grimoire Codex DB record, and the editor then shows a link to that record.
- **Give back.** GMs and admins can add a book or system to Grimoire Codex DB, or
  send corrections to the linked record.

## Turning it on

1. Sign in on [Grimoire Codex DB](https://db.grimoirecodex.org) and create a token
   under **Account → API tokens**. It starts with `cdx_` and is shown only once.
2. Add it to Grimoire's environment as `GRIMOIRE_CODEX_DB_API_TOKEN`, for example in
   `docker-compose.yml`:

   ```yaml
   environment:
     - GRIMOIRE_CODEX_DB_API_TOKEN=cdx_...
   ```

3. Restart Grimoire. **Settings → Add-ons → Grimoire Codex DB** now appears; use
   **Test connection** to check the token.
4. **Recommended:** turn on **Send file hashes when looking up books** there, or set
   `GRIMOIRE_CODEX_DB_SEND_HASHES=true`. See [Privacy](#privacy) for what it
   shares.

The token lives only in the environment. Grimoire never saves it to its database
or backups and never shows it. To turn Grimoire Codex DB off, remove the
variable and restart.

## Looking a book or system up

1. Open the book's or system's editor and choose **Fetch metadata**. While
   reading a book, the same button is at the top of its details panel's editor.
2. Pick **Grimoire Codex DB** as the source.
   - **For a book**, the first search uses everything Grimoire already knows: the
     title, file name, ISBN, authors, publisher, system, year and page count.
     Each result says why it matched, for example *Same ISBN, Same author*. If
     the right book isn't there, type a title to search for it instead.
   - **For a system**, it searches by name.
3. Choose a result, or paste a Grimoire Codex DB link or record id if you already have one.
4. Tick the fields you want and apply them.

The **Grimoire Codex DB link** row is ticked for you when the record isn't linked
yet. Untick it to copy fields without linking.

Category, page count and file size are never offered from Grimoire Codex DB, because they
describe your copy of the file rather than the book.

## Sending a book or system to Grimoire Codex DB

This is open to GMs and admins. The Grimoire Codex DB section of the editor, just above
**Save**, offers:

- **Add to Grimoire Codex DB**, for a record that isn't linked yet. It asks where the
  information comes from, because reviewers need to know. Link the game system
  first: Grimoire Codex DB needs to know which system a book belongs to.
- **Send corrections**, for a linked record. Tick only the fields you want Grimoire Codex DB
  to take from your library. Grimoire Codex DB updates just those fields and leaves the rest
  of its record alone.
- **Unlink**, which forgets the link in Grimoire only. Nothing changes in Grimoire Codex DB.

Whether an edit goes live at once or waits for review is up to Grimoire Codex DB, and depends
on the account behind the token. A new record that waits for review isn't linked
yet, since it may not be accepted. Look it up again once it has been approved.

::: tip Shared servers
Everything sent is credited to the Grimoire Codex DB account that owns the token. On a
server with several GMs, they all send as that one account.
:::

## Settings

Admins find these under **Settings → Add-ons → Grimoire Codex DB**, once
`GRIMOIRE_CODEX_DB_API_TOKEN` is set.

| Setting | Default | What it does |
|---|---|---|
| Grimoire Codex DB address | `https://db.grimoirecodex.org` | Change only to use a self-hosted Grimoire Codex DB. |
| Send file hashes when looking up books | Off | **Recommended on.** Identifies a book exactly, even with a messy file name, and helps other libraries match the books you send. It does tell Grimoire Codex DB which files your library holds. |

**Test connection** checks that Grimoire Codex DB can be reached and shows which account the
token belongs to.

Each setting can also be pinned with an
[environment variable](/configuration/env-vars#grimoire-codex-db), which shows it
read-only in Settings.

## Privacy

Grimoire only contacts Grimoire Codex DB when someone asks it to: fetching metadata with
Grimoire Codex DB selected, sending a record, or testing the connection. Nothing runs in the
background.

- A book lookup sends the details listed above. A system lookup sends its name.
- File hashes are only sent if you turn them on. We recommend it for better
  matches, but it is your call: a hash tells Grimoire Codex DB that your library
  holds that exact file.
- Every request carries the token, so Grimoire Codex DB sees lookups as coming from
  the token's account.
- File paths, access levels and cover choices never leave your server.

To keep Grimoire entirely offline, leave `GRIMOIRE_CODEX_DB_API_TOKEN` unset.
