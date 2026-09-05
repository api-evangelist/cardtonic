---
name: cardtonic-business-onboarding
description: Create and activate a Cardtonic business account - sign up, verify the email, and log in for a session token.
api: cardtonic:business-api
generated: '2026-09-05'
method: generated
source: openapi/cardtonic-openapi.yml (harvested from https://docs.cardtonic.com)
operations:
  - signUpBusinessUser
  - resendEmailVerification
  - verifyEmail
  - loginBusiness
---

# Cardtonic business onboarding

Creates a Cardtonic business account and gets you to a session token. Everything downstream —
KYC, API keys, 2FA — requires a verified, logged-in account.

## Before you start

- Every request needs the `X-Tonic-Env` header. Cardtonic's documentation only ever shows the
  value `development`; the production value is not published. Ask your Cardtonic contact.
- Base URL: `https://api.cardtonic.com/v1`
- Access is waitlist-gated at https://cardtonic.com/developer. If your account was not
  provisioned, signup will not help you.

## Steps

1. **Create the account** — `POST /auth/signup/business` (`signUpBusinessUser`).
   Required: `name`, `password`, `phoneNumber`, `username`, `email`, `country`, and a `business`
   object with `name`, `website` and `description` (all three required).
   Returns `201`. There is **no undo** — no account-close operation is published — so treat this
   as a one-way action and confirm the values before firing.

2. **Verify the email** — `PATCH /auth/verify-email` (`verifyEmail`), body `{ "token": "<token>" }`.
   The token arrives by email; no API response ever returns it.
   - `400` means the token is invalid or expired → go to step 3.
   - `422` means the token field was missing or malformed; `errors[].field` names it.

3. **Re-send verification if needed** — `POST /auth/resend-verification-email`
   (`resendEmailVerification`). Safe to call again; nothing is created.

4. **Log in** — `POST /auth/login/business` (`loginBusiness`) with `email` and `password`.
   The `200` body carries a `token`.
   - `403` with "you're yet to verify your email" means step 2 never completed. Do not retry the
     login; go back to step 3.

## What the docs do not tell you

Cardtonic does not publish the header the session token is presented in. You cannot construct an
authenticated call to the `/users/*` operations from the documentation alone — get the header name
from your Cardtonic contact before building step 2 of any downstream flow.

## Error handling

All errors are `application/json` with `{success, message, meta, errors}` — not RFC 9457, and with
no machine-readable error code. Discriminate on HTTP status, then on `errors[].field` for `422`.
See `errors/cardtonic-problem-types.yml`.

## Retry safety

No idempotency key exists on any Cardtonic operation. A retried `signUpBusinessUser` has undefined
behaviour. Retry only `resendEmailVerification`, `forgotPassword` and `loginBusiness`.
