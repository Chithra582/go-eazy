---
name: housing-listing-verifier
description: Property listing intake, authentic photo auditing, broker-free direct landlord validation, and amenity verification.
---

# Housing Listing Verifier Skill

## Overview
The `housing-listing-verifier` skill evaluates newly submitted student rental listings, verifying direct landlord ownership, inspecting room photo authenticity, detecting commercial broker watermarks, and auditing student living amenities.

## Core Capabilities
- **Direct Landlord Verification**: Inspects landlord contact credentials against commercial broker blacklists to enforce the broker-free standard.
- **Image Authenticity & HD Check**: Detects watermarks from competing real estate portals, stock imagery, or low-resolution room photos.
- **Amenity Auditing**: Confirms mandatory student living essentials (Wi-Fi, 24/7 Water, Power Backup, Heating, Study Space).
- **Deposit & Notice Term Validation**: Ensures listings provide transparent disclosures of security deposit amounts and exit terms.

## Inputs
- `property_data`: Address, city, rent amount, deposit, room type, and amenity checklist.
- `landlord_id`: Authenticated user ID of the landlord.
- `photo_urls`: Array of uploaded property image URLs.

## Outputs
- `verification_status`: Status outcome (`VERIFIED`, `FLAGGED_BROKER_SUSPECT`, `PHOTO_REVISION_REQUIRED`).
- `amenity_score`: Completeness score for student essential living amenities ($0-100$).
- `compliance_notes`: Actionable feedback for landlord listing optimization.
