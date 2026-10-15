# OpenID Connect

Grimoire supports authentication via any OpenID Connect–compliant identity provider (Keycloak, Authentik, Authelia, Auth0, Okta, etc.).

## Configure

Open **Settings → Authentication** as an admin:

1. Set the **Issuer URL** and click **Autopopulate**: Grimoire fetches the IdP's `.well-known/openid-configuration` and fills in the endpoint URLs. You can also paste the full discovery document URL directly.
2. Paste the **Client ID** and **Client Secret** from your IdP.
3. Register the displayed **Redirect URI** with your IdP. Set `BASE_URL` so the host reflects your public origin:
   ```
   https://<your.server.com>/api/auth/openid/callback
   ```
4. Enable **OpenID Connect**.

## Optional settings

| Setting | Description |
|---|---|
| **Token Issuer** | The exact `iss` value your IdP puts in tokens. Leave blank to auto-detect. Set explicitly if Authentik's application provider issuer differs from the Issuer URL. |
| **Groups Claim** | Name of the OIDC claim containing group memberships. When set, roles are assigned from groups named `admin`, `gm`, or `player` (case-insensitive). Users without a matching group are denied. |
| **Advanced Permissions Claim** | Name of the claim containing a permissions object, e.g. `{viewNSFW: bool, campaignAccess: bool, apiKeys: bool}`. `apiKeys` lets a non-admin create and use [API keys](/api#api-keys). When set, must be present in every login or access is denied. |
| **Match Existing Users By** | Link an existing local account to the OIDC subject by `email` or `username` on first login. |
| **Auto-launch** | Automatically redirect to the IdP when visiting `/login`. Suppress with `?autoLaunch=0`. |
| **Auto-register** | Automatically create local accounts on first OIDC login. |

## Style the sign-in button

Many providers ask apps to use their standard "Sign in with …" button, and a familiar button is reassuring on a login page your users haven't seen before. Grimoire doesn't ship any provider's buttons or logos. Instead you style the one OIDC button yourself, so any provider can be matched without Grimoire having to know about it.

Under **Settings → Authentication → Button Appearance**:

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
- Logging out suppresses the next auto-launch so you don't immediately bounce back to the IdP.
- The login button text is configurable (e.g. "Sign in with Acme SSO") via the `OIDC_BUTTON_TEXT` env var or in-app setting, and so is its look - see [Style the sign-in button](#style-the-sign-in-button).

## Provider-specific guides

- [Authentik Setup](/guide/oidc-authentik)
- [Pocket ID Setup](/guide/oidc-pocketid)
