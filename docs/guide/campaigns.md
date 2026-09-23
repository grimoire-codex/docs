# Campaigns

Grimoire has a built-in campaign tracker with two modes.

## Campaign types

### GM Campaigns
Created by GMs or admins. Supports player invitations, a banner image, character art and character sheets per member, resource linking with per-resource visibility, a markdown wiki for notes, and scheduling.

### Personal Campaigns
Private to a single user. No sharing.

## Creating a campaign

Campaign creation uses a short wizard: pick a system, then choose resources. The system's core books are suggested by default and anything can be added (with a search) or removed, each set to **Shared with players**, **GM only**, or **Private**. The campaign **description** supports markdown, and you can name a **custom game system** that isn't in your library (handy for keeping notes on a system you don't own).

## Invitations

When a GM invites you to a campaign, an **invite banner** appears at the top of the app so you can **accept** (join the campaign) or **decline** it from anywhere. You can dismiss the banner for the current browser session; it reappears the next time you open the app while an invite is still pending.

## Characters, art, and sheets

Campaign members can set a **character name** per campaign (editable by both the GM and the player), upload **character art** (shown as their avatar), and a **character sheet** (PDF or image).

A player can also **create a sheet from a template** (duplicating a form-fillable PDF from the library's **Character Sheets** category, filtered to the campaign's system, or a campaign file) and **fill it in directly in the app**: the real PDF is rendered in the browser and the player types into the form fields on the page itself, then saves a filled copy. The same in-app editing works for any form-fillable PDF a player uploads, so sheets can be updated as characters advance.

Sheets can be downloaded at any time, and re-uploading prompts a warning (with an option to download the current version first) before the previous one is replaced.

Users can also set a **display name** in Account Settings that appears in place of their username across the app.

## Per-user campaign access

Each user has a **campaign access** toggle (admins manage it per user in **Settings → Users**; enabled by default). Disabling it does **not** delete any existing campaigns. It only:

- Prevents the user from creating campaigns, being added to new ones, and editing/linking resources.
- Keeps their read access to campaigns they already own or belong to; in member lists they are flagged as **Access disabled**.
- Locks any campaign they **own** to read-only for everyone (players keep view access, lose all edits) until the owner's access is restored.

When OIDC is configured, this flag can be driven by the provider's [permissions claim](/guide/oidc) (`campaignAccess`); a missing key leaves access enabled.

## Guest invites

Guests let you share a single campaign with people who don't have full accounts, a code-only account scoped to one campaign, with no access to the rest of the library. See the [Guest Access](/guide/guest-access) guide.

## Notes wiki

Each campaign has a full-page markdown **wiki** (opened from the campaign overview) for building out the world: session recaps, lore, NPCs, and plans:

- **Markdown** with tables, images, and the usual formatting, edited side-by-side with a live preview.
- **Visibility per page**: *GM only*, *Public* (all members), or *Private* (specific members). Change it from the visibility badge on the page: the badge is a dropdown, and for *Private* pages it lists members so you can grant or revoke access without opening the editor.
- **GM secrets inline**: wrap text in `||double pipes||` (or use the **GM secret** button) to hide just that span inside an otherwise shared page. The GM sees it highlighted; players receive nothing at all — the secret is stripped before the page reaches them, leaving no text and no placeholder, so a player never even knows a secret is there. (Personal campaigns keep everything, since only you can read them.) Your secrets survive a player editing their own page: when they save, the server re-weaves each secret back into the exact spot it was — even if the player edited the text above and below it. If a player rewrites so much that the spot can't be found, the secret isn't lost; it's moved to the bottom of the page for you to reposition.
- **Nested pages**: organize the sidebar as a tree: any page can hold subpages to any depth. Drag pages to re-nest them, add a subpage from the parent row, and collapse/expand branches. Deleting a page lifts its subpages up to the parent rather than removing them.
- **Page links**: write `[[Page Title]]` to link pages; missing targets are auto-created as stubs, and each page shows what links back to it.
- **Grimoire embeds**: drop a book (optionally at a page), map, token, audio track (plays in the global player; a note with several can be played as a playlist via **Play all**), or campaign file straight into a page. The embed picker lists the campaign's linked resources. You can also **upload an image** right from the picker, and it's embedded inline and added to your linked resources.

::: info Migration from session notes
Existing session notes are automatically rolled into wiki pages (nested under a "Session Notes" page) the first time the new version starts; empty notes are discarded.
:::

### Import & export

GM-only. Export the whole wiki as a Markdown `.zip` (one file per page with YAML frontmatter, an Obsidian-style vault) or a JSON bundle. Import pages from Markdown, a Grimoire JSON bundle, or a **LegendKeeper** export (`.json`, `.lk`, or `.zip`). **Choose folder** picks a directory of Markdown straight off disk (an Obsidian vault, say) and uploads the whole set at once, with no zipping first; only Markdown is read, and a vault's `.obsidian/` is skipped. Either way the folder structure becomes page nesting: `Places/Cities/Waterdeep.md` imports as a Waterdeep page under Cities under Places. A folder note (`Places/Places.md`) or an `index.md` becomes that folder's page rather than a duplicate, and a page's own `parent:` frontmatter outranks its folder. LegendKeeper HTML and ProseMirror page bodies are converted to Markdown and the page hierarchy is preserved; LegendKeeper-only block types (e.g. secrets, embeds) are dropped. Imports are non-destructive: pages are always added, never overwritten.

## Resources

Link books, maps, tokens, and audio, or upload campaign files (handouts, images, etc.) the GM keeps with the campaign. Each resource has a visibility:

| Visibility | Who can see it |
|---|---|
| **Public** | All players |
| **Private** | Specific players (e.g. a handout for 2 of 4) |
| **GM only** | Just the GM |

Resources group under their type by default, but the GM can create custom categories (e.g. *Player Handouts*), drag items between categories and reorder them, and delete categories (keeping items uncategorized or unlinking them). Each group can be **collapsed or expanded** (remembered per campaign), and the GM can **reorder all the groups** (custom categories and the built-in Books / Maps / Tokens / Audio / Files groups together) from the category manager.

Uploaded campaign files live in the data directory, separate from the library. Admins can disable these uploads app-wide or cap them by per-file and per-campaign size in **Settings → App** (admins themselves are exempt).

## Session scheduling

GM campaigns support recurring session schedules:

| Pattern | Description |
|---|---|
| **Weekly** | Same day(s) every week |
| **Biweekly** | Every other week, anchored to a reference date |
| **Monthly** | Nth weekday of the month (e.g. "first Friday") |
| **Custom** | Explicit list of dates |

Session note stubs are auto-created the day before each scheduled session. Players can mark their availability for upcoming dates, and the GM can cancel individual dates.
