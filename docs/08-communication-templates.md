# Partner Communication Templates

*Reference templates for common partner and vendor communications. Replace the signature block with your own name, title, and organization in each of these.*

**Standing rule across all of these:** never include credentials, passwords, tokens, or API keys in email. Send anything sensitive through a secure credential-exchange channel instead.

## New Partner Welcome

*Use when a new partner has been formally accepted into your partner program and needs to set up their developer resource center listing.*

> Welcome officially to the Partner Program, we're excited to have you on board.
>
> As a next step, we'd love for you to complete your resource center partner page. This page is visible to current clients and prospects, so it's a great opportunity to showcase your offerings and how best to engage with your team. A few ways to get there:
>
> **Fill out our template document.** Once completed, we'll convert it into our resource center format and send it back for your review.
>
> **Build it directly.** Prefer to take the reins yourself? Let us know and we'll grant you edit access to your partner page.
>
> **Send us your existing content.** If you already have marketing materials, onboarding docs, or a preferred write-up, send them over and we'll incorporate them into the layout for your approval.
>
> Before you can access the resource center (including a sample mock-up page), you'll need to create an account with our SSO platform. Once you've done that, send us your login identifier and we'll set up your permissions.
>
> As a partner, you'll get: a dedicated channel for real-time collaboration and support, enhanced visibility in the resource center, access to integration tools and templates, and marketing mentions across our social, email, and webinar channels.
>
> Let us know which option you'd like to move forward with.

## Sandbox Access for a New Integration Build

*Use when granting a developer their first sandbox access to build a client integration.*

> Hi [Name],
>
> I've created a system admin user for you in the [Client Name] sandbox. A password reset email has been sent; use that to set your password and log in. Your username is:
>
> [sandbox username]
>
> A quick note on the username: your platform likely requires every username to be unique across the entire system, not just within one org, so even though your login email stays the same, the username follows this distinct format. This also sets you up well going forward, if you ever need sandbox access for another client, we can create a separate user following the same pattern without conflicts.
>
> Your connector user for the API integration itself is:
>
> [connector username]
>
> If you need additional test users, just let me know.
>
> A few important things as you get started:
>
> - **Use your system admin login for hands-on building and QA.** That's what you'll use to navigate Setup, create objects and fields, and test as a user.
> - **Always use the connector user for integration testing.** When testing the actual data flow from your integration into the CRM, run it as the connector user, not your system admin account. The connector user is the API-only account that mirrors how the integration will actually run in production. Testing through it confirms your permission set is correctly scoped and that nothing is accidentally relying on admin-level access a real connector wouldn't have.
> - **Sandbox vs. production:** the sandbox is your workshop, production is the live environment your client actually uses. Build and test everything in the sandbox first; once validated, promote to production. Never build or test directly in production, and the same applies if you're packaging your work, validate the package in a sandbox before it ever touches a production org.
>
> Since this is your first time in this environment, a few resources to get oriented:
> - Salesforce's Admin Beginner Trailhead trail, the best starting point for anyone new to the platform.
> - The Object Manager / Custom Objects guide, since you'll be creating several custom objects and fields.
> - The Permission Sets overview, since you'll need to build and package a permission set for your connector user.
> - The Sandbox Overview guide, for understanding sandbox types and how they relate to production.
> - The Unlocked Packages developer guide, our recommended packaging approach for deploying across orgs.
>
> Start with Trailhead if you haven't already, even an hour or two makes navigating Setup much more intuitive. Reach out as questions come up.

## SSO/OAuth Consent Screen Explanation

*Use when a partner or developer is confused by the OAuth consent screen, or needs guidance configuring what users see.*

> Subject: RE: OAuth Consent Screen
>
> Hi [Name],
>
> Great question. What you're seeing is the standard OAuth consent screen, which appears the first time a user authorizes your application to access their identity. This is expected, standard OAuth 2.0 behavior.
>
> What the screen does:
> - Asks the user to confirm they trust your application and consent to granting access.
> - Displays your application's name, the permissions (scopes) being requested, and an Allow/Deny prompt.
>
> If you need to update what users see (app name, branding), that configuration lives in your app registration, review what you submitted at registration and confirm the display name and description are correct. Let us know if anything needs updating on our side.
>
> One thing to double-check: the redirect URI in your app registration must exactly match the callback URL your application sends in the authorization request. Even a small mismatch (a trailing slash, for example) will cause the flow to fail after the user consents.
>
> Happy to walk through the full flow with you or review your registration if useful.

## SSO Integration Overview for a New Vendor

*Use before a new vendor begins development, to set the right foundation for SSO integration.*

> Subject: SSO Integration: What You Need to Know Before You Build
>
> Hi [Name],
>
> Happy to walk you through how our SSO platform fits into your integration before you start building.
>
> Our platform gives members a single set of login credentials that works across our member portal and any partner applications that support it. For your integration, the most critical concept is the external identity field in our CRM (a field like `External_Identity_Id__c`), the unique, permanent identifier for every SSO user. Use this field to look up a member after they authenticate. Don't use email as a lookup key, emails change; this identifier doesn't.
>
> To get connected, you'll need to:
> 1. Register your app using our app registration guide, this establishes your redirect URI and credentials.
> 2. Implement the OAuth 2.0 flow so users can authenticate and return to your application with a valid session.
>
> Our full integration guide in the developer resource center covers the rest, from new user registration through sandbox vs. production environments. Let me know what comes up as you work through it.

## Permission Error Response

*Use when a partner reports a permissions error on their connector user for a specific object or field.*

> Subject: RE: Permission Error in [Client Name] Sandbox
>
> Hi [Name],
>
> Thanks for flagging this, I'll take a look at the connector user's profile and confirm what's happening.
>
> To help resolve it quickly, please share:
> - The full error message your system received
> - The object and field you were trying to access
> - Whether this is happening in sandbox, production, or both
>
> Once I have that, I can determine whether this is a profile gap that needs addressing in your scoping, or something adjustable within your current approved scope.
>
> As a reminder, if you hit a permissions error, please reach out to us rather than modifying the connector user's profile or permissions directly. All permission changes go through us to stay aligned with your scoping document and avoid unintended access.

## API Overconsumption Notice

*Use when a partner's integration is consuming a disproportionate share of a client's daily API limit.*

> Subject: API Request Usage: Action Needed for [Client Name]
>
> Hi [Name],
>
> I wanted to flag something I've been tracking on the [Client Name] org. Your integration has been consuming a significant share of the org's daily API request limit. Since that limit is shared across every integration running in the org, overconsumption by one can create problems for others.
>
> At this point, we need you to review your implementation. The most common areas to look at:
> - **Polling frequency:** reducing how often you poll can meaningfully lower total usage.
> - **Field selection:** query only the fields you need rather than full records.
> - **Caching:** for data that doesn't change often, client-side caching eliminates repeat calls.
> - **Bulk API:** for large record volumes, bulk operations are far more efficient than per-record calls.
>
> You can monitor your consumption anytime through the platform's API limits endpoint. Please review and share a plan for addressing this, happy to get on a call to talk through the approach if useful.

## Documentation Update Request

*Use before a new client launch, after a partner's major product update, or as part of an annual partner review.*

> Subject: Action Needed: Update Your Resource Center Listing
>
> Hi [Name],
>
> We're reviewing partner documentation in our resource center and want to make sure your listing is current. This page is where clients and prospects learn about your product and integration scope, so keeping it accurate directly affects how you're presented.
>
> A few things to confirm or update:
> - Product description and key features, still accurate?
> - Integration scope, still reflects what your integration actually does?
> - Implementation timeline, still realistic?
> - Support contact information, correct email or portal link?
> - FAQ, any new questions worth adding based on what clients commonly ask?
>
> If you'd like to update it directly, let us know and we'll grant edit access. Otherwise, send us the updates and we'll make the changes. Please aim to have this reviewed by [Date].

## Support Case Redirect

*Use when a support request should have been routed to the partner's own support channel instead.*

> Subject: RE: [Issue Description] for [Client Name]
>
> Hi [Name],
>
> Thanks for reaching out. After reviewing this request, it falls within [Partner Name]'s scope of support rather than ours.
>
> For issues like [describe issue type, e.g., login issues within the partner application, data discrepancies originating from the partner's system], the right first contact is [Partner Name]'s support team directly: [partner support contact or URL].
>
> As a reminder, requests about a partner's application and its direct behavior should go to the partner first. We step in for issues involving the CRM configuration, the integration connection itself, or anything requiring action on the org side. If it turns out our involvement is needed, [Partner Name] will escalate to us directly.
