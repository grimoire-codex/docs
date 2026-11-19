# User Roles

Grimoire has four roles:

| Role | Permissions |
|---|---|
| `admin` | Everything: user management, app settings, metadata editing, rescan, logs |
| `gm` | Read everything, edit metadata, create GM campaigns, trigger rescan |
| `player` | Read-only access, personal campaigns, session notes |
| `guest` | Code-only account scoped to a single campaign. No access to the library, maps, tokens, audio, or search. See [Guest Access](/guide/guest-access). |

## Creating users

Create accounts in **Settings → People** after logging in as admin.

## Pre-seeding users

You can pre-create accounts before first launch by placing a `users.json` file in your data directory. See [Pre-seeding Users](/configuration/users).

## Restricting books

By default every user can see every book. If you would rather keep the adventure module your players are currently inside out of their hands, books, systems, folders and whole categories can be restricted to a minimum role.

There are three restriction levels:

| Level | Who can see it |
|---|---|
| **Everyone** | No restriction. The default. |
| **GMs and admins only** | Hidden from players and guests. |
| **Admins only** | Hidden from GMs as well. |

Restricted content is **hidden**, not locked - it disappears from the library, search, system pages, downloads, favourites, and the [OPDS feed](/guide/opds). A title and cover are themselves the spoiler, so a padlock nobody can open would defeat the point.

### Where restrictions can be set

Restrictions resolve most-specific-first, so a narrower setting always wins:

```
book  →  system  →  category default  →  everyone
```

- **A single book** - in the book editor, or for many at once through bulk edit. A book set to *Inherit* takes its system's or category's setting; a book set explicitly to *Everyone* stays visible even inside a restricted system, which is how a free player's guide can sit in an otherwise admin-only adventure line.
- **A whole system** - in the system editor. Restricting a system hides the system itself along with every book in it that has no setting of its own. A container's level does not apply to the systems inside it: restrict each one you want hidden. Someone who can see a system but not its container sees it at the top level of the library, with the container's name left out of it and its books' paths.
- **A whole category** - in **Settings → Access & sharing → Category Restrictions**. This is the library-wide default for that category. Core rulebooks and character sheets cannot be restricted: everyone at the table needs those by definition.
- **A folder** - in **Settings → Access & sharing → Folder Restrictions**. Pick any folder inside `books/` or `armory/` and every book beneath it is restricted, at any depth. This is how to keep a GM's guide from players when it sits with a system's core books: put it in a folder such as `books/D&D 5e/Core/GM/` and restrict that folder. A folder only ever *adds* a restriction - its books get the stricter of the folder's level and their system's or category's - so the one way to open a book inside a restricted folder is to set that book to *Everyone*. Folder restrictions are stored in Grimoire, not in the library, so they work on a read-only library. They follow the folder when it is renamed or moved in the [file manager](/deployment/file-management); a folder renamed outside Grimoire loses its restriction, and the setting shows it as *Folder not found*.

Only admins can change any of these, including in bulk edit.

### Granting one GM access

A locked-down library still needs the GM running the campaign to reach their own material. In **Settings → People**, expand a GM's row and use **Library access grants** to give that person access to a specific system or book without lowering the restriction for anyone else.

Grants are only available for GMs. Admins already see everything, and players and guests are exactly who the restrictions exist to exclude, so they cannot be granted past one. A grant is removed automatically if the user stops being a GM.

### Restricted books in campaigns

A restricted book can still be linked into a campaign - the GM needs it - but it is always forced to **GM-only** visibility there, and cannot be made public or private to the players. If you restrict a book, system or folder that was already shared, its existing shares are demoted to GM-only for you.

## Explicit content

Each user account has an `allow_explicit` flag. Users with this disabled will not see game systems, books, or tokens marked as explicit. Admins can toggle this per-user; users can toggle it for their own account in **Settings → Preferences → Content Preferences**.

## Campaign access

Each user has a **campaign access** toggle (managed per user in **Settings → People**; enabled by default). Disabling it stops the user from creating campaigns, being added to new ones, and editing resources, and locks any campaign they own to read-only, but never deletes existing campaigns. When OIDC is configured this can be driven by the provider's permissions claim. See [Per-user campaign access](/guide/campaigns#per-user-campaign-access).

## Display names

Users can set a **display name** in **Settings → Account** that appears in place of their username across the app.
