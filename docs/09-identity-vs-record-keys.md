# Identity Keys vs. Record Keys: Matching SSO Users to CRM Records

*How an integrator should tie a person who just signed in through SSO to the right record in your CRM, and why that takes two identifiers, not one, and never an email address.*

## The Short Version

Every SSO-connected integration has two separate jobs, and each one needs its own identifier:

| Job | Identifier to use | Issued by | Why this one |
|---|---|---|---|
| **Who just signed in?** | The SSO subject identifier (for example, the `sub` claim in an OpenID Connect token, mirrored onto the CRM record in a field like `External_Identity_Id__c`) | Your identity provider | Permanent, unique per person, never reassigned |
| **Which record do I read or write?** | The CRM record ID (in Salesforce, the 18-character `Id`) | Your CRM platform | The platform's own primary key; every update needs it anyway |

The integrator stores **both, together, in one row** in their own database. One authenticates, the other matches records. An integration that keeps only one of them either authenticates people and then writes to nothing, or writes to records it can no longer tie back to a person.

Email is neither of these. It's contact information, not an identifier.

## Why Email Fails as a Key

Email feels like the obvious choice because every system has it. It fails in ways that are hard to see until they've already done damage:

- **It changes.** People graduate, change employers, get married, or abandon an old address. The person is the same; the key is not. The integration either stops finding them or creates a duplicate.
- **It's shared.** Households, offices, and role-based inboxes (`billing@...`) put several real people behind one address. A match on email picks one of them, and nothing tells you it picked the wrong one.
- **It's not unique in the CRM.** Most CRMs allow duplicate emails across records, and most real-world data has them. "First match wins" is a coin flip.
- **It gets reassigned.** An address released by one person can later belong to someone else entirely.

The consequence that matters most: an email mismatch usually **fails silently**. There's no error. One person's data lands on another person's record, and the first sign is a confused member or a privacy complaint. That makes it a data-protection problem, not only a data-quality one.

## Why the SSO Identifier Isn't the Record Key Either

If the SSO identifier is permanent, it's reasonable to ask why it can't do both jobs. Three reasons:

1. **Not everyone has one.** A data sync covers every record in scope, including people who have never signed in and never will. They have a CRM record ID but no SSO identifier yet.
2. **Every write would need an extra lookup.** Updates and deletes are addressed by record ID. Resolving "SSO identifier to record ID" on every call adds a query per operation and burns API capacity that the whole org shares.
3. **You want a recovery path.** Records get merged and deleted. When a stored record ID stops working, the SSO identifier is how you find the right record again (see below). If both jobs rode on one key, a broken key would leave you with nothing to recover from.

## The Pattern

### 1. Authenticate and capture the subject identifier

After the OAuth 2.0 / OpenID Connect flow completes, read the subject identifier from the ID token or the userinfo response. In OpenID Connect, `sub` is guaranteed unique and never reassigned **within a single issuer**, so if you accept logins from more than one identity provider, store the issuer alongside it.

### 2. Resolve the CRM record once, on first sign-in

Look the person up by the identity field, not by email:

```sql
SELECT Id, FirstName, LastName, Email
FROM Contact
WHERE External_Identity_Id__c = '8f14e45f-ceea-467a-a8c9-6a5d0f3f2b11'
LIMIT 2
```

`LIMIT 2` is deliberate. Exactly one row is the only good answer. Zero rows means the person isn't provisioned in the CRM yet. Two rows means a data problem on the platform side. Neither of those should be "fixed" by the integration guessing.

### 3. Store the pair

```sql
CREATE TABLE member_link (
  identity_issuer   VARCHAR(255) NOT NULL,
  identity_subject  VARCHAR(255) NOT NULL,  -- from the SSO token; never changes
  crm_record_id     CHAR(18)     NOT NULL,  -- 18-character Salesforce Id
  linked_at         TIMESTAMP    NOT NULL,
  last_verified_at  TIMESTAMP,
  PRIMARY KEY (identity_issuer, identity_subject)
);
```

Store the **18-character** Salesforce ID, which is what the API returns. The 15-character form is case-sensitive, and it will collide in any tool or database column that compares text case-insensitively.

### 4. Use the record ID for every read and write after that

Once the link exists, operations go straight to the record by ID. No per-call lookup, no email anywhere in the path.

### 5. Handle merges and deletions deliberately

When two CRM records are merged, the surviving record keeps its ID and the others are deleted. An integrator holding a deleted ID will start getting errors such as `ENTITY_IS_DELETED` or `NOT_FOUND`.

The right response:

1. Re-resolve using the stored SSO identifier (step 2).
2. If exactly one record comes back, update the stored `crm_record_id` and retry.
3. If none or several come back, **stop and flag it for a human**. Do not create a new record, and do not fall back to matching on email.

For Salesforce contacts, a `queryAll` against the deleted record will usually show a populated `MasterRecordId` pointing at the survivor, which is useful for confirming what happened.

## Loud Failures vs. Silent Failures

| What went wrong | What you see | Loud or silent |
|---|---|---|
| Matched on email; two people share it | Data written to the wrong person | **Silent** |
| Matched on email; person changed it | Duplicate record created, history split | **Silent** |
| Stored only the SSO identifier | Extra query per write; API limit pressure over time | Slow and silent |
| Stored only the record ID; record merged | Errors, then no way to recover the link | Loud, then stuck |
| Stored both, handled merges | Occasional re-resolve, logged | Loud and recoverable |

The goal is to move every failure into the bottom row: visible when it happens, and recoverable without guessing.

## Where Email Is Still Fine

Email is useful for what it actually is: display, notifications, and human-assisted review during a one-time data migration. In a migration, an email match can **suggest** a link for a person to confirm. It should never create one automatically.

## Integrator Review Checklist

Before an integration that combines SSO and CRM data goes to production, confirm:

- The integrator can name which identifier they use for authentication and which for record matching, and they're different
- Both identifiers are stored together, with the issuer if more than one identity provider is in play
- Record IDs are stored in the 18-character form
- No code path looks a person up by email to authenticate or to write
- "Zero matches" and "more than one match" both fail loudly instead of creating or guessing
- Merge and delete errors trigger a re-resolve, not a new record

Pairs with the [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md) and the [SSO Integration Overview](08-communication-templates.md#sso-integration-overview-for-a-new-vendor) template.
