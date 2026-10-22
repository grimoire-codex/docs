# Share Links

A share link is a public URL for one book, map, token, audio track or 3D model. Anyone who has it can open that item — no account, no login — until the link expires or you revoke it. Use it to get a handout to a player who doesn't use Grimoire, a battlemap into a virtual tabletop, or a track into a Discord channel.

A link opens exactly one item and nothing else. It isn't a login, and it reveals nothing about the rest of your library.

## Making a link

Choose **Share** from an item's **⋮** menu — on a map, token, audio or model page, on a book in the library, and in the reader for a PDF (other books have a Share button in the reader's toolbar). Give the link an optional label, pick how long it lasts (1, 7, 30, 90 or 365 days, or never), and optionally set a password. The dialog lists every link you've made for that item, ready to copy or revoke.

Each link has two addresses:

| Button | Use it for |
| --- | --- |
| **Copy link** | Sending to a person. It opens a page with a preview, a download button and, if the link has one, a password prompt. |
| **Copy file URL** | Tools that load a file from a URL — a VTT's "import by URL", [OTFBM](./otfbm), a Discord embed. It's the file itself, ending in the file's real extension, because some tools decide how to read a file from that. |

| Item | The file URL serves |
| --- | --- |
| Book, token, audio, 3D model | The file. Audio can be seeked. |
| Image map | The original image |
| PDF map | The first page, as an image |
| Universal VTT map | The battlemap image inside it |

A map's [OTFBM setup](./otfbm) makes a special kind of share link that serves the map resized to match OTFBM's grid rather than the original image. It's listed with your other links as "OTFBM background" and revoked the same way.

### Passwords

A password-protected link asks for the password on its share page before showing anything — not even the item's name. Once entered, that browser stays unlocked for 12 hours, and changing the password locks everyone out again.

A link password needs at least 8 characters (an admin can change this with [`SHARE_LINK_PASSWORD_MIN_LENGTH`](/configuration/security#password-rules)), and commonly used passwords are refused. Wrong passwords are limited per address and per link, so guesses spread across many addresses still run out (`ACCOUNT_RATE_LIMIT`, 20 an hour by default).

Tools that load the file URL directly — VTTs, OTFBM, Discord — can't enter a password, so a protected link only works on its share page. Leave the password empty for those.

## Taking a link down

**Revoke** stops a link working for good. It stays in your list, marked revoked, until you delete it.

A link also stops working on its own when it expires, when you're no longer allowed to create share links, when you can no longer open the item yourself (a higher access level, or explicit content you've turned off), or when the item is deleted. Losing access is not permanent: the link is *suspended*, not deleted, and works again if your access returns.

Your links and their states are under **Settings → Account → Sharing** — still reachable after you lose the right to make new ones, so you can always revoke them. Anyone opening a dead link sees the same "This link doesn't work" page whatever the reason.

## Who can make links

Share links are on by default; `SHARE_LINKS_ENABLED=false` turns them off for the whole server (see [Environment Variables](/configuration/env-vars)). Every link stops working and the Share buttons disappear, but the links are kept for when it's turned back on.

In **Settings → Users**:

- **By role** — admins always can. GMs can by default; players can't. The **Share links** panel above the user table switches GMs and players on or off.
- **Per user** — each user's row has a **Share links** setting (*Follow role*, *Allow* or *Deny*) that wins over the role either way, with the result shown underneath, e.g. "Allowed (from role)".
- **Guests** never can.

Each user's row also lists every link they've made, so an admin can revoke any of them.

## Things to know

- **Your server has to be reachable.** VTTs and OTFBM fetch the file from their own servers, so Grimoire must be reachable from the internet at the address in `BASE_URL`. If an authentication proxy sits in front of Grimoire (Authelia, Authentik, Cloudflare Access), let `/share/` and `/api/share/` through it.
- **Links are capabilities.** Anyone with the URL can use it, and URLs get forwarded. Use an expiry for anything you wouldn't want passed around.
- **You can copy a link again later** — the token is kept, not hashed. Revoke a link you think has leaked.
- **Guessing is throttled.** Requests for links that don't exist count against the login rate limit. Revoked or expired links don't count, so a tool that keeps fetching a dead link can't lock out the working ones.
