# Vendor and Integration Offboarding

Use this process when a client's contract with a vendor is ending and that vendor's access needs to be fully terminated. Deactivating a single login is not enough, every layer of access needs to be addressed independently, or you risk leaving a live pathway open after the relationship has ended.

## Steps to Take

1. **Deactivate the vendor's user record(s) in your CRM.** If there are multiple contacts at the vendor organization, deactivate each individually. This immediately blocks login and stops the account from participating in active sessions or syncs.

2. **Revoke all active OAuth tokens tied to that user, don't stop at deactivation.** OAuth access and refresh tokens exist independently of login state. A refresh token can silently re-authenticate a connection even after the user record is deactivated, so explicit revocation is required to actually close the pathway.

3. **Disable or restrict the vendor's external app/connected app configuration.** Before disabling it outright, confirm it isn't shared with other active integrations or vendors, check with your primary contacts if you're unsure. Disabling a shared app takes down more than the one relationship you intended to close.

4. **Determine whether the vendor had SSO/federated identity access, and confirm with the client whether that should be deactivated too.**
   - If yes, submit a request to your identity/platform team including: vendor name, client organization name, the app name or client ID (if known), and the requested deactivation date.
   - If the vendor's SSO app is shared platform-wide across multiple clients (common for things like payment processors, financial platforms, or other cross-client integrations), it stays active, note the reasoning in your internal tracking rather than disabling it unilaterally.
   - This matters because a federated identity layer can keep authenticating a vendor's systems or passing identity data even after your CRM access has been revoked, if it isn't closed independently.

5. **Remove any portal links, navigation items, tiles, or embedded elements pointing to the vendor's platform** (an SSO launch button, embedded iframe, LMS link, etc.), and verify no bookmarked deep links still route to the terminated integration. Confirm with the client's primary contact that the portal looks correct before closing out.

6. **Update the vendor's account record in your internal admin system.** If this client was the vendor's only relationship with you, revoke their access flag entirely. If you don't have sufficient permissions to do this yourself, route it to whoever does.

7. **End-date every relationship record tied to the vendor's service, regardless of how many clients they served**, effective as of the day before deactivation.

## Final Validation Checklist

- Vendor user record(s) deactivated
- OAuth sessions revoked on all vendor user records
- External app/connected app disabled or restricted
- SSO/identity integration status confirmed with client
- Deactivation request submitted for shared identity app, if applicable
- All vendor references removed from the client portal
- Client confirmed portal changes look correct
- Vendor account record updated and all relationships end-dated
- Internal records updated with deactivation dates and actions taken

## Notes / Best Practices

- Complete every layer, don't stop at user deactivation. It's a necessary first step, but by itself it doesn't stop OAuth token activity, disconnect the external app, or close identity access.
- Timing matters. Coordinate with the client to align offboarding with the actual contract end date. Too early disrupts active users; too late risks continued, unauthorized data flow.
- Communicate proactively with the client. Confirm what the vendor had access to and walk them through what's being shut down, this avoids surprises and keeps the client's timeline accounted for.
- If you're unsure whether an app or identity integration is shared with other active relationships, don't disable it unilaterally. Escalate first.
