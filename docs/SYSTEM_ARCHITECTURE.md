# Growkyat — System Architecture

**Project:** Growkyat Promotional Platform  
**Document:** System Architecture  
**Version:** 0.1.0  
**Status:** Initial architecture specification — proposed, pending implementation and technical validation  
**Related document:** `README.md`

---

## 1. Purpose

This document defines the proposed system architecture for Growkyat, a multi-tenant SaaS promotional automation platform.

It establishes the responsibilities of the frontend, backend, database, background processing system, external integrations, and media storage. It also defines the engineering principles that developers must follow when implementing the platform.

This document is based on the initial Growkyat project scope and subsequent confirmed decisions. Details not yet agreed upon remain subject to technical validation.

## 2. Architecture Overview

Growkyat will use a frontend-backend architecture.

- **Frontend:** Next.js, React, and TypeScript.
- **Backend:** Python and Django REST Framework.
- **Database:** PostgreSQL.
- **Background processing:** Celery.
- **Task queue and broker:** Redis.
- **Deployment:** Docker on a Linux VPS, with Hostinger VPS proposed as the initial deployment target.
- **Media storage:** The available local disk space on the Hostinger VPS will be used initially for uploaded videos, generated designs, and rendered outputs.

The frontend will communicate with the backend through authenticated HTTPS API requests. The backend will handle authentication, authorization, tenant isolation, business rules, integrations, usage validation, and data persistence.

Background workers will process scheduled campaigns, long-running operations, and video-processing tasks outside normal web requests.

The final server configuration and resource requirements must be validated before production deployment.

## 3. High-Level System Architecture

```text
                 Merchant / User
                        |
                        v
              Next.js Frontend
           React + TypeScript
                        |
                HTTPS API Requests
                        |
                        v
             Django REST Framework
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
     PostgreSQL    Usage and      Integration
     Database      Permission     Services
                   Validation          |
                                       v
                              External Provider APIs
                              - Meta WhatsApp Cloud API
                              - Meta Graph API
                              - Brevo API
                              - Sangamam SMS API
                              - Hunter APIs
                              - OpenAI APIs
                        |
                        v
                   Redis Queue
                        |
                        v
                  Celery Workers
                        |
               +--------+---------+
               |                  |
               v                  v
          Provider APIs      Video Processing
                             HyperFrames
                             GSAP
                             FFmpeg
                                  |
                                  v
                         Hostinger VPS Disk
                         Uploaded and Generated
                         Media Files
```

**Architecture note:** This diagram represents the proposed logical flow. Exact service boundaries, queue configuration, storage paths, and deployment topology will be finalized during implementation.

## 4. Frontend Architecture

### 4.1 Technology

The proposed frontend stack is:

- Next.js
- React
- TypeScript

Tailwind CSS v4 may be used for styling, with shared CSS variables and reusable components to maintain a consistent interface. The final styling configuration should be agreed upon before implementation.

### 4.2 Frontend Responsibilities

The frontend will provide:

- Public marketing website.
- Registration, login, and account-related interfaces.
- Merchant workspace and dashboard.
- Promotional module interfaces.
- Campaign creation and scheduling interfaces.
- Lead management interfaces.
- Social publishing interfaces.
- AI Design Studio and AI Video Studio interfaces.
- AI Talking Assistant interface.
- Usage, subscription, settings, and permission views.
- Status messages, validation, loading states, and error displays.

The frontend is responsible for presenting information and collecting user input. It must not independently authorize access to tenant data or decide whether a billable operation is permitted.

### 4.3 Shared Design System

All frontend pages must use a shared design system.

The design system should centrally define:

- Growkyat brand colours and semantic colours.
- Typography and heading hierarchy.
- Button variants and dimensions.
- Form inputs and validation styles.
- Cards, modals, badges, and other reusable components.
- Spacing, border radii, and shadows.
- Responsive layout conventions.
- Loading, empty, success, warning, and error states.

Brand colours and component styles must not be independently hardcoded throughout individual pages.

The exact approved design tokens should be documented separately in `docs/DESIGN_SYSTEM.md` and implemented in the frontend theme files.

### 4.4 Frontend API Communication

The frontend will communicate with Django REST Framework through a shared API communication layer.

The implementation must establish consistent conventions for:

- API base URL configuration.
- Authentication and session handling.
- Request and response formats.
- Validation errors and API failures.
- Loading and retry behaviour where appropriate.
- Handling expired or invalid authentication.
- Environment-specific configuration.

The API base URL must be configurable through environment settings rather than hardcoded throughout the application.

The authentication mechanism, token or session strategy, and exact API response envelope must be decided and documented before the related features are implemented.

## 5. Backend Architecture

### 5.1 Technology

The proposed backend stack is:

- Python
- Django
- Django REST Framework

### 5.2 Backend Responsibilities

The backend is responsible for:

- User authentication and authorization.
- Merchant workspace management.
- Tenant-level data isolation.
- Business rules and validation.
- Promotional campaign management.
- Lead management and activity history.
- Scheduling and automation workflows.
- Third-party API communication.
- Webhook processing.
- Usage and entitlement validation.
- Recording execution results and failures.
- Managing media metadata and storage references.
- Providing data to the frontend through APIs.

All security-sensitive decisions must be enforced by the backend.

### 5.3 Backend Module Boundaries

The planned functional modules are:

1. WhatsApp Marketing.
2. SMS Marketing.
3. Email Marketing.
4. Lead Management.
5. Social Media Management.
6. AI Design Studio.
7. AI Video Studio.
8. AI Talking Assistant.
9. Usage, Billing and Settings.

The modules may have separate internal components, but they must use common authentication, tenant isolation, permission checks, usage validation, logging, and error-handling conventions.

The exact Django application boundaries and package structure will be finalized during initial implementation.

## 6. Database Architecture

### 6.1 Technology

PostgreSQL is the proposed relational database.

It will store structured platform data, including merchant accounts, users, campaign configurations, leads, social posts, execution records, usage information, and other application records.

### 6.2 Data Ownership

Every tenant-owned record must have a clear relationship to its merchant workspace, directly or through a validated ownership relationship.

Examples of tenant-owned information include:

- Campaigns and scheduled promotions.
- Leads and activity history.
- Connected account references.
- Social publishing records.
- Generated creative metadata.
- Video-processing records.
- Execution statuses.
- Usage records.
- Merchant settings.

The final database schema, relationships, constraints, and indexes must be designed during implementation.

### 6.3 Database Rules

- Use appropriate relationships and constraints to preserve data integrity.
- Validate ownership before reading or modifying tenant-owned records.
- Do not rely on frontend-supplied identifiers as proof of authorization.
- Use database migrations for schema changes.
- Do not store large media files directly in PostgreSQL.
- Store media metadata and file references in the database.
- Keep provider credentials out of ordinary application records unless an approved secure credential-management mechanism is implemented.

## 7. Multi-Tenant Architecture and Data Isolation

Growkyat is designed as a multi-tenant SaaS platform.

Multiple businesses will use the shared application, backend, and database. Each business must have access only to its own authorized workspace data.

### 7.1 Tenant Identification

For every authenticated request involving tenant-owned data, the backend must establish the authorized merchant workspace using a trusted authentication and authorization process.

A `merchant_id` supplied by the frontend must never be accepted as sufficient proof of access.

### 7.2 Tenant Isolation Rules

- Scope tenant-owned queries to the authorized workspace.
- Check workspace membership and permissions before performing operations.
- Validate ownership when accessing individual records.
- Apply the same isolation rules to API endpoints, background jobs, webhooks, and media access.
- Prevent users from accessing another merchant's campaigns, leads, usage records, connected accounts, or generated assets.
- Ensure exports and reports follow the same tenant restrictions.
- Record relevant security-sensitive actions for auditing.

### 7.3 Background Jobs

Each queued job that operates on tenant-owned data must carry sufficient validated context to identify the intended workspace and target record.

Workers must revalidate the relevant ownership and operation requirements rather than trusting arbitrary job parameters.

### 7.4 Webhooks

Provider webhooks must be validated using the relevant provider-supported verification mechanism before they are trusted.

Webhook processing must identify the relevant provider account and associated records safely. A webhook payload must not be treated as unrestricted authorization to modify arbitrary tenant data.

The exact verification procedures depend on each provider's supported API and webhook documentation.

## 8. Background Processing Architecture

### 8.1 Technology

The proposed background-processing stack is:

- Celery for task execution.
- Redis for queue and broker support.

### 8.2 Operations Suitable for Background Processing

Background jobs may be used for:

- Scheduled promotional campaigns.
- WhatsApp and SMS sending workflows.
- Outbound email workflows.
- Processing provider delivery reports.
- Retrying eligible transient failures.
- Social media publishing at scheduled times.
- AI-assisted creative generation.
- Video analysis and rendering.
- Long-running media-processing operations.
- Usage reconciliation and other scheduled maintenance tasks.

The exact task list and execution schedules will be finalized during implementation.

### 8.3 Job Execution Requirements

Every relevant job should maintain an execution state appropriate to its operation.

The implementation must define how jobs are queued, started, completed, failed, retried, or cancelled where cancellation is supported.

It must also:

- Prevent accidental duplicate processing.
- Apply controlled retries for eligible transient errors.
- Avoid endless retry loops.
- Record permanent failures and useful diagnostic information.
- Associate jobs with the appropriate merchant and module.
- Handle provider rate limits and temporary outages.
- Ensure that retry behaviour does not unintentionally duplicate billable operations.

The precise state model, retry limits, queue priorities, and scheduling strategy remain implementation decisions.

## 9. External Integration Architecture

All external integrations will be accessed through the backend or appropriate background workers. Provider credentials must not be exposed to frontend code.

### 9.1 Planned Integrations

| Provider or service | Status and planned purpose |
|---|---|
| Meta WhatsApp Cloud API | Planned: WhatsApp Business connection, supported promotional messaging, and message-status events |
| Sangamam SMS API | Planned: SMS submission and supported delivery reports |
| Brevo API | Selected: outbound promotional email and supported delivery information |
| Hunter Data Platform API | Planned: supported professional email discovery |
| Hunter Email Verifier API | Planned: email deliverability verification |
| Meta Graph API | Planned: supported Instagram and Facebook publishing |
| OpenAI APIs | Planned: AI-assisted creative generation, transcription, content analysis, and assistant interactions |
| HyperFrames, GSAP, FFmpeg | Proposed video composition, animation, and rendering technologies |

The table describes the current planning status, not verified production readiness.

### 9.2 Integration Rules

Before implementing an integration, verify:

- Current provider documentation.
- Account eligibility and onboarding requirements.
- API permissions and access requirements.
- Authentication and credential-management requirements.
- Supported endpoints and payloads.
- Rate limits and usage limits.
- Webhook availability and verification requirements.
- Current pricing and billing behaviour.
- Data privacy and retention requirements.
- Error responses and retry guidance.

Use proof-of-concept implementations to validate integrations before treating them as production-ready.

Provider-specific logic should be kept separate from general campaign and automation rules where practical.

### 9.3 Email Integration — Brevo API

Brevo API is the selected integration for Growkyat's outbound promotional email functionality.

The implementation will use the Brevo API rather than SMTP for this integration.

The backend will manage API credentials and provider communication. The frontend must never receive the Brevo API key.

The integration must be tested for authentication, supported sending operations, delivery events, error handling, and applicable account limits before production use.

Email Marketing covers outbound promotional email and supported delivery reporting. Receiving and replying to emails inside Growkyat are outside the current scope.

### 9.4 Social Media Scope

Social publishing is limited to supported Instagram and Facebook capabilities.

Publishing must respect account eligibility, permissions, content restrictions, and API capabilities available through Meta.

### 9.5 AI Provider Validation

AI model names and capabilities listed in the planning document must be verified against current provider documentation and actual account access before implementation.

Do not treat proposed model identifiers, access, or pricing as confirmed.

## 10. Media Storage and Video Processing

### 10.1 Initial Storage Decision

Growkyat will initially use the available disk space on the Hostinger VPS for media files.

This includes:

- Uploaded videos.
- Generated promotional designs.
- Rendered video outputs.
- Other supported media assets.

The exact storage directory, allowed file types, file-size limits, retention periods, and access-control implementation will be defined during server setup.

The storage directory must be configured outside temporary or disposable application locations and must persist across normal application deployments and container recreation.

### 10.2 Storage Implementation Rules

- Keep media files separate from PostgreSQL data.
- Store file metadata, ownership, and storage references in PostgreSQL.
- Use unique file names or identifiers to prevent collisions.
- Validate uploads and enforce appropriate file-size limits.
- Restrict access to private media and verify tenant ownership.
- Do not expose unrestricted server filesystem paths to the frontend.
- Monitor disk usage and establish a cleanup policy for temporary and expired files.
- Ensure deployments do not unintentionally delete uploaded media.
- Configure and test backups and recovery procedures for important media files.

**Capacity note:** The available storage depends on the selected Hostinger VPS plan and other files occupying the server. Confirm the actual disk capacity and expected video workload before setting upload limits.

### 10.3 Future Storage Consideration

S3-compatible object storage is not part of the initial storage decision. It may be considered later if media volume, storage capacity, backup requirements, or operational needs justify moving media away from the VPS.

If the storage approach changes, update this document before implementing the migration.

### 10.4 Video Processing

The proposed video-processing technologies are:

- HyperFrames.
- GSAP.
- FFmpeg.

The planned workflow includes uploading and validating talking-head video, analyzing speech, composing motion graphics, rendering output, and providing previews.

Video processing should be handled through background jobs when it is long-running.

The implementation must define supported input formats, file-size limits, processing limits, output formats, error handling, and media-retention behaviour.

These details require technical validation and are not yet finalized.

## 11. Usage, Entitlements and Billing

Usage, entitlements, and billing rules must be handled centrally by the backend.

Individual promotional modules must not implement conflicting quota or billing rules.

### 11.1 Required Validation

Before starting an operation that consumes messages, AI generation, video processing, or another billable resource, the backend must check the applicable:

- Merchant workspace.
- User permissions.
- Subscription or plan entitlements.
- Feature availability.
- Usage limits and available quota.

### 11.2 Usage Records

Usage records should identify the relevant merchant, module, operation, job, and provider reference where available.

The implementation must define how usage is recorded for successful, failed, retried, cancelled, and partially completed operations.

### 11.3 Duplicate Protection

The system must protect usage recording and billable operations against accidental duplicate processing.

The team must define idempotency rules for relevant requests, background jobs, provider responses, and webhook events.

### 11.4 Decisions Still Required

The following are not finalized:

- Commercial plans and prices.
- Quotas and feature entitlements.
- Recharge and renewal rules.
- Payment gateway.
- Usage reservation and settlement rules.
- Treatment of failed or partially completed operations.
- Approved monetary budget.

These rules must be agreed upon before implementing production billing behaviour.

## 12. Security and Operational Requirements

### Authentication and Authorization

- Authenticate users before allowing access to protected operations.
- Enforce authorization on the backend.
- Validate permissions for workspace and module operations.
- Protect account recovery and verification workflows.

### Secret Management

- Keep provider API keys and credentials on the backend.
- Use environment configuration or an approved secure secret-management solution.
- Never commit real secrets to GitHub.
- Maintain an `.env.example` file containing variable names and safe placeholders only.

### Input Validation

- Validate incoming request data on the backend.
- Reject invalid or unauthorized operations.
- Apply suitable validation to uploaded media and provider callback payloads.

### Reliable Processing

- Use controlled retries for eligible transient failures.
- Make relevant operations idempotent.
- Record execution status and permanent failures.
- Avoid accidental duplicate messaging or usage charges.

### Operational Protection

Before production release, configure and test:

- HTTPS.
- Appropriate firewall and server access rules.
- Database and media-storage access restrictions.
- Backups and restoration procedures.
- Error logging and monitoring.
- Disk-space monitoring and cleanup procedures.
- Recovery procedures.
- Resource usage and load limits.

The exact deployment configuration and operational thresholds remain to be determined.

## 13. API Design and Frontend-Backend Integration

The frontend and backend must agree on API contracts before implementing dependent features.

Each API contract should specify:

- Endpoint and HTTP method.
- Authentication requirements.
- Request parameters and payload.
- Successful response format.
- Validation and error responses.
- Permission requirements.
- Tenant ownership requirements.
- Relevant usage or quota checks.
- Pagination behaviour where applicable.

The frontend must not invent request or response fields independently of the backend implementation.

Changes to shared API contracts must be communicated to both developers and reflected in the documentation.

The exact API versioning strategy, URL conventions, response envelope, and pagination format remain to be finalized.

## 14. Development and Repository Conventions

The initial repository is expected to contain:

```text
growkyat/
├── README.md
├── docs/
│   └── SYSTEM_ARCHITECTURE.md
├── frontend/
├── backend/
├── infrastructure/
├── .gitignore
└── .env.example
```

This is the proposed target structure. Directories will be added when the relevant application setup is created.

### Development Rules

- Use the approved technology stack unless a change is agreed upon.
- Maintain one shared design system.
- Reuse common UI components rather than duplicating them.
- Keep frontend and backend responsibilities separate.
- Use Git feature branches for independent work.
- Review changes before merging into the main branch.
- Keep environment-specific configuration out of application source code.
- Update documentation when approved architectural decisions change.
- Test changes that affect shared components, API contracts, security, or usage rules.

The team must agree on branch naming, code formatting, linting, test commands, and pull-request requirements during project setup.

## 15. Decisions and Validation Still Pending

The following items must not be treated as finalized until explicitly agreed upon and verified:

1. Final frontend styling configuration.
2. Exact application folder and package structure.
3. Authentication and session strategy.
4. API response conventions and versioning.
5. Final database schema and migration conventions.
6. Tenant membership and role-permission model.
7. Celery queue configuration, retries, and job states.
8. Provider eligibility, permissions, pricing, and API compatibility.
9. Confirmed AI models and model availability.
10. Exact Hostinger VPS storage capacity and media-storage configuration.
11. Media retention, cleanup, and backup procedures.
12. Subscription plans, quotas, billing and recharge behaviour.
13. Payment gateway.
14. Production server sizing and deployment configuration.
15. Monitoring and recovery procedures.

These decisions should be documented as they are validated. Proposed integrations must not be presented as completed implementations.

## 16. Implementation Sequence

The recommended implementation sequence is:

1. Confirm the initial repository structure and development environment.
2. Establish the frontend and backend project foundations.
3. Define the shared design system and reusable frontend components.
4. Define API conventions and authentication requirements.
5. Establish the database foundation and tenant-isolation rules.
6. Implement and test authentication and workspace authorization.
7. Establish shared permission, entitlement, and usage-validation rules.
8. Establish media storage directories, access controls, and persistence.
9. Implement background-job and execution-status conventions.
10. Validate external integrations through proof-of-concept implementations.
11. Develop individual promotional modules using the shared architecture.
12. Test tenant isolation, failure handling, duplicate protection, and usage accounting.
13. Configure deployment, monitoring, backups, and production safeguards.

The sequence may be refined as implementation dependencies become clear, but security, tenant isolation, media persistence, and shared usage rules must not be bypassed to accelerate individual modules.

## 17. Document Status

This document describes the proposed architecture for Growkyat based on the initial project planning scope and decisions recorded during project setup.

It does not indicate that the frontend, backend, database, integrations, or deployment have already been implemented.

Update this document when architectural decisions are approved, technical validation is completed, or implementation requirements change.

**End of document.**
