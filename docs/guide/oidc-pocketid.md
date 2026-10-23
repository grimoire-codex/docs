# Pocket ID Setup

A minimal setup for [Pocket ID](https://pocket-id.org) that maps user groups to Grimoire roles and controls explicit content access.

## 1. Create the groups

In Pocket ID, go to **User Groups** and create these groups:

| Group | Purpose |
|---|---|
| `grimoire-admin` | Full admin access |
| `grimoire-gm` | GM role |
| `grimoire-player` | Player role |

Pocket ID sends group names in the `groups` claim. You tell Grimoire these names in step 4, so no scope mapping is needed. (Grimoire matches group names exactly, so a group from another app that happens to end in `-admin` grants nothing here.)

Assign your users to the appropriate groups.

## 2. Add the permissions claim

Grimoire reads explicit content access (`viewNSFW`), campaign access (`campaignAccess`) and API key access (`apiKeys`) from a `permissions` claim. Admins always have API keys. Pocket ID has no scripted claims, so the claim is set as a **custom claim** (open the group or user and use **Custom Claims**).

Pocket ID doesn't merge custom claims. The `permissions` value is one JSON object, and a user gets exactly one of them:

- A claim set on the **user** always wins over their groups' claims.
- If several of a user's **groups** set `permissions`, only one group's object is sent, and Pocket ID doesn't define which. A `{"viewNSFW": true}` on one group and a `{"viewNSFW": false}` on another don't combine; the user gets one or the other.

So give each role group a complete default, and make exceptions per user rather than with extra groups:

| Group | Key | Value |
|---|---|---|
| `grimoire-admin` | `permissions` | `{"viewNSFW": true, "campaignAccess": true, "apiKeys": true}` |
| `grimoire-gm` | `permissions` | `{"viewNSFW": false, "campaignAccess": true, "apiKeys": false}` |
| `grimoire-player` | `permissions` | `{"viewNSFW": false, "campaignAccess": true, "apiKeys": false}` |

To let one player see explicit content or create API keys, open that **user** and add a `permissions` custom claim with the full object, e.g. `{"viewNSFW": true, "campaignAccess": true, "apiKeys": true}`. Delete it to put them back on their group's default.

Spell out every key in each object. A missing key only sets the default on a new account (explicit content off, campaign access on, API keys off). After that it leaves the account as it was, so deleting a user's override doesn't take back what it granted unless their group's object says `false`. See [Permissions claim](/guide/oidc#permissions-claim).

::: warning
Grimoire denies login if the permissions claim is missing, so every user must belong to a role group that sets it (or carry it on their own account). The claim must arrive as a JSON object, not a string.
:::

## 3. Create the OIDC client

Go to **OIDC Clients** and click **Add OIDC Client**.

| Field | Value |
|---|---|
| **Name** | `Grimoire` |
| **Client Launch URL** | `https://<your.server.com>` |
| **Callback URLs** | `https://<your.server.com>/api/auth/openid/callback` |
| **Logout Callback URLs** | `https://<your.server.com>/login` |
| **PKCE** | On |

Set `BASE_URL` in Grimoire so the host reflects your public origin.

Save the client, then note the **Client ID** and **Client Secret**.

## 4. Configure Grimoire

In **Settings → Sign-in**:

1. Tick **Enable OpenID Connect** - the rest of the fields appear once it is on.
2. Set **Issuer URL** to your Pocket ID address (e.g. `https://id.example.com`) and click **Autopopulate**.
3. Paste your **Client ID** and **Client Secret**.
4. Confirm the displayed **Redirect URI** matches the Callback URL you set in Pocket ID.
5. Set **Groups Claim** to `groups`, then set **Admin group** to `grimoire-admin`, **GM group** to `grimoire-gm` and **Player group** to `grimoire-player`.
6. Set **Advanced Permissions Claim** to `permissions`.
7. Enable **Auto-register** if you want accounts created automatically on first login.

::: warning
With **Groups Claim** set, users who are not in the `grimoire-admin`, `grimoire-gm`, or `grimoire-player` group are denied.
:::
