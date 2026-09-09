# Partner & Third-Party Integration Playbook

*Practical, field-tested guidance for managing third-party vendor and partner integrations against a CRM + SSO platform.*

## About

I'm Stephanie Dlatt, an Implementation Specialist working on Salesforce architecture: integrations, single sign-on, data migrations, and the security and permission design that decides whether a connector is trustworthy or a liability.

This playbook is a generalized version of process documentation I built over several years handling third-party vendor onboarding, security provisioning, incident response, and partner communication. It's written to be useful to anyone doing similar work against a CRM platform (Salesforce or otherwise) with an SSO layer in front of it, regardless of employer or specific tooling.

**A note on genericization:** every page here has been deliberately stripped of employer-specific product names, internal tooling, real client and vendor names, and any architecture detail specific to a particular company's systems. What's left is the reusable methodology: the decision frameworks, the escalation philosophy, the security posture, and the communication patterns. Where an example is needed, it uses generic placeholder names.

More background on how I think about this work: [butlerblue.github.io](https://butlerblue.github.io) | [LinkedIn](https://www.linkedin.com/in/sdlatt)

## Contents

| Page | What it covers |
|---|---|
| [Vendor Onboarding](docs/01-vendor-onboarding.md) | Granting a client-selected, non-partner vendor access to developer resources, with proper consent and recordkeeping. |
| [Custom Application Requests](docs/02-custom-application-requests.md) | How to respond when a parent organization or local branch wants to connect a custom-built or independent-developer app, including ready-to-use messaging templates. |
| [Third-Party Integrator Security Playbook](docs/03-integrator-security-playbook.md) | The full framework for provisioning, scoping, and securing third-party API access: the responsibility line, OAuth flow selection, least-privilege permissioning, and escalation philosophy. |
| [Configuring an API-Only Integration User](docs/04-api-only-user-setup.md) | Step-by-step Salesforce configuration reference for the API-only integration user license type. Pairs with the security playbook above. |
| [Vendor and Integration Offboarding](docs/05-vendor-offboarding.md) | Fully terminating a vendor's access when a client relationship ends: deactivation, token revocation, and closing every layer, not just the login. |
| [Incident Runbook: Org Suspended or Disabled](docs/06-incident-org-suspended.md) | What to do when a client's CRM production environment gets locked, most often a billing issue, and how to safely bring integrations back online afterward. |
| [Responding to an API Limit Alert](docs/07-api-limit-alert-response.md) | Identifying and resolving runaway API consumption before it takes an entire org offline. |
| [Partner Communication Templates](docs/08-communication-templates.md) | Reusable email templates for partner onboarding, OAuth troubleshooting, permission errors, overconsumption notices, and support case redirects. |

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). Use it, adapt it, build on it, just credit where it came from.
