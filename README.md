# Growkyat --- Promotional Platform

**Project type:** Multi-tenant SaaS promotional automation platform\
**Project status:** Initial planning and repository setup\
**Version:** 0.1.0 (planning draft)

## 1. Project Overview

Growkyat is a proposed SaaS platform that brings business promotional
activities into one merchant workspace. The platform is intended to help
businesses manage promotional messaging, social media publishing,
AI-assisted content creation, lead management, and supported automation
workflows.

Each business will have its own workspace, connected accounts, settings,
promotional data, usage records, and permissions. The application will
use a shared backend and database while enforcing strict tenant-level
data isolation.

> **Current phase:** Initial project setup and technical planning.
> Features, API integrations, deployment capacity, and pricing must be
> validated during implementation.

## 2. Project Objectives

-   Centralize supported promotional activities in one application.
-   Automate WhatsApp, SMS, and outbound email campaigns.
-   Schedule supported Instagram and Facebook publishing.
-   Generate promotional designs and videos with AI-assisted workflows.
-   Organize leads and their activity history.
-   Provide a text/voice assistant for supported platform operations.
-   Track usage, quotas, execution status, and subscription
    entitlements.
-   Protect merchant data through authentication, authorization, and
    tenant isolation.

## 3. Planned Modules

  -----------------------------------------------------------------------
  Module                              Initial scope
  ----------------------------------- -----------------------------------
  WhatsApp Marketing                  Connect eligible WhatsApp Business
                                      accounts, manage approved
                                      templates, send/schedule
                                      promotions, process status events

  SMS Marketing                       Manage sender and approved template
                                      details, send/schedule SMS, process
                                      delivery reports

  Email Marketing                     Prepare and schedule outbound
                                      promotional email; use supported
                                      delivery reporting

  Lead Management                     Create/import leads, track status
                                      and notes, maintain activity
                                      history and follow-ups

  Social Media Management             Prepare and schedule supported
                                      Instagram and Facebook publishing

  AI Design Studio                    Configure brand details, generate
                                      and save promotional creatives

  AI Video Studio                     Upload and validate talking-head
                                      video, analyze speech, compose
                                      motion graphics, render and preview
                                      output

  AI Talking Assistant                Support authorized text/voice
                                      requests for platform operations

  Usage, Billing and Settings         Display plans, entitlements, usage
                                      records, quotas, settings, and
                                      permissions
  -----------------------------------------------------------------------

### Scope notes

-   Email Marketing currently covers outbound promotional email;
    receiving and replying to emails inside the platform are not
    included.
-   The AI Talking Assistant is limited to approved promotional-platform
    operations; ecommerce catalogues, inventory, carts, checkout,
    orders, and payment processing are out of scope.
-   Third-party model names and API capabilities in the planning
    document are proposed integrations, not confirmed dependencies.
    Verify current provider documentation, model identifiers, access,
    permissions, and pricing before implementation.

## 4. Proposed Technology Stack

  -----------------------------------------------------------------------
  Layer                   Proposed technology     Purpose
  ----------------------- ----------------------- -----------------------
  Frontend                Next.js, React,         Merchant dashboard and
                          TypeScript              module interfaces

  Backend                 Python, Django REST     Authentication,
                          Framework               business rules, APIs,
                                                  integrations, webhooks

  Database                PostgreSQL              Structured platform and
                                                  tenant data

  Background jobs         Celery                  Scheduled and
                                                  long-running tasks

  Queue/broker            Redis                   Background job queue
                                                  support

  Initial server          Linux VPS (proposed:    Initial deployment
                          Hostinger VPS)          target, subject to load
                                                  testing

  Deployment              Docker                  Consistent service
                                                  packaging and
                                                  deployment

  Media storage           S3-compatible object    Uploaded videos,
                          storage (recommended)   generated designs,
                                                  rendered outputs

  Video processing        HyperFrames, GSAP,      Motion graphics and
                          FFmpeg                  video rendering
  -----------------------------------------------------------------------

The proposed stack is subject to technical validation. The team should
confirm the final architecture before production implementation.

## 5. Planned External Integrations

  -----------------------------------------------------------------------
  Provider/service                    Planned use
  ----------------------------------- -----------------------------------
  Meta WhatsApp Cloud API             WhatsApp Business connection,
                                      messaging, supported status events

  Sangamam SMS API                    SMS delivery and supported delivery
                                      reports

  Brevo API/SMTP                      Outbound promotional email

  Hunter Data Platform API            Supported professional email
                                      discovery

  Hunter Email Verifier API           Email deliverability checks

  Meta Graph API                      Supported Instagram and Facebook
                                      publishing

  OpenAI APIs                         Planned AI creative generation,
                                      transcription, content analysis,
                                      and assistant interactions

  HyperFrames, GSAP, FFmpeg           Video composition and rendering
  -----------------------------------------------------------------------

All integrations must be tested with proof-of-concept implementations.
Keep provider credentials on the backend and out of frontend code.
Confirm provider eligibility, API permissions, webhook support,
data-handling requirements, usage limits, and current pricing.

## 6. High-Level Architecture

1.  The merchant uses the Next.js application.
2.  The frontend sends authenticated HTTPS requests to the Django REST
    Framework API.
3.  The backend identifies the merchant workspace and checks
    permissions, input validity, and applicable usage limits.
4.  PostgreSQL stores structured records, including campaigns, leads,
    schedules, statuses, and usage.
5.  Celery workers process scheduled or long-running jobs, with Redis
    supporting the queue.
6.  Workers call the relevant provider API or perform media-processing
    tasks.
7.  Validated provider webhooks and responses update execution statuses.
8.  Media assets are stored separately in object storage; PostgreSQL
    stores ownership and metadata references.
9.  The dashboard displays results, errors, usage, and available
    analytics.

## 7. Core Engineering Principles

-   **Tenant isolation:** Every tenant-owned record and operation must
    be scoped to the authenticated merchant. Never trust a
    client-supplied `merchant_id` as authorization.
-   **Server-side secrets:** API keys, tokens, and provider credentials
    must never be exposed in frontend code.
-   **Usage checks:** Validate entitlements and available quota before
    starting billable operations.
-   **Reliable jobs:** Use controlled retries for transient errors,
    avoid endless retries, and record permanent failures.
-   **Idempotency:** Protect message sending, webhook handling, and
    billing/usage recording from accidental duplicate processing.
-   **Auditing:** Record meaningful campaign, automation, usage, and
    status events.
-   **Consent and compliance:** Respect applicable provider policies,
    recipient consent, privacy requirements, and messaging regulations.
-   **Operational safety:** Configure HTTPS, backups, firewall rules,
    monitoring, and recovery procedures before production release.

## 8. Usage and Billing --- Initial Design Requirement

The backend should check the merchant's plan, feature permissions, and
available quota before executing an operation that consumes messages, AI
generation, video processing, or other billable resources.

At the start of implementation, agree on: - Which actions consume
quota. - Whether usage is reserved before processing or recorded after
provider acceptance. - How failed, retried, cancelled, and partially
completed jobs affect usage. - How recharge or plan renewal restores
availability. - How duplicate requests are prevented from charging
twice. - How each usage event is linked to the merchant, module, job,
and provider reference. - How balances, quotas, and usage history are
reconciled.

**Important:** The exact commercial plans, quota amounts, recharge
rules, payment gateway, and approved monetary budget are not finalized
in this planning draft. Design these rules centrally so individual
modules do not implement conflicting billing logic.

## 9. Initial Repository Structure

This is a suggested starting structure, not a statement that these
directories already exist.

``` text
growkyat/
├── README.md
├── docs/
│   ├── project-overview.md
│   ├── architecture.md
│   ├── api-integrations.md
│   ├── security-and-tenancy.md
│   └── usage-and-billing.md
├── frontend/                 # Next.js application
├── backend/                  # Django REST Framework application
├── infrastructure/           # Docker and deployment configuration
├── .gitignore
└── .env.example              # Names/placeholders only; never commit secrets
```

The team can refine this structure once the application entry points and
deployment workflow are agreed upon.


**Document status:** Initial planning draft. Update this file as
decisions are approved and implementation details are verified.
