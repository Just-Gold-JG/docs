# Portal Access

B2B partner users can access the JustGold partner portal in two ways:

- Create portal credentials
- Sign in with OAuth2 / OpenID Connect

Portal access is used to manage organizations, API credentials, users, webhooks, and reports. It is separate from API authentication. API calls still use `X-Client-Id`, `X-Timestamp`, and `X-Signature`.

## Create Portal Credentials

Use this option when partner users will sign in directly with a portal email address and password.

### How It Works

1. A partner admin or JustGold onboarding user invites the partner user to the portal.
2. The invited user receives an email invite.
3. The user opens the invite link and creates a password.
4. If MFA is required, the user scans the authenticator QR code and verifies the 6-digit code.
5. The user signs in to the portal with email, password, and MFA when prompted.

### User Requirements

| Item | Requirement |
| --- | --- |
| Email | Must be a valid email address. |
| Password | Minimum 8 characters, with uppercase, lowercase, number, and special character. |
| MFA code | 6-digit authenticator app code when MFA is enabled. |

### When To Use

Use portal credentials when:

- the partner does not use a central identity provider
- users should be managed directly in the JustGold portal
- onboarding needs to be fast and limited to a few users

## OAuth2 / OpenID Connect

Use this option when partner users should access the portal through the partner's identity provider.

OAuth2 / OpenID Connect allows partner users to sign in with their existing corporate identity. JustGold uses the identity provider to authenticate the user and then grants portal access based on the configured organization and role mapping.

### How It Works

1. The partner shares OpenID Connect configuration with JustGold during onboarding.
2. JustGold configures the partner identity provider for portal sign-in.
3. The user selects the configured identity provider on the portal sign-in page.
4. The user authenticates with the partner identity provider.
5. The identity provider returns the user identity to JustGold.
6. JustGold creates or matches the portal user and applies the configured role and organization access.

### Configuration Details

Share the following details with JustGold when enabling OpenID Connect:

| Item | Description |
| --- | --- |
| Issuer URL | OpenID Connect issuer URL for the partner identity provider. |
| Discovery URL | `.well-known/openid-configuration` URL, if different from the issuer URL. |
| Client ID | OAuth2 client ID created for JustGold portal sign-in. |
| Client secret | OAuth2 client secret, shared securely. |
| Allowed domains | Email domains allowed to use this identity provider. |
| Claims | Claims used to identify the user, usually `sub`, `email`, and `name`. |
| Role mapping | Mapping from identity provider groups or claims to JustGold portal roles. |
| Redirect URI | Redirect URI provided by JustGold and configured in the partner identity provider. |

### Recommended Scopes

| Scope | Purpose |
| --- | --- |
| `openid` | Required for OpenID Connect authentication. |
| `email` | Provides the user's email address. |
| `profile` | Provides basic user profile information. |

### When To Use

Use OAuth2 / OpenID Connect when:

- the partner requires SSO
- users are managed in a corporate identity provider
- access should follow centralized joiner, mover, and leaver processes
- the partner wants identity provider policies such as conditional access or enforced MFA

## Roles And Access

Portal permissions depend on the role assigned to the user.

| Role | Typical Access |
| --- | --- |
| Developer | View own organization and API credentials. |
| Admin | Manage organization users, organization details, credentials, reports, and webhook settings. |

Exact access can vary by partner setup. Confirm the required roles with the JustGold onboarding team before production access is enabled.

## Important Notes

- Portal sign-in is not used to authenticate API requests.
- API credentials are generated and managed inside the portal after the user has portal access.
- OAuth2 / OpenID Connect must be configured by JustGold before partner users can use it.
- If a user loses access to the partner identity provider, their portal SSO access should also be removed according to the partner's identity policy.
