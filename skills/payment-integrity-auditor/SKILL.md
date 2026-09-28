---
name: payment-integrity-auditor
description: Razorpay HMAC cryptographic signature validation, transaction amount cross-checking, and pay-to-go-live listing gating.
---

# Payment Integrity Auditor Skill

## Overview
The `payment-integrity-auditor` skill protects financial and database integrity by validating Razorpay payment callbacks using SHA-256 HMAC signatures, verifying listing fee amounts, and preventing unauthorized property activations.

## Core Capabilities
- **Server-Side HMAC Verification**: Computes cryptographic HMAC SHA-256 signatures over order and payment IDs using the secret key.
- **Amount Cross-Checking**: Confirms that the paid amount matches the required ₹199.00 fee exactly, preventing zero-payment payload tampering.
- **Gated Database Activation**: Emits signed activation tokens that permit Edge Functions to update property statuses from `Draft` to `Live`.
- **Payment Reconciliation**: Resolves transient webhook dropouts through asynchronous signed status polling.

## Inputs
- `razorpay_order_id`: Razorpay generated order identifier.
- `razorpay_payment_id`: Transaction payment identifier.
- `razorpay_signature`: Cryptographic signature from client/webhook payload.
- `expected_amount_inr`: Mandatory listing fee (₹199.00).

## Outputs
- `is_valid_transaction`: Boolean indicating verified cryptographic authenticity.
- `activation_token`: One-time signed token to activate property in PostgreSQL.
- `audit_record`: Structured transaction payload for financial compliance logging.
