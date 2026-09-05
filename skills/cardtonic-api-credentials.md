---
name: cardtonic-api-credentials
description: Complete Cardtonic business KYC and mint the API key that authenticates a merchant integration.
api: cardtonic:business-api
generated: '2026-09-05'
method: generated
source: openapi/cardtonic-openapi.yml (harvested from https://docs.cardtonic.com)
operations:
  - uploadFile
  - verifyKyc
  - addBvn
  - generateApiKey
  - fetchApiKey
---

# Cardtonic API credentials

Takes a verified business account through KYC and out the other side with an API key.
Run `cardtonic-business-onboarding` first — every operation here is account-authenticated.

## Steps

1. **Upload the documents** — `PUT /users/upload` (`uploadFile`), body
   `{ "files": [ { "name": "example.png", "type": "image/png" } ] }`.
   The `200` response returns one descriptor per file with `name`, `type`, `url` and `imageUrl`.
   **Keep the `url` values** — step 2 references them and there is no operation to list uploads
   later, and none to delete one.

2. **Submit corporate KYC** — `POST /users/kyc/verify-kyc` (`verifyKyc`) with:
   - `documents`: an array of the `url` strings from step 1.
   - `shareHoldersInfo`: an array of `{ firstName, lastName, email, phoneNumber, id }`, where `id`
     is itself an uploaded identity-document URL, not a system identifier.
   This is **irreversible** — no amend, withdraw or resubmit operation is published. Validate the
   shareholder list before calling.

3. **Submit the BVN** — `POST /users/kyc/verify-bvn` (`addBvn`), body `{ "bvn": "22222222222" }`.
   The BVN is a Central Bank of Nigeria identifier. Also irreversible.

4. **Mint the key** — `POST /users/generate-key` (`generateApiKey`), empty body.
   Returns `data.apiKey`, a `PRIV_`-prefixed 64-hex-character secret. **Store it immediately.**

5. **Read it back if needed** — `GET /users/show-api-key` (`fetchApiKey`).

## The credential warning worth reading twice

Cardtonic publishes **no revoke and no rotate operation**. `fetchApiKey` reads the key; nothing
invalidates it. If a `PRIV_` key leaks, your only published remedy is to contact
support@cardtonic.com. Treat it as a long-lived secret from the moment it is minted, and never let
an agent surface it in a log, a transcript or an error message.

Cardtonic also does not document which header the key is sent in. Get that from your Cardtonic
contact.

## Retry safety

No idempotency mechanism exists. Calling `generateApiKey` twice is not documented as returning the
same key — assume it does not, and assume you cannot clean up the extra one.
