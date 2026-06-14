# Ethos Life - GraphQL Schema

## Overview

Ethos Life operates a Partnership API that allows approved business partners to embed quoting, identity validation, instant underwriting, application capture, billing, and signed policy issuance directly into their own products. This conceptual GraphQL schema models the full domain of the Ethos Partnership API.

The schema is derived from public documentation available at https://partners.ethoslife.com/api/ and reflects the core workflows: applicant creation, instant quoting, health assessment, underwriting decisions, offer acceptance, policy issuance, and claims management.

## Authentication

The Ethos Partnership API uses a two-layer bearer-token authentication model:

- **Partner API Key** - A long-lived credential issued per partner after a partnership agreement is signed. Used to authenticate partner-level operations.
- **Per-Applicant Session Token** - A short-lived token scoped to a single applicant session, generated via the `createToken` mutation. Required for all applicant-facing operations.

There is no public, self-serve developer signup. Access requires a partnership agreement with Ethos.

## Core Workflows

### 1. Applicant Onboarding

Create or retrieve an `Applicant` record with demographic and contact details. The `upsertApplicant` mutation accepts minimal required fields (name, date of birth, state of residence) to start a session.

### 2. Instant Quoting

Call `createQuote` with coverage amount, term length, product type, and state. Ethos combines identity validation and quoting into a single round-trip to enable instant decisioning for eligible applicants. The resulting `Quote` includes multiple `QuoteTerms` options across available term lengths and face amounts.

### 3. Health Assessment

Retrieve applicable `healthQuestions` for the selected product type. Submit applicant responses via `submitHealthAnswers`. Responses feed into the `RiskAssessment` computation, which determines whether a medical exam is required and what health class the applicant qualifies for.

### 4. Application Submission

Create an `Application` linking the applicant and selected quote, then call `submitApplication` to trigger underwriting. Applications may receive an instant `UnderwritingDecision` or enter a review queue.

### 5. Offer Acceptance

When underwriting produces an approved decision, an `Offer` is generated. The offer contains final pricing, health class, and any modifications from the applied terms. Partners present the offer to the applicant and call `acceptOffer` to proceed to policy issuance.

### 6. Policy Issuance

Upon offer acceptance and payment setup, a `Policy` is issued with a policy number, effective date, and associated `PolicyDocument` records. Beneficiary information can be added via `addBeneficiary` or `updateBeneficiaries`.

### 7. Claims

Policyholders or beneficiaries may file a `Claim` against an active policy via `submitClaim`. The claim progresses through review stages tracked by `ClaimStatus`. Approved claims result in `Payment` records disbursed to the named beneficiaries.

### 8. Webhooks

Partners register `Webhook` endpoints to receive real-time `WebhookEvent` notifications for key lifecycle events including application decisions, policy issuance, payment status changes, and claim outcomes.

## Schema Reference

### Named Types (55+)

| Type | Category | Description |
|---|---|---|
| `Application` | Application | A life insurance application |
| `ApplicationDetails` | Application | Metadata captured during application |
| `ApplicationStatus` | Application | Current status with history |
| `ApplicationStatusHistory` | Application | Historical status change record |
| `Quote` | Quote | A generated insurance quote |
| `QuoteDetails` | Quote | Carrier and product details for a quote |
| `QuoteTerms` | Quote | Term/coverage options within a quote |
| `Policy` | Policy | An issued life insurance policy |
| `PolicyDetails` | Policy | Carrier, product, and policy metadata |
| `PolicyStatus` | Policy | Current status and lapse tracking |
| `PolicyDocument` | Policy | Documents associated with a policy |
| `Coverage` | Coverage | Coverage type and amounts |
| `CoverageAmount` | Coverage | Monetary coverage amount with bounds |
| `Term` | Coverage | Policy term with start/end dates |
| `TermLength` | Coverage | Duration of a term in years/months |
| `Premium` | Premium | Summary premium amounts |
| `PremiumDetails` | Premium | Full premium breakdown with modal factors |
| `Beneficiary` | Beneficiary | A named beneficiary on a policy |
| `BeneficiaryDetails` | Beneficiary | Personal and legal beneficiary info |
| `BeneficiaryType` | Beneficiary | Classification (primary, contingent, etc.) |
| `Applicant` | Applicant | An individual applying for coverage |
| `ApplicantDetails` | Applicant | Contact and identity information |
| `ApplicantDemographics` | Applicant | Health and demographic underwriting data |
| `DateOfBirth` | Applicant | Structured date of birth |
| `Height` | Applicant | Height in feet/inches and centimeters |
| `Weight` | Applicant | Weight in pounds and kilograms |
| `HealthQuestion` | Health | A question in the health assessment |
| `HealthAnswer` | Health | An applicant's answer to a health question |
| `RiskAssessment` | Underwriting | Computed risk profile for an applicant |
| `RiskFactor` | Underwriting | Individual contributing risk factor |
| `UnderwritingDecision` | Underwriting | Formal underwriting outcome |
| `Offer` | Offer | Insurance offer post-underwriting |
| `OfferDetails` | Offer | Pricing and terms of an offer |
| `OfferExpiry` | Offer | Expiration tracking for an offer |
| `DeathBenefit` | Benefits | Core death benefit amounts |
| `AccidentalDeathBenefit` | Benefits | Accidental death rider details |
| `RiderDetails` | Benefits | Policy rider information and pricing |
| `Claim` | Claims | A filed insurance claim |
| `ClaimDetails` | Claims | Claimant and incident information |
| `ClaimStatus` | Claims | Current claim status |
| `Payment` | Payments | A payment record |
| `Agent` | Distribution | A licensed insurance agent |
| `AgentDetails` | Distribution | Agent contact and license info |
| `Agency` | Distribution | An insurance agency |
| `AgencyDetails` | Distribution | Agency contact and NPN info |
| `Integration` | Partner | Active API integration record |
| `PartnerDetails` | Partner | Business partner metadata |
| `APIKey` | Auth | Partner API key |
| `Token` | Auth | Per-applicant session token |
| `Webhook` | Webhooks | A registered webhook endpoint |
| `WebhookEvent` | Webhooks | A webhook delivery record |
| `Address` | Shared | Mailing/service address |

### Enum Types

- `ApplicationStatusValue` - DRAFT, SUBMITTED, IN_REVIEW, APPROVED, DECLINED, CANCELLED, EXPIRED
- `PolicyStatusValue` - PENDING_ISSUE, ACTIVE, LAPSED, CANCELLED, MATURED, PAID_UP
- `ClaimStatusValue` - SUBMITTED, IN_REVIEW, APPROVED, DENIED, PAID, WITHDRAWN
- `BeneficiaryTypeValue` - PRIMARY, CONTINGENT, TERTIARY
- `BeneficiaryRelationship` - SPOUSE, CHILD, PARENT, SIBLING, TRUST, ESTATE, CHARITY, and others
- `Gender` - MALE, FEMALE, NON_BINARY
- `TobaccoStatus` - NEVER, FORMER, CURRENT
- `UnderwritingDecisionValue` - APPROVED_AS_APPLIED, APPROVED_WITH_MODIFICATION, RATED, DECLINED, POSTPONED, PENDING_MEDICAL_INFO, INSTANT_DECISION
- `WebhookEventType` - APPLICATION_SUBMITTED, APPLICATION_APPROVED, POLICY_ISSUED, PAYMENT_RECEIVED, CLAIM_SUBMITTED, and others
- `PaymentFrequency` - MONTHLY, QUARTERLY, SEMI_ANNUAL, ANNUAL
- `PaymentMethod` - ACH, CREDIT_CARD, CHECK
- `PaymentStatus` - PENDING, PROCESSED, FAILED, REFUNDED
- `HealthQuestionCategory` - GENERAL_HEALTH, CARDIOVASCULAR, RESPIRATORY, CANCER, MENTAL_HEALTH, and others
- `CoverageType` - TERM_LIFE, WHOLE_LIFE, INDEXED_UNIVERSAL_LIFE, ACCIDENTAL_DEATH

## Coverage Products

Ethos currently offers the following life insurance products through its partner network:

- **Term Life Insurance** - 10, 15, 20, 30, and 40 year terms; face amounts from $15,000 to $3 million; underwritten by Ameritas, Banner Life, Protective, and TruStage
- **Whole Life Insurance** - Permanent coverage with cash value accumulation
- **Indexed Universal Life (IUL)** - Flexible premium permanent coverage with index-linked cash value growth
- **Accidental Death Benefit** - Available as a rider on term life policies

## Available States

Ethos coverage is available in 49 US states and Washington D.C.

## Partner Resources

- Partnership API Documentation: https://partners.ethoslife.com/api/
- Partner FAQ: https://www.ethos.com/faq/partners/
- Agent Portal: https://agents.ethoslife.com/login
- GitHub (Design System): https://github.com/getethos/ethos-design-system
- Engineering Blog: https://techandethos.medium.com
