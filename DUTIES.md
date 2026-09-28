# DUTIES.md - GoEazy Operational Responsibilities & Workflows

> **Specification:** OpenGAP Spec 0.1.0  
> **Agent Name:** GoEazy Agent (`go-eazy-agent`)  
> **Lifecycle Stages:** Listing Ingestion, Payment Verification, Discovery Indexing, Service Provider Triage, System Auditing  

---

## 1. Property Listing Ingestion & Verification

- **Listing Intake Parsing**: Ingest property submission payloads from landlords:
  - Property title, physical address, city (Dehradun, Srinagar, etc.), and landmark coordinates
  - Accommodation type: Single Room, Shared PG, 1BHK/2BHK Apartment
  - Verified amenities: Wi-Fi, 24/7 Water, Power Backup, Mess/Food, Heating, Laundry
  - Monthly rent amount, security deposit, and notice period
- **Visual Asset Inspection**: Verify that uploaded property images meet high-definition standards, contain no watermarks from competing broker portals, and feature authentic room shots.
- **Landlord Identity Verification**: Cross-check landlord contact details against public identity documents and previous platform dispute history.

---

## 2. Server-Side Payment & HMAC Integrity Verification

- **Webhook / Signature Verification**: Intercept Razorpay payment callback events and compute expected HMAC SHA-256 signatures:
  $$\text{HMAC}(\text{order\_id} + "|" + \text{payment\_id}, \, \text{secret\_key})$$
- **Transaction Amount Validation**: Verify that the paid amount matches the required ₹199 listing placement fee exactly, preventing zero-payment payload tampering.
- **Atomic State Activation**: Transition property status from `Draft / Pending Payment` to `Active / Live` strictly upon verified cryptographic proof.

---

## 3. Real-Time Property Discovery & Search Indexing

- **Debounced Search Optimization**: Index active properties to support sub-100ms discovery queries filtering by rent range, proximity to campus, room type, and verified amenities.
- **Rolling Window Cache Pruning**: Maintain user "Recently Viewed" history across a 72-hour rolling window in PostgreSQL, automatically pruning stale records to minimize database bloat.
- **Zero Layout Shift (ZLS) Synchronization**: Ensure property card metadata delivers pre-calculated aspect ratios and image placeholders to client frontends to eliminate layout shift.

---

## 4. Local Service Provider Triage & Verification

- **Document Review Pipeline**: Ingest government registrations, FSSAI certificates (for tiffin services), and identity proofs for local student service providers.
- **Admin Verification Packaging**: Compile review packets for System Administrators with document zoom inspections and fraud risk scores.
- **Directory Publishing**: Activate verified service provider cards in the student community services directory upon admin sign-off.

---

## 5. Security Telemetry & Platform Health Auditing

- **Anti-Scraping Sentinel**: Monitor client request patterns for automated scraping, excessive rapid pagination, or unauthorized content extraction.
- **Row-Level Security Auditing**: Periodically audit Supabase database policies to ensure zero tenant data leaks across user and landlord roles.
- **Audit Logging**: Store tamper-evident transaction logs, payment signatures, and verification statuses in the Supabase audit collection.
