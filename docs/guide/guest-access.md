# Guest Access

Guests let you share a single campaign with people who don't have full accounts, for example a player who's only joining one game. A guest is a **code-only account**: no password, no OIDC, and no access to the library, maps, tokens, audio, or search.

A guest can only see the campaign they were invited to (and its shared resources, wiki, and schedule), and can edit only their **own** character name, character art, character sheet, session notes, and availability.

## Enabling guest access

Guest access is **off by default**. Turn it on either way:

- **In-app**: **Settings → Authentication → Guest Access**.
- **Pinned via env**: set [`GUEST_ACCESS_ENABLED`](/configuration/env-vars#authentication) to `true` or `false`. When set, it overrides the in-app toggle (shown read-only).

## Inviting a guest

From a GM campaign (admins and GMs only):

1. Open the members roster and use **Guests**.
2. Add a guest with a nickname. Each guest gets a unique **10-character invite code**. A campaign can have multiple guests.
3. Share the code with the built-in **Share** button: copy a ready-made message, copy a version for a Discord DM, or open a pre-filled email. The message includes a deep link (`/guest?code=…`) and the code itself.

## Managing guests

- **Regenerate** a guest's code, invalidating the old one.
- **Remove** the guest entirely, which deletes their guest account and contributions.

A guest's session ends once guest access is turned off or the guest is removed, and deleting a campaign removes its guest accounts too. Invite codes can only be managed from a signed-in session, never with an [API key](/api#api-keys).

## Logging in as a guest

Guests log in from the login screen via **Have an invite code?**. This works even on OIDC-only servers where password login is disabled. In the app a guest sees the nickname their GM gave them and a **GUEST** role.

## Admin overview

**Settings → Users** lists every guest account (grouped separately from full users) with its nickname, the campaign it's attached to, and who invited it. From there an admin can **convert a guest to a permanent user**: give it a username (and a password when password auth is enabled) and it keeps its campaign membership and character. The password must meet the [password rules](/configuration/security#password-rules).
