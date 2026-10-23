# Authentik Setup

A complete setup for Authentik that maps groups to Grimoire roles and controls explicit content access.

## 1. Create the groups

In Authentik, create these groups:

| Group | Purpose |
|---|---|
| `grimoire-admin` | Full admin access |
| `grimoire-gm` | GM role |
| `grimoire-player` | Player role |
| `nsfw` | Grants explicit content access to non-admin users |

Assign your users to the appropriate groups.

## 2. Create custom scopes

Go to **Customization → Property Mappings** and create two **Scope Mappings**.

### Grimoire Groups scope

**Name:** `Grimoire Groups`  
**Scope:** `groups`

```python
groups = [group.name for group in user.ak_groups.all()]

grimoire_groups = []
if "grimoire-admin" in groups:
    grimoire_groups.append("admin")
if "grimoire-gm" in groups:
    grimoire_groups.append("gm")
if "grimoire-player" in groups:
    grimoire_groups.append("player")

return {"groups": grimoire_groups}
```

### Grimoire Permissions scope

**Name:** `Grimoire Permissions`  
**Scope:** `permissions`

```python
groups = [group.name for group in user.ak_groups.all()]

if "grimoire-admin" in groups:
    return {"permissions": {"viewNSFW": True}}

explicit = "nsfw" in groups

return {"permissions": {"viewNSFW": explicit}}
```

This sends `viewNSFW` for every user, so anyone outside `nsfw` gets an explicit `false`. Keep it that way if you edit the script: a missing `viewNSFW` only denies explicit content to a new account, and an existing account keeps whatever it had, so removing someone from `nsfw` wouldn't take their access away. See [Permissions claim](/guide/oidc#permissions-claim).

## 3. Configure the provider

In your Authentik OAuth2/OIDC provider:

1. Open (or create) your **OAuth2/OpenID Provider**.
2. Go to **Advanced Protocol Settings**.
3. Under **Scopes**, add both `Grimoire Groups` and `Grimoire Permissions`.
4. Save the provider.
5. Note your **Client ID** and **Client Secret**.

## 4. Configure Grimoire

In **Settings → Sign-in**:

1. Tick **Enable OpenID Connect** - the rest of the fields appear once it is on.
2. Set **Issuer URL** to your Authentik application's issuer (e.g. `https://authentik.example.com/application/o/<app-slug>/`) and click **Autopopulate**.

   ::: warning Token issuer mismatch
   Authentik's token issuer often differs from the provider URL. If login fails with an issuer mismatch, decode a token and copy the `iss` value into the **Token Issuer** field.
   :::

3. Paste your **Client ID** and **Client Secret**.
4. Register the displayed **Redirect URI** in your Authentik provider.
5. Set **Groups Claim** to `groups`.
6. Set **Advanced Permissions Claim** to `permissions`.
7. Enable **Auto-register** if you want accounts created automatically on first login.
