---
name: Request a Homeward cash offer for a seller lead
description: Submit a seller's property to Homeward and return the cash Offer Estimate (amount, value range, offer PDF, and finalization link).
api: openapi/homeward-offer-estimate-openapi.yml
operations: [checkBuyboxEligibility, createOfferRequest, getOfferEstimate, finalizeOfferRequest]
---

# Request a Homeward cash offer

Use this skill to turn a seller lead into a Homeward Offer Estimate. Homeward is a partner-only API; you need a token issued by Homeward (email api@homeward.com) supplied in the `Authentication` header on every authenticated call.

## Prerequisites
- Homeward partner token (testing or production).
- Base host `https://api.homeward.com`. All JSON; send `Content-Type: application/json` and `Accept: application/json`.

## Steps

1. **(Optional) Pre-check eligibility** with `checkBuyboxEligibility` — `POST /buybox/`. This endpoint needs no credential. Send `property_address` (only `state` is required; more fields = more accurate) plus optional `opinion_of_value`, `property_type`, `listing_status`, `year_built`. A `200` returns `is_eligible` and, when false, a `reason_for_denial` (e.g. "Estimated sale price outside of buybox"). This is indicative only, not a final determination.

2. **Create the offer request** with `createOfferRequest` — `POST /api/1.0.0/partner/offer-request/` with the `Authentication` header. Required fields: `partner_offer_request_id` (your unique id), `street_one`, `city`, `state`, `postal_code`, `customer_first_name`, `customer_last_name`, `customer_email` (must be unique). Add property detail (`bedrooms`, `home_size_sq_ft`, `property_type`, `year_built`, ...) to sharpen the estimate. A `201` returns `homeward_offer_id`, `offer_amount`, `opinion_of_value_min/max`, `preliminary_offer_pdf`, `offer_finalization_link`, and `preliminary_offer_id`. If the property is denied, `reason_for_denial` is set and the offer fields are `null`.

3. **Fetch the full breakdown** (optional) with `getOfferEstimate` — `GET /api/1.0.0/estimate/sell/{preliminary_offer_id}/?fmt=json` using the `preliminary_offer_id` from step 2. Returns milestones, net offer price range, program fees, and net proceeds so you can explain the offer in your UI.

4. **Finalize** by sending the customer to `finalizeOfferRequest` — `POST /agents/finalize-lead/{lead_id}/` on `https://app.homeward.com`, or simply redirect to the `offer_finalization_link` from step 2. This hands off into the Homeward application, which may collect more information.

## Rules
- Authenticate with the `Authentication` header on every call except `/buybox/`.
- Errors use a `{ "status": <int>, "data": "<message>" }` envelope; `4xx` = your request was invalid, `5xx` = retry later or contact api@homeward.com.
- `partner_offer_request_id` and a unique `customer_email` de-duplicate leads; there is no formal idempotency-key contract, so avoid blind retries of `createOfferRequest`.
