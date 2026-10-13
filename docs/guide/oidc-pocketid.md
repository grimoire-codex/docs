# Pocket ID Setup

A minimal setup for [Pocket ID](https://pocket-id.org) that maps user groups to Grimoire roles and controls explicit content access.

## 1. Create the groups

In Pocket ID, go to **User Groups** and create these groups:

| Group | Purpose |
|---|---|
| `grimoire-admin` | Full admin access |
| `grimoire-gm` | GM role |
| `grimoire-player` | Player role |
| `nsfw` | Grants explicit content access to non-admin users |
| `apikeys` | Lets non-admin users create [API keys](/api#api-keys) |

Pocket ID sends group names in the `groups` claim. Grimoire recognizes names that end in `-admin`, `-gm`, or `-player` (case-insensitive), so no scope mapping is needed.

Assign your users to the appropriate groups.

## 2. Add the permissions claim

Grimoire reads explicit content access (`viewNSFW`) and API key access (`apiKeys`) from a `permissions` claim. Admins always have API keys. Pocket ID has no scripted claims, so add a **custom claim** to each group (open the group and use **Custom Claims**):

| Group | Key | Value |
|---|---|---|
| `grimoire-admin` | `permissions` | `{"viewNSFW": true}` |
| `nsfw` | `permissions` | `{"viewNSFW": true}` |
| `apikeys` | `permissions` | `{"apiKeys": true}` |
| `grimoire-gm` | `permissions` | `{"viewNSFW": false}` |
| `grimoire-player` | `permissions` | `{"viewNSFW": false}` |

::: warning
Grimoire denies login if the permissions claim is missing, so every user must belong to a group that sets it. The claim must arrive as a JSON object, not a string.
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

In **Settings → Authentication**:

1. Set **Issuer URL** to your Pocket ID address (e.g. `https://id.example.com`) and click **Autopopulate**.
2. Paste your **Client ID** and **Client Secret**.
3. Confirm the displayed **Redirect URI** matches the Callback URL you set in Pocket ID.
4. Set **Groups Claim** to `groups`.
5. Set **Advanced Permissions Claim** to `permissions`.
6. Enable **Auto-register** if you want accounts created automatically on first login.
7. Enable **OpenID Connect**.

::: warning
With **Groups Claim** set, users who are not in the `grimoire-admin`, `grimoire-gm`, or `grimoire-player` group are denied.
:::
