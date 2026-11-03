# Product Specification & Technical Documentation
## Lead Intelligence & Attribution Platform

**Document type:** Product Requirements + System Architecture + UX/UI Design Specification  
**Status:** Final product direction  
**Release strategy:** Version 1 — Form Lead Intelligence | Version 2 — Call Tracking & Intelligence  
**Working product name:** Not finalized

---

# 1. Product Overview

## 1.1 Product Vision

Build a centralized lead intelligence and attribution platform that helps individual businesses understand which enquiries are genuine, which enquiries are worth following up, and which marketing activities generate qualified leads.

Businesses receive enquiries through different websites, forms and eventually phone calls. These enquiries often contain spam, irrelevant messages, duplicate submissions and low-quality leads.

The platform collects these enquiries, analyzes them using predefined rules and AI, classifies them, and provides a centralized dashboard for managing and understanding lead quality.

It also sends qualified lead events to Google Analytics 4 (GA4), allowing businesses to measure meaningful enquiries rather than simply counting every form submission.

## 1.2 Core Value Proposition

**Stop counting every enquiry as a lead. Start measuring the enquiries that actually matter.**

The platform provides:

- Centralized lead collection.
- AI-assisted lead qualification.
- Spam and duplicate detection.
- Lead classification and review.
- Multi-website management.
- Lead source and attribution tracking.
- Qualified lead reporting.
- GA4 qualified-lead event integration.
- Future call tracking and call intelligence.

## 1.3 What This Product Is Not

The product should not become a complicated CRM or marketing automation platform.

It is not intended to replace:

- Existing website form plugins.
- Existing CRM systems.
- Google Analytics.
- Business phone systems.
- Email marketing platforms.
- Customer support software.

Its primary responsibility is **lead collection, intelligence, qualification and attribution**.

---

# 2. Product Scope & Release Strategy

## Version 1 — Form Lead Intelligence

The first version focuses on website form enquiries.

### Included

1. Business account management.
2. Multiple website management.
3. Form registration and management.
4. WordPress plugin integration.
5. External website webhook/API integration.
6. Centralized form submission collection.
7. AI-based lead analysis using Gemini.
8. Rule-based spam and duplicate detection.
9. Lead classification.
10. Manual lead review and classification overrides.
11. Lead dashboard and reporting.
12. Website-level GA4 integration.
13. Qualified lead event tracking.
14. Integration logs and delivery monitoring.
15. Account settings and security.

### Not included in Version 1

- Phone number provisioning.
- Incoming call routing.
- Call forwarding.
- Call recording.
- Call transcription.
- Call qualification.
- Dynamic Number Insertion (DNI).
- Telephony provider billing.

These are reserved for Version 2.

## Version 2 — Call Tracking & Intelligence

Call tracking will extend the existing lead intelligence platform.

### Planned capabilities

- Dedicated tracking numbers for each website.
- Multiple tracking numbers for campaigns and branches.
- Platform-managed Twilio integration.
- Business-specific telephony subaccounts.
- Incoming call forwarding.
- Call logs and call status.
- Call recording, subject to applicable consent requirements.
- Secure recording storage.
- Call transcription.
- Gemini-powered call analysis.
- Call-based lead qualification.
- Form and call interaction linking.
- Call attribution.
- Qualified call events sent to GA4.

### Important architectural requirement

Version 1 must already support the concept of different lead sources and interactions.

A form submission and a phone call should eventually be different interaction types connected to a common lead/contact model.

This prevents the need to rebuild the database and dashboard when Version 2 is introduced.

---

# 3. Account & Data Structure

The platform will use a multi-tenant architecture.

## 3.1 Main Hierarchy

**Business Account → Websites → Forms → Submissions → Leads**

A business account can manage multiple websites. Each website can have multiple forms, and each form can receive multiple submissions.

### Example

Business Account: ABC Consulting

- Website 1: abcconsulting.com
  - Contact Form
  - Consultation Form
  - Service Enquiry Form

- Website 2: abcservices.com
  - Contact Form
  - Quote Request Form

All submissions are stored centrally but remain associated with their original business, website and form.

## 3.2 Core Relationships

Every form submission must contain:

- Business ID
- Website ID
- Form ID
- Submission ID
- Submission timestamp
- Original submission data
- Source and attribution data, when available

Every website has its own integration configuration.

Forms belonging to the same website share that website's GA4 configuration.

## 3.3 Multi-Tenant Isolation

All business-owned records must be scoped to the authenticated business account.

The backend must never rely on a client-provided business ID as proof of authorization.

Business access must be derived from the authenticated user's membership and validated server-side.

The database and API must enforce tenant boundaries consistently.

### Future compatibility

The data model should support multiple team members per business in the future, even if Version 1 initially provides only a simple account-owner experience.

---

# 4. Main Product Modules

## 4.1 Business Account

Responsible for the business's main account information.

Fields:

- Business name
- Business email
- Business website
- Time zone
- Country
- Account status
- Created date

## 4.2 Website Management

Each website is an independent configuration unit.

Fields:

- Website name
- Website URL
- Website domain
- Website status
- Connected forms
- API integration status
- GA4 configuration
- Tracking settings
- Website-specific qualification settings

A business should be able to add, edit, disable and remove websites.

Removing a website must not automatically delete its historical lead records.

## 4.3 Form Management

Forms are registered under individual websites.

Each form should have:

- Form name
- Form identifier
- Website association
- Integration type
- Form provider
- Status
- Submission count
- Last received submission
- Created date

Integration types:

1. WordPress plugin.
2. External webhook.
3. Custom REST API.

The platform should not attempt to rebuild the customer's existing forms.

It should receive a copy of successfully submitted form data.

## 4.4 Lead Collection

All incoming submissions must be stored before AI analysis begins.

The original submission is the source record and must remain available even if AI processing fails.

A submission should contain:

- Contact name
- Email
- Phone number
- Message
- Form fields
- Website URL
- Form identifier
- Submission timestamp
- IP address, only where justified and lawfully collected
- Attribution information
- Processing status

Sensitive information must not be unnecessarily exposed in logs or analytics integrations.

## 4.5 Lead Intelligence

This module determines whether an enquiry is relevant and worth following up.

Use a hybrid approach:

**Deterministic rules + Gemini AI + Backend validation**

Do not rely on Gemini alone for all classification decisions.

### Deterministic processing

The backend should handle obvious cases:

- Missing required data.
- Invalid email or phone formats.
- Known spam patterns.
- Repeated submissions.
- Internal test submissions.
- Duplicate detection.
- Configured blocklists.

### AI processing

Gemini should analyze:

- Enquiry intent.
- Requested service.
- Business relevance.
- Customer requirements.
- Urgency.
- Lead quality.
- Summary.
- Recommended next action.
- Classification confidence.

AI responses must use structured JSON output and pass backend validation before being stored.

### Classification categories

| Category | Meaning |
|---|---|
| Qualified | Relevant enquiry with sufficient intent to pursue |
| Potential | Relevant but requires further review or information |
| Spam | Clearly irrelevant or malicious submission |
| Internal Test | Test or internal submission |
| Duplicate | Repeated interaction associated with an existing record |
| Not a Lead | Non-commercial, irrelevant or unrelated message |

A low-confidence classification should go to manual review.

A genuine enquiry is not automatically a qualified lead. A qualified lead is not automatically a paying customer.

### Classification history

Every classification should preserve:

- Previous classification.
- New classification.
- Classification source (rule, AI or manual).
- Confidence where applicable.
- Reason.
- Timestamp.
- User responsible for manual changes, when applicable.

Manual overrides must not erase the original AI result.

---

# 5. Optimized Lead Processing Workflow

The processing flow should be asynchronous and resilient.

## 5.1 Form Submission Flow

1. A visitor submits a form on the business website.
2. The existing form plugin validates and processes the submission.
3. Once successfully submitted, the integration forwards a copy to the platform.
4. The platform validates the incoming request.
5. It verifies the website and form integration.
6. It checks the idempotency key to prevent duplicate ingestion.
7. It stores the original submission.
8. It creates a processing job.
9. Deterministic validation and spam rules run.
10. Gemini analyzes the enquiry where required.
11. The backend validates and stores the classification.
12. The lead appears in the dashboard.
13. If qualified, a GA4 event is queued.
14. The GA4 worker attempts delivery and records the result.

## 5.2 Critical Reliability Rule

The platform must never become a dependency for the original form submission.

If the platform is unavailable:

- The customer's form must continue working.
- Existing email notifications must continue.
- Existing form integrations must continue.
- The integration should retry sending the submission later.

The WordPress plugin should use non-blocking background delivery, retries and idempotency.

## 5.3 Processing Statuses

Keep processing status separate from lead classification.

Suggested processing statuses:

- Received
- Pending Analysis
- Processing
- Completed
- Needs Review
- Failed

This avoids confusing a technical processing failure with a lead-quality decision.

## 5.4 Retry Strategy

Use a database-backed job queue initially.

Implement:

- Retry with exponential backoff.
- Maximum retry attempts.
- Failed job tracking.
- Manual retry where appropriate.
- Idempotent job execution.
- Processing timestamps.

Redis and BullMQ can be introduced later if actual throughput requires them.

---

# 6. Lead & Contact Management

The platform should distinguish between a lead, a contact and an individual interaction.

## 6.1 Contact

Represents a person or organization that has made an enquiry.

## 6.2 Lead

Represents a potential business opportunity.

## 6.3 Interaction

Represents an individual enquiry or communication event.

Version 1 interaction types:

- Form Submission

Version 2 planned interaction types:

- Incoming Call
- Missed Call
- Other supported communication sources

A contact or lead may have multiple interactions.

For example, the same person may submit a consultation form twice and later call the business.

The platform should preserve all three interactions while linking them to the same contact where identity matching is sufficiently reliable.

Do not merge records solely because their names are similar.

Duplicate detection should support:

- Exact email matching.
- Normalized phone matching.
- Time-based matching.
- Website and form context.
- Configurable matching rules.

Uncertain matches should be reviewed rather than automatically merged.

---

# 7. Attribution & GA4 Integration

## 7.1 Integration Strategy

GA4 integration is configured at the website level.

Each website can have its own GA4 Measurement ID and API secret.

All forms connected to that website use the same GA4 configuration.

For the initial release, support one primary GA4 destination per website.

## 7.2 Event Strategy

Only qualified leads should be sent to GA4.

Recommended event:

`generate_lead`

Suggested event parameters:

| Parameter | Example |
|---|---|
| lead_source_type | form |
| service_type | consultation |
| form_type | contact |
| website_id | non-PII identifier |

Never send the following to GA4:

- Names
- Email addresses
- Phone numbers
- Messages
- Form contents
- Call recordings
- Transcripts
- Other personally identifiable information

The platform database remains the source of truth for actual lead records.

## 7.3 Attribution Data

Where consent and implementation allow, collect:

- GA4 client ID.
- GA4 session ID.
- Landing page URL.
- Page URL.
- UTM source.
- UTM medium.
- UTM campaign.
- Referrer.
- Submission timestamp.

The WordPress integration or custom website implementation should capture available attribution context when the visitor submits the form.

Attribution must not be fabricated when identifiers are unavailable.

## 7.4 GA4 Delivery

Use GA4 Measurement Protocol from the backend.

The API secret must remain server-side and must never be exposed in frontend JavaScript.

GA4 event delivery should happen asynchronously.

Store:

- Event name.
- Website ID.
- Associated lead ID.
- Payload status.
- Attempt count.
- Last attempt.
- Delivery result.
- Error details.

Implement retry handling and idempotency to reduce duplicate event delivery.

GA4 reporting and attribution are not guaranteed to be perfect. Consent, browser restrictions, missing session information and measurement limitations can affect the results.

---

# 8. Integration Architecture

## 8.1 WordPress Integration

A dedicated WordPress plugin will connect WordPress websites to the platform.

The plugin should support popular form providers through separate adapters:

- Contact Form 7
- WPForms
- Gravity Forms
- Elementor Forms
- Divi Contact Form
- Fluent Forms

Each adapter should detect successful submissions and forward the data to the platform.

The plugin must not replace or interrupt the existing form-processing workflow.

### WordPress plugin responsibilities

- Platform API connection.
- Website verification.
- Form provider detection.
- Successful submission capture.
- Field mapping.
- Attribution data capture where available.
- Background delivery.
- Retry handling.
- Submission idempotency.
- Connection diagnostics.

The plugin should have a simple configuration interface.

Avoid unnecessary WordPress admin complexity.

## 8.2 External Website Integration

Non-WordPress websites should connect using:

- Webhooks.
- REST API.
- Server-side integrations.

A website should be able to send successfully processed submissions to the platform.

Do not expose privileged API secrets in browser-side JavaScript.

Public ingestion credentials, where required, must be restricted and protected through origin controls where possible, rate limiting, abuse detection and request validation.

## 8.3 Shared Ingestion Engine

All integrations should use the same backend ingestion service.

WordPress submissions and external website submissions must follow the same validation, storage, classification and reporting workflow.

Do not build separate lead-processing systems for different integrations.

---

# 9. Version 2 Call Tracking Architecture

Call tracking is intentionally excluded from the initial release, but the system must be designed to accommodate it.

## 9.1 Future Call Structure

The planned hierarchy:

Business → Website → Tracking Number → Calls → Call Analysis

Recommended initial approach:

- One dedicated tracking number per website.
- Additional numbers for separate campaigns or branches later.
- Number assignment managed by the platform.

A shared tracking number across unrelated websites would weaken website-level attribution.

Dynamic Number Insertion can be introduced in a later phase for visitor-level and campaign-level call attribution.

## 9.2 Telephony Provider

Initial planned provider: Twilio Voice.

Use a provider adapter so the platform is not permanently tied to one telephony provider.

The platform-managed model should use a separate provider subaccount for each business.

Important: numbers provisioned under the platform's provider account are assigned for business use; this is not automatically the same as direct legal ownership by the business. Define number release, portability and exit procedures before launch.

## 9.3 Future Call Workflow

1. A customer calls the assigned tracking number.
2. The telephony provider receives the call.
3. The provider forwards the call to the business's existing number.
4. Call status webhooks are received.
5. Call metadata is stored.
6. Recording status is received separately.
7. Recording is stored securely when permitted.
8. Transcription is generated.
9. Gemini analyzes the transcript.
10. The backend classifies the call.
11. The call appears in the dashboard.
12. Qualified calls can generate GA4 events.

Call routing must be designed to continue wherever possible even if the platform's dashboard or AI processing is temporarily unavailable.

## 9.4 Call Status vs Lead Classification

These must remain separate.

Call statuses:

- Ringing
- Answered
- Missed
- Busy
- Failed
- Completed

Lead classifications:

- Qualified
- Potential
- Spam
- Not a Lead

A missed call is not automatically spam or a qualified lead. It represents an interaction that may require follow-up.

## 9.5 Recording & Privacy

The product direction is recording enabled by default for eligible calls.

However, recording must always comply with applicable laws, consent requirements and provider restrictions.

If recording is not legally permitted, do not record. Preserve only the call metadata that can lawfully be collected.

Store recordings in private object storage, not directly in PostgreSQL or publicly accessible URLs.

The database should store:

- Recording object key.
- Recording duration.
- Recording status.
- Transcription status.
- Retention date.
- Access metadata.

Provide appropriate retention and deletion controls.

---

# 10. Recommended Technical Architecture

## 10.1 Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js |
| Backend | Next.js server-side API |
| Language | TypeScript |
| Database | PostgreSQL |
| ORM | Prisma |
| Authentication | Better Auth |
| AI | Google Gemini API |
| Analytics | GA4 Measurement Protocol |
| WordPress Integration | Custom WordPress plugin |
| File Storage | S3-compatible object storage |
| Job Processing | Database-backed queue |
| Future Telephony | Twilio Voice |
| Future Transcription | Dedicated speech-to-text service |

## 10.2 Architecture Principles

- Start with a modular monolith.
- Keep business logic separate from UI components.
- Use shared services for ingestion and classification.
- Keep AI processing asynchronous.
- Use background jobs for external API calls.
- Keep integrations isolated through adapters.
- Use server-side authorization.
- Keep secrets outside frontend code.
- Preserve original records.
- Maintain audit history.
- Build for future expansion without premature microservices.

## 10.3 Main Backend Modules

1. Authentication
2. Business Accounts
3. Websites
4. Forms
5. Integrations
6. Submission Ingestion
7. Lead Management
8. Contact Management
9. AI Classification
10. Spam & Duplicate Detection
11. Attribution
12. GA4 Integration
13. Background Jobs
14. Audit Logs
15. Reporting

Future modules:

16. Telephony
17. Phone Number Management
18. Call Routing
19. Call Recording
20. Transcription
21. Call Intelligence

---

# 11. Database Design

Use PostgreSQL with Prisma.

The following is the recommended logical model. Exact columns, constraints and indexes should be finalized during implementation.

## 11.1 Core Tables

### users
- id
- name
- email
- created_at
- updated_at

### businesses
- id
- name
- timezone
- country
- status
- created_at
- updated_at

### business_memberships
- id
- business_id
- user_id
- role
- created_at

This enables future team access.

### websites
- id
- business_id
- name
- domain
- url
- status
- created_at
- updated_at

### forms
- id
- business_id
- website_id
- name
- external_form_id
- provider
- integration_type
- status
- created_at
- updated_at

### integration_credentials
- id
- business_id
- website_id
- integration_type
- encrypted_credentials
- status
- created_at
- updated_at

### form_submissions
- id
- business_id
- website_id
- form_id
- external_submission_id
- raw_payload
- normalized_payload
- attribution_data
- processing_status
- submitted_at
- created_at

### contacts
- id
- business_id
- name
- email
- phone
- created_at
- updated_at

### leads
- id
- business_id
- contact_id
- primary_source
- classification
- priority
- summary
- status
- created_at
- updated_at

### lead_interactions
- id
- business_id
- lead_id
- source_type
- source_record_id
- occurred_at
- created_at

### ai_classifications
- id
- business_id
- lead_id
- source_record_id
- classification
- confidence
- reason
- ai_model
- structured_output
- created_at

### analytics_integrations
- id
- business_id
- website_id
- measurement_id
- encrypted_api_secret
- status
- created_at
- updated_at

### analytics_events
- id
- business_id
- website_id
- lead_id
- event_name
- event_payload
- status
- attempts
- last_attempt_at
- delivered_at
- created_at

### provider_webhook_events
- id
- provider
- external_event_id
- event_type
- processing_status
- received_at
- processed_at

### audit_logs
- id
- business_id
- user_id
- action
- entity_type
- entity_id
- metadata
- created_at

## 11.2 Future Version 2 Tables

Design these when implementing call tracking:

- phone_numbers
- call_routes
- calls
- call_recordings
- call_transcripts
- telephony_accounts
- call_analysis

Calls should be independent interaction records linked to leads where appropriate.

Avoid creating a generic database structure that makes the Version 1 implementation unnecessarily complicated just to anticipate every possible Version 2 requirement.

---

# 12. Application Navigation & UX Structure

The dashboard should be designed for individual businesses, not agency account management.

The interface should feel like a modern analytics and operations product.

## 12.1 Main Navigation

Recommended sidebar:

1. Overview
2. All Leads
3. Form Leads
4. Websites
5. Analytics & Reports
6. Integrations
7. Settings

Future Version 2 navigation:

8. Call Leads

Call tracking should not appear as a fully functional module in Version 1.

It can be introduced in Version 2 without restructuring the navigation.

## 12.2 Overview Dashboard

The overview should answer four questions immediately:

1. How many enquiries have we received?
2. How many were qualified?
3. How many were rejected or identified as spam?
4. Which website or source is generating the best leads?

### Recommended KPI cards

- Total Enquiries
- Qualified Leads
- Potential Leads
- Spam / Not a Lead

### Dashboard sections

- Lead volume trend.
- Qualification rate.
- Lead classification breakdown.
- Leads by website.
- Leads by form.
- Recent enquiries.
- Integration health.

Use date filters consistently across reporting widgets.

Do not overload the dashboard with too many charts.

## 12.3 All Leads

The central lead management table.

Recommended columns:

- Contact
- Website
- Source
- Form
- Classification
- Priority
- Received
- Actions

Features:

- Search.
- Date range filtering.
- Website filtering.
- Classification filtering.
- Source filtering.
- Sorting.
- Pagination.
- Bulk actions where appropriate.

## 12.4 Lead Detail View

The lead detail page should present information in a clear hierarchy.

Recommended layout:

**Main content**

- Contact information.
- Enquiry summary.
- Original submission.
- AI analysis.
- Classification.
- Lead interaction history.

**Secondary panel**

- Website.
- Form.
- Submission date.
- Attribution information.
- Processing status.
- Manual classification controls.

Preserve the original submission and make it possible to inspect the source data.

## 12.5 Websites

List all connected websites.

Each website should have a dedicated settings area:

- General information.
- Connected forms.
- Integration status.
- API configuration.
- GA4 configuration.
- Qualification rules.

## 12.6 Integrations

Provide a central integration area.

Sections:

- WordPress.
- Webhooks.
- REST API.
- Google Analytics 4.

Each integration should show:

- Connection status.
- Setup instructions.
- Last successful activity.
- Last error.
- Configuration actions.

## 12.7 Settings

Initial settings:

- Business profile.
- Account preferences.
- Time zone.
- Security.
- API credentials.
- Data retention.
- Account management.

Team management can be introduced when multi-user access is implemented.

---

# 13. Product Design System

The UI should follow a clean, premium SaaS design language.

The objective is clarity and usability, not decorative complexity.

## 13.1 Design Direction

Recommended characteristics:

- Minimal interface.
- Clear visual hierarchy.
- Neutral background surfaces.
- Strong typography.
- Consistent spacing.
- Subtle borders.
- Restrained use of color.
- Consistent status indicators.
- Compact but readable data tables.
- Responsive layouts.

Avoid excessive gradients, oversized cards, unnecessary illustrations and decorative dashboard elements.

## 13.2 Design Tokens

Establish centralized design tokens before building individual pages.

### Color system

Define semantic tokens rather than hardcoding colors inside components.

- Background
- Surface
- Surface Elevated
- Primary Text
- Secondary Text
- Muted Text
- Border
- Primary Action
- Success
- Warning
- Danger
- Information

Classification colors should remain consistent throughout the application.

Suggested semantic mapping:

| Classification | Semantic color |
|---|---|
| Qualified | Success |
| Potential | Warning |
| Spam | Danger |
| Internal Test | Neutral |
| Duplicate | Information |
| Not a Lead | Neutral |

Color must never be the only way to communicate status. Always include a text label.

### Typography

Use a consistent typography scale.

Recommended hierarchy:

- Page Title
- Section Heading
- Card Heading
- Body
- Supporting Text
- Caption
- Table Metadata

Choose a modern, highly readable sans-serif typeface.

### Spacing

Use a consistent spacing scale based on a 4px or 8px rhythm.

Keep page padding, card padding, table density and section spacing consistent.

### Components

Create reusable components for:

- Buttons.
- Inputs.
- Selects.
- Date filters.
- Search fields.
- Status badges.
- KPI cards.
- Data tables.
- Dropdown menus.
- Dialogs.
- Side panels.
- Empty states.
- Loading states.
- Error states.
- Toast notifications.
- Pagination.
- Tabs.
- Tooltips.

## 13.3 Design Consistency

Every module must use the same design tokens and shared components.

Do not create independent visual styles for Overview, Leads, Websites and Integrations.

The design system should be documented and implemented before the application grows.

## 13.4 UX States

Every major page and component must account for:

- Loading.
- Empty.
- Error.
- Success.
- Disabled.
- Permission restricted.
- No integration configured.
- Integration disconnected.

Do not leave blank areas when a business has no data.

Provide useful onboarding guidance and clear next actions.

## 13.5 Responsive Design

The application must support:

- Desktop.
- Laptop.
- Tablet.
- Mobile.

The desktop experience can prioritize information density.

On mobile, use simplified tables, responsive filters and readable lead detail layouts.

Accessibility requirements:

- Keyboard navigation.
- Visible focus states.
- Semantic HTML.
- Proper labels.
- Sufficient contrast.
- Screen-reader-friendly status indicators.

---

# 14. Security & Production Requirements

Security must be built into the foundation.

## 14.1 Authentication & Authorization

- Use Better Auth.
- Enforce server-side authentication.
- Validate business membership.
- Implement role-based authorization.
- Protect all business-specific API routes.
- Never trust client-provided tenant identifiers.

## 14.2 API Security

- Validate request payloads.
- Use rate limiting.
- Verify integration credentials.
- Apply origin controls where applicable.
- Implement idempotency.
- Protect against replay and abuse.
- Avoid exposing privileged secrets.
- Validate webhook signatures where supported.

## 14.3 Data Security

- Encrypt sensitive integration credentials.
- Use HTTPS.
- Avoid logging PII unnecessarily.
- Restrict access to raw submission data.
- Define retention and deletion policies.
- Back up PostgreSQL.
- Use private object storage for future call recordings.

## 14.4 AI Security

Treat all incoming form content as untrusted input.

Gemini must not be allowed to execute arbitrary instructions contained in submissions.

Use constrained prompts, structured output and backend validation.

Do not expose internal prompts, API secrets or other customers' information to the model.

## 14.5 Operational Monitoring

Monitor:

- API errors.
- Failed ingestion requests.
- Background job failures.
- Gemini API errors.
- GA4 delivery failures.
- Integration health.
- Database performance.
- Queue processing time.

Provide enough logging to troubleshoot problems without storing unnecessary personal data.

---

# 15. API Design

Use versioned API routes.

Base path:

`/api/v1`

Recommended initial endpoints:

### Business

- `GET /business`
- `PATCH /business`

### Websites

- `GET /websites`
- `POST /websites`
- `GET /websites/:id`
- `PATCH /websites/:id`
- `DELETE /websites/:id`

### Forms

- `GET /websites/:id/forms`
- `POST /websites/:id/forms`
- `PATCH /forms/:id`
- `DELETE /forms/:id`

### Submission Ingestion

- `POST /ingest/form-submission`

### Leads

- `GET /leads`
- `GET /leads/:id`
- `PATCH /leads/:id`
- `POST /leads/:id/reclassify`

### Analytics

- `GET /websites/:id/analytics`
- `POST /websites/:id/analytics`
- `PATCH /websites/:id/analytics`

### Integrations

- `GET /integrations`
- `GET /integrations/:id/status`
- `POST /integrations/:id/test`

The actual API contracts must define request validation, response schemas, authentication, authorization and error codes.

External ingestion endpoints should use restricted integration credentials rather than normal user session authentication.

---

# 16. Onboarding Flow

The initial onboarding should be simple and focused.

## Step 1 — Create Account

Register and authenticate the business owner.

## Step 2 — Create Business

Collect:

- Business name.
- Business website.
- Country.
- Time zone.

## Step 3 — Connect Website

Register the first website.

## Step 4 — Connect Forms

Choose an integration method:

- WordPress.
- Webhook.
- Custom API.

## Step 5 — Configure GA4

Connect the website's GA4 Measurement ID and API secret.

Explain that only qualified lead events will be sent.

## Step 6 — Verify Integration

Send a test submission and confirm that it appears in the platform.

## Step 7 — Start Receiving Leads

Display the dashboard with the current integration status and incoming submissions.

Do not force users to complete optional configurations before they can begin receiving leads.

---

# 17. Implementation Roadmap

## Phase 1 — Foundation

- Project setup.
- Design tokens.
- Shared UI components.
- Database schema.
- Authentication.
- Business accounts.
- Tenant isolation.
- Application navigation.

## Phase 2 — Website & Form Management

- Website CRUD.
- Form CRUD.
- Integration credentials.
- WordPress integration architecture.
- Webhook/API ingestion.
- Request validation.
- Idempotency.

## Phase 3 — Lead Intelligence

- Submission storage.
- Deterministic rules.
- Gemini integration.
- Structured classification.
- Duplicate detection.
- Manual review.
- Classification history.

## Phase 4 — Dashboard

- Overview.
- All Leads.
- Lead details.
- Website management.
- Integration status.
- Reporting.
- Search and filters.

## Phase 5 — GA4 Integration

- Website-level configuration.
- Attribution capture.
- Qualified lead event creation.
- Measurement Protocol delivery.
- Retry handling.
- Delivery logs.

## Phase 6 — Production Hardening

- Security review.
- Tenant isolation tests.
- Error handling.
- Background job reliability.
- Rate limiting.
- Data retention.
- Backup strategy.
- Monitoring.
- Performance optimization.
- End-to-end testing.

## Phase 7 — Version 2: Call Tracking

- Telephony provider adapter.
- Business-specific provider subaccounts.
- Phone number provisioning.
- Number assignment.
- Call forwarding.
- Call webhooks.
- Call logs.
- Recording.
- Secure storage.
- Transcription.
- Gemini call analysis.
- Lead interaction linking.
- Qualified call events.
- Advanced call attribution.

---

# 18. Definition of Production Readiness

Version 1 should not be considered production-ready simply because the dashboard and AI classification work.

The following must be verified.

### Data integrity

- Every submission is associated with the correct business, website and form.
- Original submission data is preserved.
- Duplicate ingestion is prevented.
- Classification history is maintained.
- Manual changes are auditable.

### Integration reliability

- Existing website forms continue working if the platform is unavailable.
- Failed forwarding is retried.
- API credentials are protected.
- Integration status is visible.
- Invalid submissions are rejected safely.

### AI reliability

- Gemini output follows a validated schema.
- AI failures do not lose submissions.
- Low-confidence results are reviewable.
- AI decisions can be manually overridden.
- AI does not make irreversible database decisions.

### Analytics reliability

- Only qualified leads are sent to GA4.
- No PII is sent to GA4.
- GA4 credentials remain server-side.
- Events are retried safely.
- Attribution is recorded only when available.

### Application quality

- Tenant isolation is tested.
- Authorization is enforced server-side.
- Loading and error states are implemented.
- Mobile layouts are usable.
- Accessibility is considered.
- Database backups and monitoring are configured.

---

# 19. Final Product Principles

These principles should guide every implementation decision.

1. **Simplicity first:** Build a focused lead intelligence product, not a full CRM.
2. **Preserve original data:** Never allow AI classification to replace or destroy the original enquiry.
3. **AI assists, backend decides:** Business rules and validation remain under application control.
4. **Asynchronous processing:** External services must not block the original form submission.
5. **One source of truth:** PostgreSQL stores the platform's business and lead records.
6. **Website-level configuration:** Each website owns its integration and GA4 settings.
7. **Qualified leads only:** GA4 receives qualified lead events, not every enquiry.
8. **Privacy by design:** Never send personal lead information to analytics.
9. **Multi-tenant security:** Every business must remain isolated.
10. **Future-ready architecture:** Call tracking extends the same lead and interaction model.
11. **Modular implementation:** Use a modular monolith, not premature microservices.
12. **Consistent UX:** Every page must follow the shared design system.

---

# 20. Final Product Definition

The product is a centralized lead intelligence and attribution platform for individual businesses.

Version 1 collects website form enquiries, stores original submissions, analyzes their quality using deterministic rules and Gemini AI, classifies leads, and reports qualified lead activity through a centralized dashboard and GA4 integration.

Version 2 extends the same platform with business-assigned tracking numbers, call forwarding, call recording, transcription, AI-powered call qualification and call attribution.

The product should remain focused on one outcome:

**Helping businesses understand the quality and source of their enquiries, rather than simply counting how many enquiries they receive.**

This document is the baseline for product design, database implementation, API development, frontend development and release planning.
