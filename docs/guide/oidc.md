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

## Pinning settings via environment

Any OIDC setting can be pinned via an environment variable (e.g. `OIDC_ENABLED=true`, `OIDC_ISSUER_URL=...`). Pinned fields are shown read-only in the admin UI. See [Environment Variables](/configuration/env-vars) for the full list.

## Notes

- First-run setup always uses username + password. The OIDC button appears only after the IdP is fully configured.
- Logging out suppresses the next auto-launch so you don't immediately bounce back to the IdP.
- The login button text is configurable (e.g. "Sign in with Acme SSO") via the `OIDC_BUTTON_TEXT` env var or in-app setting.

## Provider-specific guides

- [Authentik Setup](/guide/oidc-authentik)
