# RULES.md - GoEazy Operational Constraints & Guardrails

> **Specification:** OpenGAP Spec 0.1.0  
> **Agent Name:** GoEazy Agent (`go-eazy-agent`)  
> **Enforcement Level:** Mandatory & Deterministic  

---

## 1. Broker-Free & Marketplace Integrity Guardrails

1. **Zero-Brokerage Mandate**: The agent must deterministically flag and refuse any listing where the contact person acts as a commission-charging broker or intermediary rather than the verified property owner or authorized facility manager.
2. **Mandatory Payment Gating**: Property listings cannot be transitioned to active discovery status directly by client requests. Every live listing requires a verified server-side Razorpay HMAC signature and confirmed payment verification.
3. **Price Transparency**: Listings with hidden additional surcharges, bait-and-switch pricing, or undisclosed mandatory brokerage fees are immediately suspended pending administrative review.

---

## 2. Zero-Trust Database & Security Constraints

1. **Row-Level Security (RLS) Enforcement**: Supabase PostgreSQL queries must strictly enforce user tenancy isolation. No landlord or tenant may read or mutate another user's private records without explicit database policy authorization.
2. **Content Protection & Anti-Scraping**: The agent strictly enforces content integrity rules: programmatic scraping, unauthorized image downloads, and context-menu injection attacks are blocked globally.
3. **Cryptographic Token Verification**: User sessions must utilize verified ES256/RS256 JWT tokens. Expired, spoofed, or unauthenticated tokens trigger immediate session termination.

---

## 3. Data Governance & Student Privacy Standards

1. **FERPA & GDPR Compliance**: Student user profiles, search histories, and payment receipts are classified as sensitive personal data with encryption at rest and in transit.
2. **Rolling Window Data Pruning**: "Recently Viewed" user records must be pruned automatically via a 72-hour rolling window to prevent excessive data retention and preserve query performance.
3. **Right to Erasure**: Students and landlords have the right to request complete account and listing deletion with 0-byte database purging.

---

## 4. Human-in-the-Loop & Administrative Governance

1. **Master System Admin Review**: Provider document approvals (e.g., local laundry, tiffin services, maintenance contractors) require human system administrator sign-off before public directory listing.
2. **Disputed Listing Escalation**: In the event of a tenant complaint regarding property misrepresentation or landlord misconduct, the listing is quarantined into an administrative dispute triage queue.
