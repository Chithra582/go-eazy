---
name: landlord-compliance-guardian
description: Local service provider document triage, Row-Level Security (RLS) policy enforcement, anti-scraping sentinels, and master admin auditing.
---

# Landlord Compliance Guardian Skill

## Overview
The `landlord-compliance-guardian` skill safeguards platform trust and security, auditing Supabase Row-Level Security (RLS) policies, inspecting service provider registration documents, blocking automated content scraping, and compiling admin dossiers.

## Core Capabilities
- **Service Provider Document Triage**: Audits business registrations, FSSAI certificates, and government IDs for local student service contractors (laundry, mess, cleaning).
- **Row-Level Security Auditing**: Verifies that PostgreSQL tenancy isolation policies prevent unauthorized horizontal privilege escalation between landlords and renters.
- **Anti-Scraping & Content Protection**: Monitors client telemetry to detect automated scraping bots, mass pagination, or unauthorized content extraction attempts.
- **Admin Review Dossiers**: Compiles structured review packets for Master System Administrators to approve or reject provider accounts.

## Inputs
- `provider_application`: Business documentation and identity proofs.
- `audit_scope`: Verification focus (`service_provider_approval`, `rls_policy_audit`, `scraping_investigation`).

## Outputs
- `compliance_decision`: Recommended status (`APPROVED`, `REJECTED`, `FLAGGED_FOR_MANUAL_REVIEW`).
- `risk_assessment`: Security risk score ($0-100$).
- `admin_notes`: Summary of document findings and fraud indicators.
