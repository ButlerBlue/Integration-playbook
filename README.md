# Partner & Third-Party Integration Playbook

*Practical, field-tested guidance for managing third-party vendor and partner integrations against a CRM + SSO platform.*

## About

I'm Stephanie Dlatt, an Implementation Specialist working on Salesforce architecture: integrations, single sign-on, data migrations, and the security and permission design that decides whether a connector is trustworthy or a liability.

This playbook is a generalized version of process documentation I built over several years handling third-party vendor onboarding, security provisioning, incident response, and partner communication. It's written to be useful to anyone doing similar work against a CRM platform (Salesforce or otherwise) with an SSO layer in front of it, regardless of employer or specific tooling.

**A note on genericization:** every page here has been deliberately stripped of employer-specific product names, internal tooling, real client and vendor names, and any architecture detail specific to a particular company's systems. What's left is the reusable methodology: the decision frameworks, the escalation philosophy, the security posture, and the communication patterns. Where an example is needed, it uses generic placeholder names.

More background on how I think about this work: [Portfolio](https://butlerblue.github.io/StephanieDlatt.github.io/) | [LinkedIn](https://www.linkedin.com/in/sdlatt)

## Contents

| Page | What it covers |
|---|---|
| [Vendor Onboarding](docs/01-vendor-onboarding.md) | Granting a client-selected, non-partner vendor access to developer documentation, with proper consent and recordkeeping. |
| [Custom Application Requests](docs/02-custom-application-requests.md) | How to respond when a parent organization or local branch wants to connect a custom-built or independent-developer app, including ready-to-use messaging templates. |
| [Third-Party Integrator Security Playbook](docs/03-integrator-security-playbook.md) | The full framework for provisioning, scoping, and securing third-party API access: the responsibility line, OAuth flow selection, least-privilege permissioning, and escalation philosophy. |
| [Configuring an API-Only Integration User](docs/04-api-only-user-setup.md) | Step-by-step Salesforce configuration reference for the API-only integration user license type. Pairs with the security playbook above. |
| [Vendor and Integration Offboarding](docs/05-vendor-offboarding.md) | Fully terminating a vendor's access when a client relationship ends: deactivation, token revocation, and closing every layer, not just the login. |
| [Incident Runbook: Org Suspended or Disabled](docs/06-incident-org-suspended.md) | What to do when a client's CRM production environment gets locked, most often a billing issue, and how to safely bring integrations back online afterward. |
| [Responding to an API Limit Alert](docs/07-api-limit-alert-response.md) | Confirming which limit was actually hit, then identifying and resolving runaway API consumption before it takes an entire org offline. |
| [Partner Communication Templates](docs/08-communication-templates.md) | Reusable email templates for partner onboarding, OAuth troubleshooting, permission errors, overconsumption notices, and support case redirects. |
| [Identity Keys vs. Record Keys](docs/09-identity-vs-record-keys.md) | Why an SSO integration needs two identifiers, one to authenticate and one to match records, and why email should never be either. |
| [Runbook: Records Missing from an Incremental Sync](docs/10-runbook-records-missing-from-sync.md) | Diagnosing records that never sync and never error, and fixing them safely without tripping automation or API limits. |
| [Runbook: Pre-Load Checks for a Historical Data Load](docs/11-runbook-historical-data-load.md) | Verifying a vendor's history file before it goes into the CRM: upsert keys, parent key formats, time zones, and a clean ready, held, and dropped split. |
| [Retiring the OAuth Implicit Flow](docs/12-retiring-oauth-implicit-flow.md) | Scoping, inventorying, and communicating a move from the implicit flow to authorization code with PKCE, including vendor and client message templates. |
| [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](docs/13-platform-oauth-enforcement-fleet-migration.md) | Running the program when the CRM platform tightens how outside apps sign in: exposure by sign-in pattern, ownership, one-at-a-time cutover, and leadership status reporting. |
| [Governing Interim Workarounds](docs/14-interim-workaround-governance.md) | Keeping a manual workaround a bridge with an end date: root cause first, workaround types, ownership lines, and closing it out when the fix ships. |
| [Debugging Authorization Code + PKCE Flows](docs/15-debugging-oauth-pkce-flows.md) | The specific errors you hit testing an OAuth 2.0 Authorization Code flow with PKCE against a platform's native OAuth endpoint, decoded, plus how to configure an OAuth-aware API client so you stop hand-building the requests that cause them. |
| [Partner Integration Intake](docs/16-partner-integration-intake.md) | One front door per request type, readiness gates before anything is scheduled, handling requests that arrive around intake, and making intake delays visible to leadership. |
| [Runbook: Authentication Failures Isolated to One Org](docs/17-runbook-auth-failures-one-org.md) | Deciding whether intermittent `invalid_grant` errors are one org's problem or a fleet-wide change, then diagnosing the user, the app record, and the fix. |
| [Packaging a Partner Integration into a Reusable Salesforce Package](docs/18-packaging-integration-for-reuse.md) | Turning a hand-configured integration into an installable Unlocked Package: pruning unused grants, splitting the permission set by who uses it, external ID relationships, the Recycle Bin's effect on unique fields, API limits vs. storage limits, and declaring a dependency on a package you don't own. |
| [Vendor Change Requests in a Client Org](docs/19-vendor-change-requests-in-client-orgs.md) | Who decides when a vendor asks to change a client's configuration or refresh a sandbox mid-investigation, what a complete request contains, the narrow production exception, and treating sandbox parity as an operating standard. |

## Templates

Fill-in documents that put the playbook into practice.

| Template | Use it for |
|---|---|
| [Integration Scope Document](templates/integration-scope-document.md) | Scoping a third-party integration field by field before anything is provisioned: approvals, auth, data scope, identifiers, responsibilities, cutover. |
| [Scoped Working Session Agenda](templates/scoped-working-session-agenda.md) | Turning "can we hop on a call?" into a focused session, with the invitation, agenda, and same-day recap. |
| [Defect Escalation Brief](templates/defect-escalation-brief.md) | Reporting an integration defect so it gets prioritized on the first read: traced case, confirmed cases, exposure, and numbered questions for the product owner. |
| [Weekly Integration Status Report](templates/weekly-integration-status-report.md) | A weekly upward update built to be forwarded unedited: outcomes, decisions made, and open items measured against their dates. |
| [Vendor Change Authorization](templates/vendor-change-authorization.md) | Recording a vendor's configuration change or sandbox refresh request: who decided, the before and after values, the reversal plan, and independent confirmation. |

## Claude Skills

Reusable [Claude skills](skills/README.md) built on the same methodology, ready to install and adapt.

| Skill | What it does |
|---|---|
| [explain-the-why](skills/explain-the-why/SKILL.md) | Drafts replies that explain a technical fix so the colleague can handle it themselves next time. |
| [wikijs-page-draft](skills/wikijs-page-draft/SKILL.md) | Drafts publish-ready Wiki.js pages for internal wikis or external developer docs. |
| [integrator-access-review](skills/integrator-access-review/SKILL.md) | Reviews a vendor access request against least-privilege gate checks and drafts the reply. |
| [data-load-preflight](skills/data-load-preflight/SKILL.md) | Checks a vendor's data export before it's loaded and drafts one consolidated question list for the vendor. |

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). Use it, adapt it, build on it, just credit where it came from.
