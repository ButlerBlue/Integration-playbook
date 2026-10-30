# Runbook: Intermittent Authentication Failures Isolated to One Org

*What to do when an integration that runs across many client orgs starts failing to authenticate in just one of them, on and off, while every other org on the same setup is fine.*

## The Symptom

A vendor reports authentication errors for one client: a few in one day, or dozens over two. The response is usually some form of:

```json
{"error": "invalid_grant", "error_description": "authentication failure"}
```

Between failures, the same connection succeeds. Nothing has changed that anyone knows of. The same vendor connects to many other orgs in exactly the same way, and none of them are failing.

Two things make this case easy to get wrong:

1. **`invalid_grant` is a catch-all.** It can mean a wrong password or security token, an expired password, a locked or frozen user, a login IP or login hours restriction, a user who isn't permitted to use the app, or a problem with the app record itself. The error rarely says which.
2. **Intermittent failures look like noise.** A single failure that clears on retry gets written off as a one-off, especially when login history looks clean at the time. The second report is usually the first one anyone takes seriously.

## Why the Isolation Matters

Before you change anything, decide whether this is one org's problem or the leading edge of a fleet-wide one. That decision sets the whole response.

- **Fleet-wide** (several orgs on the same pattern failing): suspect a platform change, such as an OAuth enforcement date, a retired sign-in flow, or a change on the vendor's side. This becomes a program, not a ticket. See [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](13-platform-oauth-enforcement-fleet-migration.md).
- **One org only**, with everything else on the same setup healthy: the cause is almost always in that org's integration user, its app record, or its security settings. Treat it as a local diagnosis.

Getting this wrong is expensive in both directions. Treating a local problem as a platform change sends the vendor chasing a code fix they don't need. Treating the first sign of a platform change as a local problem means the other orgs fail next, without warning.

## Diagnose

1. **Get exact timestamps and the exact error body.** Ask the vendor for several failure times in UTC and the full response, not a summary. You can't line up a description against login history.

2. **Check for earlier signals.** Search your tickets and channels for the same client and the same vendor. An earlier one-off that was closed as "clean at the time" is part of the pattern, not a separate event.

3. **Confirm the isolation.** Compare against the other orgs on the same vendor and sign-in pattern over the same window. If you run the comparison across many orgs at once, note that in Salesforce the `Application` field on `LoginHistory` can be selected and grouped but **can't be filtered in a `WHERE` clause**, so filter on login type and date, then group by application and pick out the vendor's rows:

   ```sql
   SELECT Application, LoginSubType, Status,
          COUNT(Id) NumLogins, COUNT_DISTINCT(UserId) NumUsers,
          MAX(LoginTime) LastLoginTime
   FROM LoginHistory
   WHERE LoginTime = LAST_N_DAYS:7
   GROUP BY Application, LoginSubType, Status
   ORDER BY Application
   ```

   Test any query like this in one org yourself before handing it to someone else to run across the fleet. It saves a round trip and keeps the request credible.

4. **Read the affected org's login history at the reported times.** Filter to the integration user and the failure window, and look at the `Status` value on each failed attempt. It is often more specific than the error the vendor sees.

   ```sql
   SELECT LoginTime, Status, Application, SourceIp, LoginType
   FROM LoginHistory
   WHERE UserId = '[Integration User Id]'
   AND LoginTime >= [Start, UTC] AND LoginTime <= [End, UTC]
   ORDER BY LoginTime DESC
   ```

   If the failures don't appear at all, the request may not be reaching this org, or it may be reaching a different user. Confirm the username and the login endpoint the vendor is using.

5. **Check the integration user.** In order of how often each turns out to be the cause:
   - Active, not frozen, not locked out.
   - Password expiration: an integration user on a profile whose password policy expires passwords will start failing on the expiry date. Integration users should be on a profile or policy that doesn't expire passwords, or on a sign-in flow that doesn't use one.
   - Security token: reset recently by anyone? A reset invalidates the old token immediately.
   - Login IP ranges and login hours on the profile, compared against the source IPs in login history.

6. **Check the app record.** Open the external app the vendor signs in through and confirm its OAuth policies are actually configured: which users are permitted, IP relaxation, and token policies. An app record created in a hurry, or migrated from an older app type, can be missing its policy configuration entirely. Confirm the integration user's profile or permission set is assigned to the app.

7. **Write down what you confirmed and what you are inferring.** "Login history shows `Invalid Password` at three of five reported times" is a finding. "Probably the token" is a theory.

## Fix

1. **Change one thing at a time and note the time you changed it,** in UTC. If you reset a password and edit the app policies in the same minute, you won't know which one fixed it, and the next person won't either.
2. **Deliver any new credential over your secure credential-exchange channel only.** Never email, chat, or a ticket.
3. **Do it in a sandbox first when the change touches shared configuration.** A policy change on an app used by more than one integration affects all of them.
4. **Watch login history for the integration user** for an hour after the change. A clean run of successful logins at the vendor's normal interval is a good early sign, not proof.

## Close It Out

1. **Ask the vendor to watch their side for a defined period,** for example a day or two of normal runs, and to report any recurrence with timestamps. Intermittent problems need a longer quiet period before they are resolved.
2. **Close it only after that period passes cleanly,** and record the root cause in one paragraph on the ticket.
3. **Make this org a first candidate to move off password-based sign-in.** If the integration still uses username, password, and security token, a flaky connection is the best argument for migrating it first to Client Credentials or JWT Bearer, which remove the password dependency entirely. See [Configuring an API-Only Integration User](04-api-only-user-setup.md).
4. **Check the rest of the fleet for the same gap.** If the cause was a missing app policy or a password-expiry setting, query the other orgs for the same configuration before it surfaces as the next ticket.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| First failure closed as a one-off | Pattern noticed only after dozens of failures | Diagnose, step 2 |
| Local problem treated as a platform change | Vendor chases a code fix they don't need | Diagnose, step 3 |
| Platform change treated as a local problem | Other orgs fail next, without warning | Diagnose, step 3 |
| Fleet query handed off untested | Round trip lost to a query error | Diagnose, step 3 |
| Several changes made at once | Nobody knows what fixed it, or what to check next time | Fix, step 1 |
| New credential sent by email | Credential exposure on top of the original issue | Fix, step 2 |
| Closed after one clean hour | Recurrence reopens the ticket and costs credibility | Close It Out |
| Same gap left in other orgs | The next ticket is the same ticket | Close It Out, step 4 |

Related: [Identity Keys vs. Record Keys](09-identity-vs-record-keys.md) for confirming the vendor is signing in as the user you think, and [Partner Communication Templates](08-communication-templates.md) for the OAuth troubleshooting message.
