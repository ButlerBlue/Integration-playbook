# Configuring a Salesforce API-Only Integration User

*A step-by-step reference for setting up the "Salesforce Integration" license type for a third-party integrator. Pairs with the [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md), that page covers the why and the security posture, this one covers the concrete setup steps.*

## Summary: Pros and Cons

This license exists for system-to-system integrations only, no UI login of any kind, access strictly through the API (REST, SOAP, Bulk). Use it to give each integrator its own least-privilege user without consuming a full user license.

**Pros:**
- Cost: most paid Salesforce editions include a handful of free integration licenses; additional ones are inexpensive relative to a full license (confirm current pricing, Salesforce updates this periodically)
- One user per integration keeps permissions minimal and every transaction attributable to a specific integrator
- No UI login means these users sit outside interactive-login MFA enforcement, since that applies to UI logins specifically
- Frees full licenses for human users

**Cons:**
- Zero UI access, ever, if the integrator ever needs occasional UI access, this license type won't work
- Permissions are gated by a specific permission set license, and some permissions (e.g., broad data-modification or log-file access) can't be granted at all on this license type
- Easy-to-miss setup dependency: the permission set license has to be assigned to the user *before* the permission set will actually work
- **Known platform limitation:** on this license type, converting a lead to a contact does not reliably set the record type on the resulting contact. If your CRM's data model depends on record types for contacts (common in managed packages), don't assign this license to any integrator whose workflow includes lead conversion, that integration needs a different license type.

## External App Setup

Modern Salesforce orgs create an External Client App rather than a legacy Connected App (new Connected App creation is now blocked by default on current platform versions). External Client Apps only support modern OAuth flows and default PKCE to on.

1. Setup > External Client App Manager > New External Client App.
2. Fill in the app name, API name, contact email, and a description identifying the integrator and what it connects to.
3. Enable OAuth, set the callback URL to the integrator's actual redirect URI (never a placeholder or your own domain).
4. Scope OAuth permissions to the minimum needed (basic API access, plus refresh token support only if the agreed flow actually uses one).
5. Require a secret for the web server flow, and save.
6. After saving, set: permitted users to admin-approved only, IP restrictions enforced if the integrator has stable egress IPs, a defined refresh token expiration and session timeout, and leave PKCE enabled (it only affects authorization-code-style flows and costs nothing otherwise).
7. Copy the resulting client ID and secret and deliver them to the integrator over a secure credential-exchange channel, never email or chat.

## Profile

1. Clone your platform's most restrictive integration-user profile template (avoid any deprecated predecessor profile still present in older orgs, check for known over-grant issues before reusing one).
2. Assign the "integration" user license type and save.
3. Grant access only to the specific apps the integrator actually needs (your organization's custom apps, plus any managed package apps in scope).
4. Assign the integrator's External App under assigned external apps.
5. Grant minimal object/field access here; anything more substantial belongs in a permission set instead.
6. Confirm API-only and API-enabled system permissions are set, and that interactive MFA requirements aren't mistakenly applied to this API-only profile.
7. Set session timeout, a non-expiring password policy (the password is never actually used for API-only auth), and optionally restrict login IP ranges to the integrator's published ranges as a hardening measure.

## Permission Set

1. Create a permission set using the correct API-integration license type, this is the step most commonly missed; a permission set on the wrong license type simply won't assign.
2. Grant the real object and field access here. Keep the profile minimal and put the integrator's actual scope in the permission set, where it's easier to audit and revoke later.

## User

1. Create the user with the integrator's designated support/dev email, the integration license type, and the profile created above. Save.
2. This sends a password-setup invitation. The integrator should open it in an incognito/private window, since this user has no UI access, they'll see an access-restricted message after setting their password, which is expected.

## Assign the Permission Set License

The user will not function without this step. On the user record, assign the permission set license first, only then will the permission set itself be assignable. Then assign the actual permission set from the previous section.

## Security Token (Legacy Fallback Only)

Skip this unless the integrator's own tooling specifically requires legacy username/password-plus-security-token authentication, standard OAuth (Client Credentials or JWT Bearer) doesn't need it. This authentication method is being phased out platform-wide, so confirm the integrator has a migration plan if they're relying on it. If a token genuinely is required, generate it through the platform's standard token-reset flow and deliver it the same way as any other credential.

## Test the User

Verify permissions with representative queries before considering the integrator ready. If the integrator attempts a UI login, they should see an access-restricted message, that's expected behavior for this license type, not an error.

## Notes

For the platform vendor's own integration-user configuration guidance, see Salesforce's official "Best Practices for Configuring Your Integration User."
