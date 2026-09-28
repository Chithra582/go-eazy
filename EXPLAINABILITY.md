# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **GoEazy Agent** (`go-eazy-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** GoEazy Agent (`go-eazy-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Education / Student Housing, Rental Market Integrity & Consumer Protection  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), FERPA, GDPR  

---

## How the Agent Decides

GoEazy Agent is an autonomous student housing verification, rental listing orchestration, broker-free landlord triage, and payment integrity agent designed for **GoEazy** (The Housing Standard for Uttarakhand). The agent coordinates property listing validation, server-side Razorpay HMAC signature verification, zero-trust Supabase Row-Level Security (RLS), real-time debounced property discovery, and local service provider verification.

### 1. Decision Architecture

The property onboarding, payment verification, search indexing, and administrative triage pipeline operates across a deterministic, five-stage architecture:

```
Landlord / Student Action (Submit Listing / Pay Listing Fee / Search Housing / Apply as Service Provider)
    │
    ▼
[Stage 1: Listing Ingestion & Anti-Broker Triage]
    │  - Extracts property metadata: location coordinates, rent, deposit, room configuration
    │  - Evaluates broker flags: analyzes contact numbers against known commercial broker blacklists
    │  - Audits image authenticity: detects competitor watermarks and stock imagery
    ▼
[Stage 2: Payment Gating & HMAC Cryptographic Validation]
    │  - Intercepts Razorpay payment callback: computes SHA-256 HMAC signature
    │  - Cross-verifies transaction amount: strictly requires ₹199.00 payment
    │  - If HMAC is valid and amount matches ➔ Grants verified listing placement token
    │  - If signature or amount mismatch ➔ Rejects transaction and halts activation
    ▼
[Stage 3: Zero-Trust Database Commit & RLS Isolation]
    │  - Executes atomic transaction in Supabase PostgreSQL via server-side Edge Functions
    │  - Enforces tenant isolation: ensures landlord only mutates authenticated property rows
    │  - Automatically stages 72-hour rolling window maintenance for viewing history
    ▼
[Stage 4: Real-Time Search Indexing & ZLS Rendering]
    │  - Updates debounced property discovery index with geo-distance and amenity tags
    │  - Computes pre-calculated layout aspect ratios to prevent Zero Layout Shift (ZLS)
    │  - Caches high-frequency search vectors at edge locations
    ▼
[Stage 5: Service Provider & Dispute Administration]
    │  - Processes service provider verification dossiers (FSSAI, identity proofs)
    │  - Computes composite Marketplace Trust Score ($T_{\text{market}} \in [0, 100]$)
    │  - Routes sensitive verifications to Master System Admin Panel for final sign-off
    ▼
Verified, Broker-Free Housing Accommodation Published to Students
```

### 2. Marketplace Scoring & Payment Verification Formulations

GoEazy Agent evaluates listing integrity and transaction validity through deterministic mathematical formulas:

1. **Cryptographic Payment Integrity Verification**:
   A payment is valid if and only if the server-computed HMAC signature matches the Razorpay payload signature, and the transaction amount satisfies the listing fee constraint:
   $$\text{Verified}(\text{txn}) \iff \left(\text{HMAC}_{\text{SHA256}}(\text{order\_id} \mathbin{\Vert} \text{payment\_id}, \, K_{\text{secret}}) = \sigma_{\text{payload}}\right) \land (\text{amount} = \text{₹}199.00)$$

2. **Composite Property Trust Score ($T_{\text{market}}$)**:
   $$T_{\text{market}} = (w_p \cdot P_{\text{photo}}) + (w_l \cdot L_{\text{landlord}}) + (w_a \cdot A_{\text{amenities}}) + (w_t \cdot T_{\text{transparency}})$$
   where:
   - $P_{\text{photo}} \in [0, 100]$: Authentic, unwatermarked high-definition room photos.
   - $L_{\text{landlord}} \in [0, 100]$: Direct landlord ownership verification (absence of broker history).
   - $A_{\text{amenities}} \in [0, 100]$: Explicit student living amenities (Wi-Fi, water, backup, heating).
   - $T_{\text{transparency}} \in [0, 100]$: Full disclosure of security deposit and notice period terms.
   - Weights: $w_p = 0.30, w_l = 0.30, w_a = 0.20, w_t = 0.20$ ($\sum w_i = 1.0$).

### 3. Thresholding & Refusal Decision Criteria

GoEazy Agent deterministically enforces platform integrity:
- **Refusal on Broker Intermediary Detection**: Contact entries linked to third-party commercial broker syndicates charging tenant commissions are refused with code `ERR_BROKER_PROHIBITED`.
- **Refusal on Invalid Payment HMAC**: Transactions with mismatched cryptographic signatures or zero-payment payload manipulation fail with code `ERR_PAYMENT_HMAC_MISMATCH`.
- **Refusal on RLS Boundary Breach**: Attempts to access, edit, or delete another landlord's property records fail at the PostgreSQL policy boundary (`ERR_RLS_ACCESS_DENIED`).
- **Refusal on Automated Content Scraping**: IP addresses displaying automated web scraping or rapid pagination are blocked by the content protection engine (`ERR_SCRAPING_DETECTED`).

### 4. Fallback Decision Mechanism

GoEazy Agent maintains platform continuity through layered fallback strategies:
- **Edge Cache Discovery Fallback**: If the central Supabase PostgreSQL database undergoes maintenance, the search discovery layer serves read-only property index snapshots from edge CDN caches.
- **Manual Payment Reconciliation**: If Razorpay webhook delivery experiences upstream network delays, landlords can trigger a manual HMAC status re-check using their unique Razorpay transaction ID.
- **Model Fallback Cascade**: High-level dispute triage and service provider document inspection summaries default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

GoEazy preserves human administrative authority for sensitive marketplace decisions:
- **Master Admin Service Approvals**: Local student service providers (tiffin, laundry, housekeeping) must have their identity and FSSAI documents reviewed by human system administrators before appearing in the directory.
- **Tenant Dispute Resolution**: Tenant reports of property condition misrepresentation trigger immediate human investigation, with the ability to pause landlord payouts and suspend listings.
- **Discretionary Fee Waivers**: University student welfare committees can request sponsored listing fee waivers for accredited university-managed accommodations.

---

## The Data It Uses

GoEazy Agent operates under strict privacy and consumer data protection standards.

### 1. Ingested Input Data

The agent processes only verified real estate and transaction data points:
- **Property Listing Metadata**: Street address, city, geographic coordinates, monthly rental price, security deposit terms, and room amenity tags.
- **Property Imagery**: High-definition digital photographs of living rooms, bedrooms, and washrooms.
- **Payment Telemetry**: Razorpay order IDs, payment IDs, and cryptographic HMAC signatures.
- **Service Provider Credentials**: Business registration certificates, food safety licenses, and provider contact numbers.

### 2. Configuration & Reference Data

- **PostgreSQL Row-Level Security Rules**: Tenancy policies governing landlord, tenant, and administrator access scopes.
- **Razorpay API Configurations**: Server-side webhook secrets, merchant IDs, and fee structures.
- **Regional College Mapping Data**: Geospatial landmarks for Uttarakhand university campuses (Graphic Era, HNBGU Srinagar, DIT, UPES).

### 3. Base Model & Inference Lineage

- **Deterministic Cryptographic & Security Linters**: HMAC SHA-256 calculation, PostgreSQL RLS enforcement, and JWT token validation are executed by deterministic code.
- **AI Triage & Document Analysis Copilot**: High-capability foundation models (`gemini-2.0-flash`, `gpt-4o`, `claude-3-5-sonnet`) utilized exclusively for document summarization, dispute classification, and search intent parsing.
- **Zero Training on User Data**: Student search histories, tenant contact numbers, and landlord financial details are never used for commercial generative AI training.

### 4. Data Privacy, Storage, and Retention

- **FERPA & GDPR Compliance**: In full accordance with FERPA and GDPR (Articles 5, 17, and 28), student tenant data is encrypted at rest and in transit.
- **Rolling Window Data Pruning**: "Recently Viewed" history is automatically purged after 72 hours via PostgreSQL intervals, preventing database bloat and maintaining privacy.
- **Zero Commercial Monetization**: GoEazy never sells student contact data, search queries, or housing preferences to external advertising brokers.

---

## Limitations

Understanding the operational boundaries and technical constraints of GoEazy Agent is essential for realistic real estate operations.

### 1. Physical Premises Condition vs Digital Photo Verification
- **Limitation**: While the agent verifies that uploaded photos are high-definition and free of watermarks, digital images cannot reveal unseen physical issues (e.g., plumbing leaks, dampness, ambient neighborhood noise).
- **Mitigation**: GoEazy strongly encourages students to conduct physical or live video walkthroughs before signing leases and provides a community review feedback system.

### 2. Upstream Payment Gateway Latency & Webhook Drops
- **Limitation**: Transient outages or network congestion in payment gateway providers (Razorpay) can delay instant webhook notifications to Supabase Edge Functions.
- **Mitigation**: The system supports client-initiated signed status polling and asynchronous retry queues to reconcile transactions without double-charging landlords.

### 3. Dynamic Rental Availability Volatility
- **Limitation**: In peak admission seasons (July-August), rooms can be rented out offline within hours, causing temporary delays in online listing availability updates.
- **Mitigation**: Landlords are provided with a 1-tap "Mark as Occupied" dashboard toggle, and unverified listings expire automatically after 30 days unless reconfirmed.

### 4. Localized Regional Pricing Variance
- **Limitation**: Rental pricing in Dehradun varies widely based on seasonal heating, campus proximity, and private power backup availability, complicating universal algorithmic fair-price modeling.
- **Mitigation**: The platform displays neighborhood-specific median price ranges rather than enforcing rigid price caps.

### 5. Content Anti-Scraping vs Legitimate Accessibility Tools
- **Limitation**: Aggressive anti-scraping protections (blocking text selection, right-click, and developer tools) can occasionally interfere with third-party screen readers or translation browser extensions.
- **Mitigation**: The agent ensures core text content maintains valid ARIA labels and semantic HTML markup so accessibility software can read the interface naturally.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Marketplace scoring & HMAC verification formulas | Section 2 | Verified |
| - Thresholding, anti-broker refusal & criteria | Section 3 | Verified |
| - Fallback decision mechanism & edge cache | Section 4 | Verified |
| - Human-in-the-loop & master admin governance | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested property metadata, photos & payments | Section 1 | Verified |
| - Configuration, PostgreSQL RLS & campus maps | Section 2 | Verified |
| - Base model lineage & deterministic cryptography | Section 3 | Verified |
| - Data privacy, 72-hr pruning & FERPA/GDPR | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Physical premises condition vs digital photos | Section 1 | Verified |
| - Upstream payment gateway latency & webhook drops | Section 2 | Verified |
| - Dynamic rental availability volatility in peak season | Section 3 | Verified |
| - Localized regional pricing variance | Section 4 | Verified |
| - Content anti-scraping vs screen reader accessibility | Section 5 | Verified |
