# OpenID Connect

Grimoire supports authentication via any OpenID Connect–compliant identity provider (Keycloak, Authentik, Authelia, Auth0, Okta, etc.).

## Configure

Open **Settings → Sign-in** as an admin:

1. Tick **Enable OpenID Connect**. The fields below only appear once it is on; the login page shows the button once the rest is filled in.
2. Set the **Issuer URL** and click **Autopopulate**: Grimoire fetches the IdP's `.well-known/openid-configuration` and fills in the endpoint URLs. You can also paste the full discovery document URL directly.
3. Paste the **Client ID** and **Client Secret** from your IdP. The secret is stored encrypted under `SECRET_KEY`, so a backup doesn't carry it; if `SECRET_KEY` changes you'll need to enter it again. Setting `OIDC_CLIENT_SECRET` in the environment instead keeps it out of the database and is recommended.
4. Register the displayed **Redirect URI** with your IdP. Set `BASE_URL` so the host reflects your public origin:
   ```
   https://<your.server.com>/api/auth/openid/callback
   ```

## Optional settings

| Setting | Description |
|---|---|
| **Token Issuer** | The exact `iss` value your IdP puts in tokens. Leave blank to auto-detect. Set explicitly if Authentik's application provider issuer differs from the Issuer URL. |
| **Groups Claim** | Name of the OIDC claim containing group memberships. When set, roles are assigned from the **Admin group**, **GM group** and **Player group** below - by default groups named `admin`, `gm` and `player`. Highest level wins, and the role is synced on every sign-in. Users without a matching group are denied. |
| **Admin group / GM group / Player group** | The group that grants each role, if your provider names them differently (e.g. `grimoire-admin`). A group counts only when its name matches exactly, ignoring case; Keycloak's full-path form of a top-level group (`/grimoire-admin`) also counts, but a subgroup such as `/media/grimoire-admin` does not. A group that merely ends in a role name, such as `jellyfin-admin`, grants nothing. Leave blank for the default. |
| **Advanced Permissions Claim** | Name of the claim containing a permissions object, e.g. `{viewNSFW: bool, campaignAccess: bool, apiKeys: bool}`. When set, must be present in every login or access is denied. See [Permissions claim](#permissions-claim). |
| **Match Existing Users By** | Link an existing local account to the OIDC subject by `email` or `username` on first login. Later logins always match by the stable subject claim. Only accounts that aren't linked to a provider yet are considered, and campaign guest accounts never are. See [Matching existing users](#matching-existing-users). |
| **Auto-launch** | Automatically redirect to the IdP when visiting `/login`. Suppress with `?autoLaunch=0`. |
| **Auto-register** | Automatically create local accounts on first OIDC login. |

### Matching existing users

- **Email:** the provider's `email` claim is matched against the account's email. A provider that reports `email_verified: false` is not matched. The address and its `email_verified` flag come together from the ID token, or from userinfo when the ID token has no email, and userinfo is ignored when its `sub` isn't the ID token's. While this mode is on, users can't change their own email in Settings; an admin sets it, so the address keeps identifying the right person.
- **Username:** the provider's `preferred_username` claim is matched against the Grimoire username, ignoring case. Display names (`name`, `nickname`) are only used to name a newly registered account, never to match one. Admin accounts are never matched by username; link an admin by email, or let them keep signing in with their password.

::: warning
Only match by username with a provider where users cannot choose or change their own username - self-registration off, usernames set by an administrator. Otherwise anyone can register the name of an existing Grimoire account at the provider and be signed in as it.
:::

### Permissions claim

The permissions object is read on every login and each key it contains overwrites that setting on the account:

| Key | What it controls | New account without the key |
|---|---|---|
| `viewNSFW` | Explicit content. `false` also locks it off, so the user can't turn it on in their own settings. | Off and locked |
| `campaignAccess` | Creating and editing [campaigns](/guide/campaigns#per-user-campaign-access). | Enabled |
| `apiKeys` | Lets a non-admin create and use [API keys](/api#api-keys). Admins always have them. | Disabled |

A key that is **missing** from the object only matters when the account is created, which gives it the default above. After that, a missing key changes nothing: the account keeps its current value. So explicit content is opt-in for new users, but removing `viewNSFW` from a user's claim later doesn't take back access an earlier login granted.

::: tip
Send every key you care about on every login, with an explicit `true` or `false`. To take explicit content away from an existing user, the claim must say `"viewNSFW": false` - leaving the key out keeps whatever they had.
:::

## Style the sign-in button

Many providers ask apps to use their standard "Sign in with …" button, and a familiar button is reassuring on a login page your users haven't seen before. Grimoire doesn't ship any provider's buttons or logos. Instead you style the one OIDC button yourself, so any provider can be matched without Grimoire having to know about it.

Under **Settings → Sign-in → Button Appearance**:

| Setting | Description |
|---|---|
| **Button Text** | What the button says, e.g. "Sign in with Acme SSO". Empty uses the default. |
| **Background / Text / Border Color** | CSS hex values (`#rgb`, `#rgba`, `#rrggbb` or `#rrggbbaa`). Leave empty to keep the theme's look. |
| **Corner Radius** | 0 to 40 px. Empty keeps the default (8). |
| **Icon** | The provider's logo: PNG, JPEG, WebP, GIF or SVG, up to 1 MB. Drawn at 20 px beside the button text. |

A live preview updates as you type.

### Examples

Take the logo and colors from the provider's own brand page, because those change over time and staying within a provider's branding rules is up to you.

| Look | Background | Text | Border | Radius | Logo |
|---|---|---|---|---|---|
| Google, dark | `#131314` | `#E3E3E3` | `#8E918F` | `4` or `20` (pill) | the "G" from [Google's branding guidelines](https://developers.google.com/identity/branding-guidelines) |
| Google, light | `#FFFFFF` | `#1F1F1F` | `#747775` | `4` or `20` (pill) | as above |
| Discord | `#5865F2` | `#FFFFFF` | `#5865F2` | `8` | the white Clyde mark from [Discord's branding page](https://discord.com/branding) |
| GitHub | `#24292F` | `#FFFFFF` | `#24292F` | `6` | the white Invertocat from [GitHub's logos page](https://github.com/logos) |

Google's guidelines also call for Google Sans Medium at 14 px. Grimoire keeps the button in the app's own font, so the result is a close match rather than an exact one.

::: tip GitHub and Discord need a broker
GitHub and Discord sign-in is plain OAuth 2.0, not OpenID Connect, so Grimoire can't talk to them directly. Put a broker such as [Authentik](/guide/oidc-authentik) or Keycloak in front of them and point Grimoire at the broker. If the broker sends everyone straight on to one upstream provider, style the button for that provider.
:::

Uploaded icons are never served the way they were uploaded. Raster images are re-encoded and SVGs are rendered to pixels, so the stored icon is always a small PNG (in `DATA_PATH/branding/`) and an SVG can't carry a script onto your public login page. Colors and the radius are checked against a strict format, so no setting can inject arbitrary CSS.

## Pinning settings via environment

Any OIDC setting can be pinned via an environment variable (e.g. `OIDC_ENABLED=true`, `OIDC_ISSUER_URL=...`). Pinned fields are shown read-only in the admin UI. See [Environment Variables](/configuration/env-vars) for the full list.

## Notes

- First-run setup always uses username + password. The OIDC button appears only after the IdP is fully configured.
- **Password Authentication** in **Settings → Sign-in** can only be turned off once OpenID Connect is configured, and OpenID Connect can't be turned off while password sign-in is off - so there is always a way in. `ALLOW_PASSWORD_AUTHENTICATION` pins it regardless.
- Logging out suppresses the next auto-launch so you don't immediately bounce back to the IdP.
- A sign-in has to finish in the browser that started it, within 10 minutes. If the provider sends you back to Grimoire in a different browser or app (some mobile mail apps open links in their own browser), you'll see "sign-in was started in another browser"; start again from the login page there.
- The login button text is configurable (e.g. "Sign in with Acme SSO") via the `OIDC_BUTTON_TEXT` env var or in-app setting, and so is its look - see [Style the sign-in button](#style-the-sign-in-button).

## Provider-specific guides

- [Authentik Setup](/guide/oidc-authentik)
- [Pocket ID Setup](/guide/oidc-pocketid)
