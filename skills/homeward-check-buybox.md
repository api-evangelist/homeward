---
name: Check whether a property is in Homeward's buybox
description: Run a quick, credential-free eligibility check to see if a property is likely within Homeward's buybox before creating an offer request.
api: openapi/homeward-offer-estimate-openapi.yml
operations: [checkBuyboxEligibility]
---

# Check Homeward buybox eligibility

Use this skill to screen a property before submitting a full offer request. The buybox endpoint is public — it requires no token.

## Steps

1. **Call** `checkBuyboxEligibility` — `POST /buybox/` on `https://api.homeward.com` with `Content-Type: application/json`.
   Body: a `property_address` object where `state` (2-letter) is required and `street_one`, `street_two`, `city`, `postal_code` are optional; plus optional top-level `opinion_of_value`, `lot_size_acres`, `property_type` (Single Family, Condo, Townhouse, ...), `listing_status` (Listed / Not Listed / Unknown), and `year_built`. The more fields you send, the more accurate the result.

2. **Read the result.** A `200` returns `is_eligible` (boolean). When `false`, `reason_for_denial` explains why, e.g. "Property lot size is outside of buybox", "Year built is outside of buybox", "Listing status is outside of buybox", "Estimated sale price outside of buybox", "Property type is outside of buybox", or "Property address outside of buybox". A `422` means the request failed validation (check that `state` is present).

## Rules
- No credential is sent to `/buybox/`.
- Treat the result as indicative only — Homeward always relies on its trusted data sources for the definitive Offer Estimate outcome via `createOfferRequest`.
