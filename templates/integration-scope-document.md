# Integration Scope Document (Template)

*A fill-in template for scoping a third-party integration before anything is provisioned. The [Third-Party Integrator Security Playbook](../docs/03-integrator-security-playbook.md) says provisioning waits until scope is signed. This is the document that gets signed.*

**How to use it:** copy this file, replace everything in `[brackets]`, and delete any guidance lines in italics before sending. If a section doesn't apply, write "Not applicable" and why, rather than deleting it. A blank section and a section that doesn't apply look identical to the next person who reads this.

## 1. Overview

| | |
|---|---|
| **Client organization** | [Client name] |
| **Integrator / vendor** | [Vendor name] |
| **Integration name** | [Short, descriptive name used everywhere else: tickets, app registration, user names] |
| **Business purpose** | [One or two sentences: what the client gets out of this] |
| **Target go-live** | [Date, owned by the integrator's launch plan] |
| **Document version / date** | [v1.0, YYYY-MM-DD] |

## 2. Approvals on File

- [ ] Written client approval received from [client contact, role] on [date]
- [ ] Approval saved to [system of record / location]
- [ ] Integrator's technical contact confirmed: [name, email]
- [ ] Secure credential-exchange handle on file for the integrator: [handle or channel]

*Nothing below this line gets provisioned until every box above is checked.*

## 3. Integration Pattern

| | |
|---|---|
| **Direction** | [Inbound to CRM / outbound from CRM / bi-directional] |
| **Trigger** | [Scheduled every N minutes / event-driven / user-initiated] |
| **Sync style** | [Incremental with watermark / full refresh / real-time] |
| **Includes SSO sign-in?** | [Yes / No. If yes, see section 6.] |

## 4. Authentication

| | |
|---|---|
| **OAuth flow** | [Client Credentials / JWT Bearer / Authorization Code, and why] |
| **Callback URI(s)** | [Exact URIs for sandbox and production. No placeholders.] |
| **Integrator egress IPs** | [Ranges, or "None available." If none, state which controls apply instead.] |
| **Refresh token policy** | [Expiration and session timeout, if the flow uses refresh tokens] |
| **Connector user** | [Username pattern for sandbox and production] |
| **QA user** | [Sandbox only] |

*If the integrator is asking for Authorization Code for an unattended sync, record the reason here. See the playbook's OAuth flow guidance before agreeing.*

## 5. Data Scope

List every object and every field. "All fields on Contact" is not a scope.

| Object | Field | Access | Direction | Required? | Notes |
|---|---|---|---|---|---|
| [Contact] | [`Id`] | Read | Out | Yes | Record match key |
| [Contact] | [`External_Identity_Id__c`] | Read | Out | Yes | SSO identity key |
| [Contact] | [`Email`] | Read | Out | No | Display and notifications only; never a match key |
| [Object] | [Field] | [Read / Create / Update / Delete] | [In / Out] | [Yes / No] | [Why it's needed] |

**Expected volumes:**

| Object | Records at go-live | Daily changes (est.) | Peak period |
|---|---|---|---|
| [Object] | [#] | [#] | [e.g., annual renewal period] |

*Volumes drive the API limit conversation. An estimate now is far cheaper than an overconsumption notice later.*

## 6. Identifiers

| Job | Identifier | Where it's stored on the integrator side |
|---|---|---|
| Authentication (who signed in) | [SSO subject identifier / field name] | [Table / column] |
| Record matching (which record) | [CRM record ID, 18-character] | [Table / column] |

Confirm the integrator stores both together. See [Identity Keys vs. Record Keys](../docs/09-identity-vs-record-keys.md).

## 7. Error Handling and Monitoring

| | |
|---|---|
| **Who monitors sync health** | [Integrator name / team] |
| **How failures are surfaced** | [Alerting tool, email, dashboard] |
| **Auth failure behavior** | [Refresh once, then fail loudly. No retry loops.] |
| **Record-not-found behavior** | [Re-resolve by identity key, then flag for review. Never create.] |
| **Escalation contact on the client side** | [Name, role] |

## 8. Responsibilities

| Item | Platform team | Integrator | Client |
|---|---|---|---|
| Sandbox with representative configuration | Owns | | Approves any real data |
| Connector and QA users, profile, permission sets | Owns | | |
| External app configuration and credentials | Owns | Receives securely | |
| Connector logic, mapping, retries, scheduling | | Owns | |
| Token generation and credential custody | | Owns | |
| QA of the integration | Provides test path | Owns | Accepts |
| Production launch plan | Coordinates cutover | Owns | Approves |
| Ongoing monitoring and maintenance | | Owns | |

## 9. Gate Checks

- [ ] No admin-equivalent or "view/modify all" permissions requested
- [ ] No temporary elevated access requested
- [ ] OAuth consent will be performed as the connector identity, never an administrator
- [ ] Integrator's non-production environments point only at the sandbox
- [ ] Credentials will be exchanged through [secure channel] only

## 10. Cutover Plan

| Step | Owner | Date |
|---|---|---|
| Sandbox testing signed off | [Integrator + client] | [ ] |
| Production credentials and callback URIs confirmed separate from sandbox | [Platform team] | [ ] |
| Cutover scheduled outside any data freeze or maintenance window | [Platform team] | [ ] |
| Go-live | [Integrator] | [ ] |
| Post-launch API usage check | [Platform team] | [Go-live + 1 business day] |

## 11. Change Control

Anything added after sign-off (a new object, a new field, a new write permission) comes back through this document as a new version with client approval. It is not a quick favor on a call.

## 12. Sign-off

| Role | Name | Date |
|---|---|---|
| Client approver | | |
| Integrator technical lead | | |
| Platform implementation lead | | |
