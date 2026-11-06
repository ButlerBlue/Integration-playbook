# Vendor Change Authorization (Template)

*A fill-in record for approving a vendor's request to change configuration in a client org, or to refresh a sandbox, during a defect investigation. It captures who decided, what changed, and how to reverse it, so the answer to "why is this setting different?" is one link instead of a search through old threads.*

**How to use it:** fill in Part 1 from the vendor's request, and send back for anything missing before you route it. Fill in Part 2 when the decision is made, and Part 3 once the change is confirmed. Paste the finished record into the support case or defect ticket. The rules behind each field are in [Vendor Change Requests in a Client Org](../docs/19-vendor-change-requests-in-client-orgs.md).

## Part 1: The Request

| | |
|---|---|
| **Vendor and requester** | [Vendor name, requester name and role] |
| **Related case or ticket** | [Support case or defect ticket reference] |
| **Request category** | [Access to observe / Sandbox configuration change / Production configuration change / Code, package, or automation change / Sandbox refresh] |
| **Environment** | [Sandbox name and type, or Production] |
| **What exactly changes** | [Object, setting, and the profile or permission set it applies to] |
| **Current value** | [Exact value today, with a screenshot or query output attached] |
| **Proposed value** | [Exact value requested] |
| **Why the vendor believes it is related** | [One or two sentences in the vendor's words] |
| **Who and what else it affects** | [Other users, profiles, automations, or integrations that depend on this setting] |
| **Reversal plan** | [How it will be put back, and by whom] |

## Part 2: The Decision

| | |
|---|---|
| **Decided by** | [Client data owner's name and role] |
| **Decision** | [Approved / Approved for sandbox only / Declined / Deferred until sandbox refresh] |
| **Confirmed in a sandbox first?** | [Yes, in [sandbox name] on [date] / No, production exception (complete the block below)] |
| **Date and channel of approval** | [Date, and where the written approval lives] |
| **Conditions** | [Any limits, e.g., "revert if the next retry fails," "change applies to the integration profile only"] |

**Production exception (complete only if the change was not confirmed in a sandbox first):**

| Condition | Met? |
|---|---|
| Configuration setting only, not code, package, or automation | [Yes / No] |
| Part 1 complete, including the current value and reversal plan | [Yes / No] |
| Client data owner told in plain language that it goes to production untested, and why | [Yes / No] |
| Independent confirmation planned (Part 3) | [Yes / No] |
| Sandbox refresh or rebuild scheduled to reproduce the fix | [Yes, on [date] / No] |

If any answer is No, the change waits for a sandbox.

## Part 3: Confirmation

| | |
|---|---|
| **Change made by** | [Name, and date and time] |
| **Confirmed by** | [Someone other than the vendor] |
| **How it was confirmed** | [Setting checked directly, and the Setup Audit Trail entry showing who changed it and when] |
| **Result** | [Defect resolved / No effect, reverted on [date] / New issue, see [reference]] |
| **Follow-up** | [Root cause confirmation in a sandbox, documentation updates, or nothing further] |

## Sandbox Refresh Decision (use instead of Parts 1 and 2 for a refresh request)

| | |
|---|---|
| **Sandbox** | [Name, type, last refresh date] |
| **Why the vendor needs it refreshed** | [What is missing compared with production] |
| **What will be lost** | [Work in flight in this sandbox, confirmed with its admin] |
| **What changes afterward** | [New org ID to share with the vendor; user email addresses to correct; vendor access to restore] |
| **Smaller option considered** | [e.g., refresh a Developer sandbox instead, or load a small scrubbed data set] |
| **Decided by and decision** | [Client data owner, Approved / Declined / Deferred to [date]] |
| **Runs the refresh** | [Name] |
| **Restores vendor access** | [Name] |
| **Vendor told on** | [Date] |
