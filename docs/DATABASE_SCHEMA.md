Growkyat — Database Schema & Engineering Rules
Status: Initial design contract — not a finalized list of all database tables
Applies to: All Growkyat developers, branches, backend modules, and AI coding tools
Planned database: PostgreSQL
Backend ORM: Django ORM
Last updated: 2026-10-16
Purpose: This document establishes shared database rules so independently developed Growkyat modules can integrate safely. It is a living contract: detailed module tables are designed when the relevant feature is planned, then implemented through reviewed Django models and migrations.

1. How to read this document
Requirements and decisions are classified as:
- Confirmed requirement: explicitly established by the current Growkyat project specification or architecture.
- Proposed convention: a recommended engineering rule that the team should adopt consistently.
- Open decision: not settled yet; do not silently assume it is finalized.
This document does not itself create a database, tables, Django models, or migrations. The implemented Django models and migration history are authoritative for the schema that currently exists. This document records the rules and approved design decisions that those models must follow.
2. Database architecture and scope
Confirmed requirements
- PostgreSQL is the planned durable relational database.
- The backend is Python with Django REST Framework.
- Growkyat is a multi-tenant SaaS platform: multiple businesses use the same platform, while each business's private data must be isolated.
- Long-running work such as bulk messaging, scheduled publishing, AI generation, and video processing is intended to run asynchronously using Celery and Redis.
- Uploaded and generated media will initially be stored on Hostinger VPS disk storage. PostgreSQL stores relevant metadata and file references, not the media bytes themselves.
- Growkyat is a promotional automation platform, not an ecommerce system. Ecommerce catalogues, storefronts, inventory, carts, checkout, orders, and payment processing are outside the stated product scope.
Proposed conventions
- Use Django models and migrations for relational schema changes; do not manually change production tables outside the migration process.
- Keep relational data in PostgreSQL and use the database for relationships, constraints, status, and queryable metadata.
- Keep provider API calls, media processing, and long-running work out of database transactions for as little time as practical.
- Do not require a local PostgreSQL installation during the initial frontend-only stage. Before backend integration testing or deployment, use a supported PostgreSQL environment and test against it.
3. Core domain concepts
The following are conceptual entities, not a finalized table-by-table schema.
Concept	Purpose	Status
User	A person's account and identity	Required concept; exact authentication implementation is an open decision
Merchant / Workspace	A business workspace that owns business data	Required concept
Membership	Connects a user to a workspace and describes their access there	Recommended model; membership cardinality needs approval
Role / Permission	Defines which actions a member can perform	Permission enforcement is required; exact role model is open
Subscription / Plan	Represents subscription and plan information	Commercial details and lifecycle rules are open
Usage record / Usage event	Records usage relevant to limits, reporting, or billing	Usage checks are required; exact quota and charging rules are open
Audit event	Records selected security-sensitive or administrative actions	Retention and event scope need approval
Background job / execution	Tracks asynchronous work and its status	Required by planned async processing; shared abstraction needs design
Media asset metadata	Records file location, type, size, ownership, and related metadata	Required where media is managed


Core relationship rules
1. Every business-owned record must have a documented, verifiable path to its owning merchant/workspace.
2. Shared concepts such as users, merchants, memberships, permissions, and usage must not be independently re-created by each module.
3. Module-specific records should reference the shared entities through explicit relationships.
4. The team must approve whether a user can belong to multiple merchants before finalizing membership constraints and authentication/workspace selection.
5. Do not finalize role names, plan names, pricing, quota values, or subscription rules without approved requirements.
4. Multi-tenant ownership and isolation
Tenant isolation is a security requirement, not merely a database naming convention.
Required rules
1. Resolve the active merchant/workspace from the authenticated server-side identity and validated membership/context.
2. Never treat a client-supplied merchant_id as proof of authorization.
3. Scope every merchant-owned read, update, and delete to the authorized merchant.
4. On create operations, derive or validate ownership on the backend; do not accept an arbitrary merchant identifier from the client.
5. Check ownership across related objects too. A record belonging to Merchant A must not be attached to a campaign, connected account, asset, or job belonging to Merchant B.
6. Apply the same checks to background jobs, file downloads, exports, webhook processing, and administrative endpoints.
7. Webhook authenticity and event ownership must be verified using the provider's supported verification mechanism before processing events.
8. Add automated tests proving that one merchant cannot read or modify another merchant's data, including by guessing or substituting record IDs.
Data-model guidance
- Prefer an explicit merchant foreign key on merchant-owned records when that makes ownership clear and queryable.
- If ownership is inherited through a parent relation instead, document that path and ensure all queries enforce it consistently.
- Where uniqueness is merchant-specific, use a composite uniqueness rule such as (merchant, external_id) rather than assuming the value is globally unique.
- Consider database-level protections, including PostgreSQL row-level security, only after the team has designed, implemented, and tested a complete approach. Do not assume row-level security is active merely because it is mentioned here.
5. Normalization and data duplication
Default policy
Use normalized relational design, generally aiming for Third Normal Form (3NF) where appropriate. Avoid duplicating facts across tables without a documented reason. Normalization does not replace tenant authorization, constraints, or testing.
- First Normal Form (1NF): store atomic values in columns and avoid repeating groups or comma-separated collections that need to be queried as separate records.
- Second Normal Form (2NF): where a table has a composite candidate key, every non-key attribute must depend on the whole key, not just part of it.
- Third Normal Form (3NF): non-key attributes should depend on the key, not on other non-key attributes through a transitive dependency.
Practical rules
- Store a merchant's business name in the merchant record; do not copy it into every lead row as the authoritative value.
- Use foreign keys for relationships instead of duplicating related entities' full data.
- Use a join/through model for many-to-many relationships when the relationship has its own attributes, such as membership role or join status.
- Store repeated events, activities, or delivery attempts as separate records when their history matters.
- JSON fields may be appropriate for genuinely flexible provider-specific metadata. Do not hide core relationships, authorization fields, or frequently filtered fields inside JSON merely to avoid schema design.
- Denormalization or cached aggregates require a documented reason, clear source of truth, and a plan for keeping values consistent.
6. Model conventions
The following are proposed conventions to apply consistently once approved.
Naming
- Use clear, singular Django model class names, such as Lead or SocialPost; Django's default table naming may be used unless there is a documented reason to override it.
- Use snake_case for Python fields and database column names.
- Use explicit, descriptive relationship names. Avoid ambiguous fields such as owner_id when the actual ownership concept is a merchant or user.
- Do not create a new shared model because a module needs a slightly different view of existing data.
Primary keys and foreign keys
- Use one consistent primary-key strategy across the project. Open decision: approve integer/big-integer IDs or UUIDs before the first persistent models are implemented.
- Use Django ForeignKey, OneToOneField, and ManyToManyField where their semantics fit; specify on_delete deliberately.
- Avoid unmanaged database tables and manually maintained IDs unless there is an approved integration reason.
Common fields
- Use timezone-aware timestamps. A shared base model may provide created_at and updated_at where appropriate.
- Add timestamps only where meaningful; event and state-transition history may need more specific timestamps.
- Use explicit choices or validated state transitions for finite status sets.
- Store money, if introduced by approved requirements, using an appropriate decimal representation and explicit currency rather than floating-point values.
- Store dates/times consistently and use UTC internally where appropriate; convert to the user's timezone for display.
- Avoid storing derived values as independent truth unless there is a documented consistency strategy.
Sensitive data
- Never store passwords as plaintext; use Django's supported password hashing/authentication mechanisms.
- Keep provider secrets and API tokens out of frontend code, source control, logs, and ordinary plaintext fields. If tokens must be persisted, use a reviewed secret-protection strategy and restrict access.
- Do not log authentication tokens, secret values, or unnecessary personal data.
7. Constraints, indexes, and integrity
Use database constraints as well as application validation.
- Add foreign keys for real relationships.
- Add unique constraints for true uniqueness requirements.
- Use merchant-scoped uniqueness when a value only needs to be unique within a workspace.
- Use check constraints for invariants that the database can reliably enforce.
- Make fields non-null only when the business rule genuinely requires a value.
- Define deletion behavior intentionally. Do not cascade-delete important business history by accident.
- Add indexes to support known filters, joins, ordering, uniqueness, and tenant-scoped queries. Do not add indexes blindly; indexes have write and storage costs.
- For common tenant queries, evaluate indexes that begin with the merchant/workspace key and continue with the relevant filter or ordering fields.
- Review whether a constraint or index needs to be added in a migration and test it with realistic data where practical.
8. Module boundaries and shared ownership
Each module must own its specific business logic and module-specific records, while reusing shared platform concepts.
Module	Typical data responsibility
Authentication and workspace access	User identity integration, memberships, roles/permissions, workspace selection
Lead Management	Leads, notes, activities, follow-ups, import history
WhatsApp Marketing	WhatsApp-specific templates/configuration, campaigns or send records, provider delivery events
SMS Marketing	SMS-specific campaign/configuration and delivery records
Outbound Email Marketing	Email campaign/configuration, recipient/send records, delivery events; no inbound/reply inbox unless separately approved
Social Media Management	Connected social accounts, scheduled/published posts, provider status
AI Design Studio	Design projects, generated asset metadata, prompts/settings where retention is approved
AI Video Studio	Video projects, processing state, output asset references, failure information
AI Talking Assistant	Approved assistant settings and operation records if persistence is required
Usage and Billing	Plans, subscriptions, usage events, quota/entitlement state as requirements are finalized
Shared execution infrastructure	Job execution status, retries, provider references, idempotency, error classification where a shared model is approved


This table defines boundaries, not a final list of models. Before introducing a shared campaign or execution abstraction, document its responsibilities and verify that it fits the actual modules. Do not create competing generic systems independently in different modules.
9. Transactions, background jobs, retries, and concurrency
- Use database transactions when multiple related changes must succeed or fail together.
- Keep transactions short. Do not hold a transaction open while waiting for external provider APIs or long-running media processing.
- Assume background tasks and webhook deliveries can be retried.
- Use provider event identifiers, idempotency keys, or equivalent deduplication controls where supported so retries do not accidentally repeat side effects.
- Record job state transitions and failures where needed for reliable operations and support.
- Define retryability and terminal failure states deliberately; do not retry permanent validation or authorization failures indefinitely.
- Protect quota and balance-like counters from concurrent updates if those concepts are introduced. Use suitable atomic database operations, transactions, locks, or another reviewed strategy.
- Do not assume that a database transaction can roll back an external WhatsApp, SMS, email, social, or AI provider action.
- Make partial-success behavior explicit for bulk operations.
10. Media metadata and file storage
The initial storage decision is Hostinger VPS disk storage. Actual disk paths, capacity, permissions, retention, backup, and recovery configuration must be established during deployment.
- Store file bytes on the configured media storage, not as large binary blobs in ordinary relational tables.
- Store metadata such as owning merchant, storage-relative path or key, media type, size, checksum if needed, processing status, and timestamps where appropriate.
- Never expose raw server filesystem paths or allow a client to choose arbitrary paths.
- Authorize media access by merchant ownership before serving or deleting a file.
- Validate uploaded file types and sizes and use safe file names.
- Ensure generated files and temporary processing outputs have an explicit lifecycle and cleanup policy.
- Back up media consistently with the database metadata. A database backup without its referenced media is not a complete recovery plan.
- Reassess object storage if VPS capacity, durability, or operational needs justify it; do not switch storage without an approved change.
11. Django models and migrations
- Django models and their committed migrations define the implemented schema.
- Every schema change must be represented by a migration committed to version control.
- Do not manually alter a shared or production database and leave the migration history unaware of the change.
- Do not delete, rewrite, or squash migrations that may already be applied in a shared environment without an explicit migration plan and review.
- When branches independently create migrations for the same Django app, inspect migration dependencies and reconcile conflicts before merging. Use Django's migration tooling where appropriate, then run the full migration check.
- After syncing with the integration branch, test migrations from a clean database and, when applicable, from the previous supported schema.
- Data migrations must be safe, repeatable where feasible, and considered separately from schema migrations.
- For incompatible changes, use an expand-and-contract approach when needed: add compatible structures, migrate data and application code, then remove obsolete structures in a later reviewed change.
- Do not make the frontend depend on database table names. Frontend integration should use documented API contracts.
12. Three-developer Git and AI workflow
For every change that affects persistent data:
1. Pull or rebase against the team's agreed base branch according to the repository workflow.
2. Inspect existing models, migrations, services, and API contracts before proposing new ones.
3. Describe the proposed schema change before implementation: purpose, fields, relationships, constraints, ownership, migration impact, and API impact.
4. Reuse shared models; do not create duplicate User, Merchant, Membership, or other shared entities.
5. Implement the model change, migration, validation, and tests together.
6. Run Django checks, migration checks, relevant tests, and tenant-isolation tests.
7. Review changes to shared models, permissions, and migration dependencies before merging.
8. Resolve migration conflicts intentionally; do not blindly accept both generated migrations or delete one to make the check pass.
9. Update this document when an approved architectural rule or important shared relationship changes.
10. Merge only through the team's agreed review process.
AI tools must stop and report conflicts with the existing schema or documented requirements rather than silently inventing a new architecture.
13. Required testing checklist
Relevant tests should cover:
- Model creation and required-field validation.
- Foreign-key integrity and deletion behavior.
- Unique and check constraints, including merchant-scoped uniqueness.
- Transaction rollback for multi-record operations.
- Cross-merchant read, update, delete, export, and media-access attempts.
- Access to related records that belong to another merchant.
- Duplicate webhook or job-event processing.
- Retry and failure behavior for asynchronous operations.
- Concurrent usage/quota changes where applicable.
- Migration application from a clean database.
- Migration compatibility after integrating changes from multiple branches.
- API behavior and permissions for success, validation failure, and unauthorized access.
A passing UI build is not evidence that database integrity or tenant isolation is correct.
14. Schema change proposal and review
Before adding or changing a model, the developer or AI tool should provide:
- Reason: what requirement needs the change?
- Existing-model check: why can existing models not meet the requirement?
- Proposed design: models, fields, types, relationships, constraints, and indexes.
- Ownership: which merchant owns each record, and how is access enforced?
- Normalization: what facts are stored, and is any duplication intentional?
- Migration: how will existing data be preserved or transformed?
- API impact: does the change alter request or response contracts?
- Tests: what integrity, tenant, migration, and regression tests will be added?
- Rollback/recovery: what is the safe response if deployment fails?
Shared model, identity, membership, permission, tenant-isolation, and migration-history changes require explicit review before merge.
15. Open decisions — do not silently finalize
The following decisions require confirmation in the relevant project discussion before affected functionality is implemented:
1. Whether one user may belong to multiple merchants/workspaces.
2. Authentication method and whether the project uses Django's built-in user model or a custom user model. This must be decided before the first authentication migrations.
3. Primary-key strategy: UUID or integer/big-integer IDs.
4. Exact membership, role, and permission design.
5. Whether platform staff, merchant owners, and merchant staff need distinct identity or authorization paths.
6. Subscription lifecycle, plan entitlements, quota definitions, and billing/payment integration. Do not invent pricing or limits.
7. Whether campaign and background-job execution records use one shared model or module-specific models connected to shared infrastructure.
8. Data retention, audit-log scope, soft deletion, and permanent deletion rules.
9. Exact media path strategy, capacity, backups, retention, and recovery on the Hostinger VPS.
10. Whether PostgreSQL row-level security will be used in addition to application-level tenant authorization.
11. Provider-specific data retention and credential/token encryption requirements.
12. Production migration, backup, restore, and deployment procedures.
16. Definition of done for a database change
A database-related change is ready for review only when:
- It follows the approved architecture and reuses shared models.
- Ownership and authorization are clear.
- Normalization, constraints, and indexes have been considered.
- A migration is included and migration history is consistent.
- Relevant automated tests pass, including tenant-isolation tests where applicable.
- API contracts and documentation are updated when affected.
- No secrets, local database files, virtual environments, or generated media are committed.
- Any unresolved architectural choice is explicitly documented rather than silently decided.
Change log
- Initial version: establishes database engineering conventions and the review process. It intentionally does not define every future module table or settle the open decisions above.
