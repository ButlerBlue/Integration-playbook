# Handling Custom Application Requests Outside a Vetted Vendor Relationship

*Use this when a headquarters/parent organization or a local branch introduces a custom-built application or independent developer requesting SSO authentication and/or CRM data access, outside of any existing vetted partner relationship.*

## Background

A headquarters team or local chapter/branch officer may approach your Client Success or Support team about integrating a custom-built third-party application, often built by a member, alumnus, or small independent developer. These tools typically promise niche functionality (wellness reporting, engagement tracking, member self-service) and request SSO authentication plus access to CRM roster data.

**The key risk:** your SSO layer should handle authentication only, not data access. Exchanging actual member data requires a separate, full API integration with your CRM platform (e.g., Salesforce), along with ongoing maintenance and adherence to your security and platform standards. Small custom projects frequently underestimate this distinction, creating sustainability, security, and support risk for both the parent organization and the broader ecosystem of clients you support.

This playbook exists to keep responses consistent, protect platform integrity, and help the parent organization make an informed decision, while still leaving room to support legitimate innovation.

**Important:** Your contract and relationship are with the parent/headquarters organization, not the local branch, chapter, or individual member. They make the final call on whether to grant access to a local officer, alumnus, or niche developer. Your role is to inform and facilitate, not to approve or deny on their behalf.

## When This Scenario Applies

- A parent organization introduces a custom-built app or independent developer requesting platform access.
- The app requests SSO authentication and/or CRM roster data without acknowledging the need for a full API integration.
- A parent-organization contact asks whether you support or approve a specific integration.
- Support receives a request directly from a local branch/chapter member wanting to integrate SSO for a local-only custom app.
- A vendor requests developer access or documentation without documented consent from the organization they claim to represent.

## Goals

- Clarify what your SSO layer can and cannot provide.
- Surface long-term ownership and sustainability concerns.
- Ensure the parent organization understands the full complexity of a custom integration.
- Offer vetted partner or supported alternatives when appropriate.
- Provide structured developer onboarding documentation when an integration is approved.

## Response Templates

### Messaging to the Parent Organization: Introducing a Niche Vendor or Independent Developer

*Use when a parent-organization contact introduces a custom app or developer and asks whether you support or approve the integration.*

> Hi {Contact Name},
>
> Thanks for sharing more detail about the application you're exploring. We want to make sure you have a clear picture of how this would connect with your existing systems so you can evaluate feasibility and long-term sustainability.
>
> Our SSO layer provides authentication only, it does not expose roster or member data. Any exchange of member information would require a separate API integration with your CRM platform. That means building and maintaining an external application connection, managing authentication tokens, monitoring API usage, and ensuring compatibility as the platform evolves.
>
> Before moving forward, it's worth clarifying ownership and support expectations. Custom integrations often succeed technically at first, but long-term maintenance, security oversight, and troubleshooting responsibility need to be defined upfront. We'd also recommend confirming whether any of your existing supported integrations could meet your goals with less operational risk.
>
> We're happy to walk through architecture options, review scope, or share relevant documentation so your team can make an informed decision. Let us know how you'd like to proceed.

### Messaging to the Parent Organization, Originating from a Support Ticket

*Use when Support receives a ticket directly from a local branch/chapter member or developer, before the parent organization has been involved.*

> Hi {Contact Name},
>
> We wanted to make you aware of a request our support team received from a local chapter member regarding a potential external application or integration. Since we coordinate these efforts at the headquarters level, we're flagging this for your visibility before any next steps are taken.
>
> Request details:
> - Member: {name}
> - Chapter/Branch: {chapter}
> - Email: {email}
> - Ticket submitted: {date}
>
> At this stage, no action has been taken. We defer to your team to determine whether you'd like to explore this further with the member or developer involved.
>
> For context, requests like this typically involve SSO authentication along with access to your CRM data. SSO supports identity authentication only, it does not provide roster or member data. Any data exchange would require a separate API integration, which carries long-term responsibilities around maintenance, security, and platform compatibility.
>
> If you'd like to evaluate this opportunity, we're happy to walk through architecture considerations, discuss feasibility, or share documentation to support your review. Let us know how you'd like to proceed, or if you'd like us to coordinate next steps with the member on your behalf.

### Messaging to the Support Ticket Submitter / Developer

*Use when responding directly to a local branch/chapter member or external developer who submitted a request.*

> Hi {Name},
>
> Thanks for reaching out, we appreciate you looking for ways to improve the technology experience for your chapter.
>
> Our SSO and the underlying data environment are managed at the headquarters level for your organization, so we aren't able to authorize or move forward with integration requests directly from individual chapters. Any evaluation or approval needs to come from your national headquarters, so they can ensure it aligns with their broader systems, security standards, and long-term plans.
>
> Please connect with {HQ contact name or email} to discuss your request and next steps. They'll be the best resource to determine whether and how this could move forward.
>
> For now, we'll close this support case while you coordinate with headquarters. If they'd like us to re-engage after reviewing, we'll be happy to jump back in.
>
> Thanks again for reaching out, and we appreciate your understanding.
