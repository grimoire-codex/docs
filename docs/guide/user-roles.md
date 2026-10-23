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

## Explicit content

Each user account has an `allow_explicit` flag. Users with this disabled will not see game systems, books, or tokens marked as explicit. Admins can toggle this per-user; users can toggle it for their own account in **Settings → Preferences → Content Preferences**.

## Campaign access

Each user has a **campaign access** toggle (managed per user in **Settings → People**; enabled by default). Disabling it stops the user from creating campaigns, being added to new ones, and editing resources, and locks any campaign they own to read-only, but never deletes existing campaigns. When OIDC is configured this can be driven by the provider's permissions claim. See [Per-user campaign access](/guide/campaigns#per-user-campaign-access).

## Display names

Users can set a **display name** in **Settings → Account** that appears in place of their username across the app.
