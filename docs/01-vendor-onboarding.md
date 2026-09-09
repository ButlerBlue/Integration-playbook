# Third-Party Vendor Access Request

Use this process when a client selects a vendor that does not have a formal partner relationship with your organization, and that vendor needs access to developer resources for an integration.

## Steps to Take

1. **Confirm this is not a partner integration.** If the vendor already has an approved partner agreement, use your partner integration process instead. This process is only for vendors a client selects independently, outside any existing partner relationship.

2. **Obtain written client sign-off before granting any access.** Because the vendor has no standing agreement with you, explicit approval from the client organization is required first.
   - Route the request to the client's primary point of contact for the integration (typically confirmed through the client's account manager).
   - Approval must come from the client, not the vendor.
   - Use a data release/consent template that captures: client organization name, vendor name, purpose of the integration, and your organization's standard signature block.

3. **Retain a copy of the approval for your records.** Save it in your internal system of record under the client's folder. This is your legal and audit trail for the authorization.

4. **Require the vendor to register for an account with your identity provider** before granting any access. If they already have an account, confirm their identity/ID against your internal record of registered users, and verify the email matches the person requesting access.

5. **Create the vendor's account and contact records in your CRM/admin system:**
   - Vendor account: record type "Vendor," name, website, partner status set to "None," and a flag enabling access to developer resources.
   - Vendor contact(s): name, linked account, email (should match their identity-provider account), and any secure credential-exchange handle you use (e.g., Keybase).

6. **Create the access/role records linking the vendor to your organization**, then a second role record linking the vendor to the specific client they'll be supporting. Mark both active.

7. **Allow time for provisioning to sync** (typically 5-10 minutes) before confirming access with the vendor.

## Final Validation Checklist

- Client approval email received and documented
- Approval record saved to internal system
- Vendor identity-provider account created
- Vendor account record created
- Vendor contact record(s) created
- Access role records created and active
- Developer resources flag enabled
- Vendor notified access is ready

## Notes / Best Practices

- Never grant access without documented client approval on file.
- Scope vendor access to only what's necessary for the approved client relationship.
- Repeat the contact and role steps for each additional vendor user needing access.
- If access issues persist after provisioning time has passed, check identity match and role/permission associations first before escalating.
