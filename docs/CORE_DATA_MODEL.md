# Growkyat — Core Data Model

**Status:** Proposed — Pending Team Review  
**Purpose:** Define the core shared data concepts and their relationships so all Growkyat modules follow a consistent structure.  
**Related document:** `docs/DATABASE_SCHEMA.md`

> This is a design proposal, not a finalized database schema. It does not create Django models, migrations, or database tables. Confirm open decisions before implementing affected features.

## 1. Product Context

Growkyat is a multi-tenant SaaS promotional automation platform with these planned modules:

- WhatsApp Marketing
- SMS Marketing
- Outbound Email Marketing
- Lead Management
- Social Media Management
- AI Design Studio
- AI Video Studio
- AI Talking Assistant
- Usage, Billing and Settings

The planned backend is Django REST Framework with PostgreSQL. Celery and Redis are planned for asynchronous processing. Media files will initially be stored on Hostinger VPS disk, with file metadata and references stored in the database.

Growkyat's current scope does not include ecommerce catalogues, storefronts, inventory, carts, checkout, orders, or ecommerce payment processing.

## 2. Core Entities

These are domain concepts, not a final list of database tables.

| Entity | Purpose | Status |
|---|---|---|
| User | A person who signs in to Growkyat | Required concept; authentication implementation is open |
| Merchant / Workspace | A business workspace using Growkyat | Required concept |
| Membership | Connects a user to a workspace and stores membership-specific access information | Proposed; membership rules need approval |
| Role / Permission | Defines what actions a user may perform | Required security concept; exact design is open |
| Lead | A contact managed by a merchant | Required for Lead Management |
| Campaign / Send operation | A promotional activity or request to send messages/content | Shared design needs review |
| Provider connection | Represents a merchant-authorized connection to an external provider | Needed where integrations are used |
| Media asset metadata | Describes a file, its owner, storage reference, and status | Needed where media is managed |
| Background execution | Tracks asynchronous work, status, retries, and errors | Needed for planned asynchronous features |
| Usage event / record | Records activity used for usage reporting or limits | Usage tracking is in scope; counting rules are open |
| Subscription / Plan | Represents plan and subscription information | Billing is in scope; lifecycle and payment rules are open |
| Audit event | Records selected security-sensitive actions | Recommended; scope and retention need approval |

Do not create all these models at once. Confirm requirements and relationships first, then implement models incrementally.

## 3. Proposed Relationships

These relationships are recommendations for review, not approved schema definitions.

### 3.1 User, Merchant and Membership

- A User represents a person who signs in.
- A Merchant represents a business workspace.
- A Membership links one user to one merchant and can hold membership-specific access information.
- **Proposed option:** one user may belong to multiple merchants, and one merchant may have multiple users. This is a many-to-many relationship represented through Membership.

This supports people who manage or work with more than one business. If Growkyat requires one user per workspace only, the design should be changed before implementation.

**Approval required:** Can a user belong to multiple merchants/workspaces?

### 3.2 Merchant-Owned Data

A merchant may own many leads, campaigns, provider connections, media assets, usage events, and background executions. Subscription information may also be associated with a merchant, depending on the approved billing design.

Every merchant-owned record must have a clear, verifiable ownership path.

### 3.3 Leads and Campaigns

- Leads belong to a merchant.
- Campaigns or send operations belong to a merchant.
- A campaign targeting selected leads may require a recipient record to track individual delivery status.
- WhatsApp, SMS and email may share some campaign concepts but also need provider-specific fields and statuses. Do not force all channels into one identical model without a documented reason.

### 3.4 Provider Connections

- A provider connection should be associated with the merchant that authorized it, where the integration is merchant-specific.
- Provider credentials must be protected and available only to authorized backend processes.
- Token storage, encryption, refresh, revocation and reconnection rules require review.

### 3.5 Media and AI Projects

- Media metadata should identify the merchant owner or another documented ownership path.
- AI Design Studio and AI Video Studio projects may reference input and output media assets.
- Store file bytes on Hostinger VPS disk initially, rather than in ordinary database rows.
- The backend must check authorization before serving or deleting files.

### 3.6 Background Executions

- Track task type, status, relevant merchant context, and retry/error details where appropriate.
- A job may reference a module-specific campaign, asset or project.
- Decide which fields and states are shared before creating a generic execution model.
- Provider webhooks and retried jobs may need event IDs or idempotency keys to prevent duplicate side effects.

## 4. Ownership Rules

| Data category | Proposed owner |
|---|---|
| User identity | Platform identity/authentication system |
| Membership | One user and one merchant |
| Merchant profile and settings | That merchant |
| Leads and lead activities | The merchant |
| Campaigns and send records | The merchant |
| Provider connections | The merchant that authorized the connection, where applicable |
| Media metadata | The merchant or a documented ownership path |
| Usage records | The merchant and relevant action/module, where needed |
| Background executions | The merchant when work is performed for a merchant |
| Platform-wide configuration | The platform, with access limited by platform permissions |

### Ownership Enforcement

1. Resolve the active merchant/workspace using authenticated server-side identity and validated membership.
2. Never treat a client-supplied `merchant_id` as proof of authorization.
3. Scope reads, updates, deletes, exports and media downloads to the authorized merchant.
4. Validate ownership of related records before linking them.
5. Apply appropriate checks to background jobs and provider/webhook processing.
6. Test cross-merchant access attempts automatically.

## 5. Shared Models and Module-Specific Models

### Shared concepts to reuse

Developers should agree on one platform-wide definition for:

- User/authentication integration
- Merchant/workspace
- Membership and access-control integration
- Shared media asset metadata, if assets are used across modules
- Usage tracking conventions
- Background execution conventions, if a common abstraction is approved

### Module-specific concepts

Keep channel- and feature-specific information in the relevant module where appropriate:

- Lead details and lead activities
- WhatsApp templates, configuration and provider statuses
- SMS configuration and delivery details
- Email campaign and delivery details
- Social accounts and publishing details
- AI design and video project details

Modules may reference shared entities but should not create competing User, Merchant or Membership models. Changes to shared models require review.

## 6. Security Principles

- Authenticate users before granting access to protected APIs.
- Verify that the user has a valid membership and the required permission for the active merchant.
- Enforce authorization in the backend; hiding a frontend button is not security.
- Do not trust a merchant identifier sent by the browser without checking access.
- Validate ownership of nested and related records.
- Keep provider secrets out of frontend code, source control and logs.
- Verify provider webhooks using the provider's supported verification mechanism.
- Design retries and idempotency so duplicate events do not accidentally repeat external actions.
- Avoid logging unnecessary personal data or credentials.
- PostgreSQL row-level security is an optional architectural decision; do not assume it is enabled unless implemented and tested.

### Required Security Tests

- Merchant A cannot read, update, delete, export or download Merchant B's data.
- Merchant A cannot attach Merchant B's leads or assets to its own campaigns or projects.
- Users without the required membership or permission are denied.
- Duplicate webhooks or retried jobs do not unintentionally repeat side effects.

## 7. Normalization and Data Integrity

- Use foreign keys for relationships instead of repeatedly copying related records' full details.
- Keep the canonical merchant profile in one place.
- Use a relationship model when the relationship has its own attributes, such as membership role.
- Apply merchant-scoped uniqueness when an identifier is unique only within a merchant.
- Use database constraints as well as application validation.
- Use JSON fields for genuinely flexible provider-specific metadata, not as a substitute for core relationships.
- Document intentional denormalization and how consistency will be maintained.

## 8. Open Decisions Requiring Approval

Do not silently finalize these choices:

1. Can a user belong to multiple merchants/workspaces?
2. Will authentication use Django's built-in user model or a custom user model? Decide before the first authentication migration.
3. Will primary keys use UUIDs or integer/big-integer IDs?
4. Which roles and permissions are needed?
5. Who can create a merchant/workspace?
6. Should campaigns use a shared model, separate channel models, or a limited shared core?
7. What recipient and delivery history must be retained?
8. How will provider credentials be protected?
9. What are the media retention and backup rules?
10. Which actions count toward usage, and what are the approved quotas?
11. What subscription, billing and payment rules are required?
12. Which background execution fields and states are shared?
13. What audit, deletion and retention policies are required?
14. How should platform administrators access merchant data?
15. Will PostgreSQL row-level security be used in addition to backend authorization?

## 9. Review and Implementation Workflow

1. Review this proposal with the product owner and all three developers.
2. Approve the authentication model, primary-key strategy, membership cardinality and permission approach.
3. Update `DATABASE_SCHEMA.md` only when a rule is approved.
4. Agree on API contracts and module boundaries.
5. Implement shared Django models and migrations through a reviewed branch and merge process.
6. Add migration, API and tenant-isolation tests.
7. Add module-specific models incrementally as features are designed.

Do not design every future table upfront. Stabilize the shared foundation and let module-specific details evolve through reviewed migrations.

## 10. Definition of Ready

The core model is ready for implementation when:

- Identity, workspace, membership, primary-key and permission decisions are approved.
- Each module knows which shared models it must reuse.
- Ownership paths for merchant-owned records are documented.
- API contracts and module boundaries are agreed.
- Migration and review responsibilities are understood.
- Tenant-isolation tests are included in the backend test plan.

---

**Change log:** Initial proposal. This document is not an approved schema and does not itself create Django models, migrations or tables.
