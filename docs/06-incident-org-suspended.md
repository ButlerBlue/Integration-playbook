# Incident Runbook: CRM Production Org Suspended or Disabled

*What to do when a client's CRM production environment is locked into a suspended or disabled state, most commonly a billing lock from the platform vendor, but not always.*

Check whether you can log into the client's production user. A disabled-org error screen means the environment has been suspended, and everything tied to it stops until access is restored.

Account Managers own all client-facing and cross-vendor communication during this kind of incident. Keeping internal coordination in a single channel avoids multiple people talking past each other about the same event.

## What Breaks During the Outage

- **Portal:** members can't sign in, and headquarters staff can't log into their own CRM users.
- **SSO:** every login attempt fails, since your SSO layer validates against the client's CRM org.
- **Integrations:** every partner integrator stops receiving traffic. Any middleware sync and any bi-directional database sync connector (e.g., Heroku Connect) stop moving data in both directions.
- **API:** all inbound and outbound API traffic is rejected.

## Why This Happens

The most common cause is a missed or late invoice payment with the CRM platform vendor. The platform locks the org for non-payment after billing notices go unanswered. The client should contact their platform Account Executive as soon as possible, they'll typically need their contract ID and/or invoice ID.

Two things to confirm before assuming it's billing:
- Most platform vendors send several notices to billing/admin contacts before hard-locking an org. Have the client check whether a notice already landed with whoever owns their platform billing or environment administration. Caught early enough, they may be able to pay before the lock takes effect.
- An org can also be disabled for reasons unrelated to payment, a lapsed contract at renewal, or a platform trust/security suspension. The remediation contact is the same (their Account Executive and the platform's support line), but the cause changes what the client needs to bring to that conversation.

Suspension locks the org, it does not immediately delete data. Access and data return once the client resolves the issue with the vendor. Prolonged non-payment can eventually put data at risk, so treat it as urgent and have the client confirm any deletion timeline directly with their Account Executive.

## Verify the Outage

Confirm before alerting anyone. Try all of the following and confirm each fails:
- Log into production.
- Log into the sandbox/dev environment.
- Log into the portal with a support/test SSO user.

If all three fail, notify your internal incident channel so there's a single source of truth, rather than several channels discussing the same event. Something like:

> ⚠️ Urgent, CRM is deactivated for [client org] ⚠️
> All access is down. Portal logins will fail, and SSO access to any integrator will fail as well.
>
> **Support teams:** for incoming requests, respond with a holding message acknowledging the issue is being worked and that you'll follow up with a resolution or timeline.
>
> **Partner services:** notify all partner integrations that their users can't receive traffic and that you're working with the client to resolve it, without a firm timeline yet.
>
> **DevOps/engineering:** put up a maintenance page for the portal immediately, since no user can log in.

Then check your internal record of the org's vendor/integration relationships to see who else needs notifying, and alert the headquarters point of contact directly, something like:

> ⚠️ Urgent, CRM is deactivated ⚠️
> We're seeing a disabled-org message when attempting to log in. Can you check with your team on whether there's an outstanding invoice with [platform vendor]?
>
> As you work toward resolution with your Account Executive, it's worth alerting your other vendors too. We've already notified [vendor/integrator placeholders] on our end, and we see you also work with [additional vendor placeholders].
>
> We've also put up a maintenance message on the portal in the meantime. Thanks for working through this with us.

Continue monitoring and post updates in the incident channel as the situation develops.

## When Access Is Restored

1. **Verify it's actually back** by logging into production and the dev/sandbox environment.
2. **Confirm integration plumbing is healthy before reopening the floodgates:**
   - Check any database sync connector; if its OAuth connection errored during the outage, it may need to be re-authenticated before it resumes syncing.
   - Confirm middleware sync traffic resumes. Don't modify sync/egress configuration, just verify data is moving again.
   - Redeploy dev, UAT, and production environments; if any fail to deploy, loop in engineering support.
3. **Notify the team:**

> ⚠️ CRM is back online for [client org] ⚠️
> All checks pass and portal deployments are successful.
>
> **Support teams:** let anyone who reported issues during the outage know they can try logging in again.
>
> **Partner services:** notify integrators that users are back up. If any resyncing is needed to backfill the outage window, ask them to wait until the next business day morning rather than resyncing immediately.
>
> **DevOps/engineering:** take down the maintenance page.

4. **Stagger the resyncs deliberately, this isn't just courtesy.** Most CRM platforms enforce a rolling daily API request limit per org. If every integrator backfills the outage window at once, they can exhaust that limit and lock out the headquarters' own legitimate usage, effectively causing a second outage. Prioritize headquarters and portal traffic first, then let integrators resync the following business day morning.
5. Remind the headquarters point of contact to let their own vendors know access has been restored and that the maintenance page has been removed.
6. Continue monitoring API usage through the day and keep the team updated in the incident channel.
