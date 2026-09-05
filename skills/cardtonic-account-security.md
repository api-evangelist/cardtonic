---
name: cardtonic-account-security
description: Manage Cardtonic business account security - two-factor enrollment, password change, and password recovery.
api: cardtonic:business-api
generated: '2026-09-05'
method: generated
source: openapi/cardtonic-openapi.yml (harvested from https://docs.cardtonic.com)
operations:
  - enable2fa
  - disable2fa
  - changePassword
  - forgotPassword
  - resetPassword
---

# Cardtonic account security

The three security controls Cardtonic publishes on a business account.

## Two-factor authentication

- **Enable** — `POST /users/enable-2fa` (`enable2fa`), empty body.
- **Disable** — `POST /users/disable-2fa` (`disable2fa`), empty body.

This is the **only reversible pair** in the whole published Cardtonic surface. Cardtonic states no
window and no precondition on the reversal.

Cardtonic does not document the factor type (TOTP, SMS, email), nor any enrollment challenge or
verification step. Expect an out-of-band step that the contract does not describe, and do not build
an unattended flow that assumes `enable2fa` alone completes enrollment.

## Change a known password

`PATCH /users/change-password` (`changePassword`) on an authenticated account.
`422` returns `errors[]` with `{message, field, location}` per offending field.

There is no undo. `forgotPassword`/`resetPassword` is a recovery flow, not a rollback — it cannot
restore the previous credential.

## Recover a forgotten password

1. `POST /auth/forgot-password` (`forgotPassword`) with the account email. Safe to retry.
2. `POST /auth/reset-password` (`resetPassword`) with the emailed `token` and the new password.
   - `400` = token invalid or expired → repeat step 1.
   - `422` = field validation; read `errors[].field`.

## Conventions that apply to all of these

- `X-Tonic-Env` is required on every request.
- Success bodies are `{message, success, data, meta}` — unwrap `data`.
- Errors are `{success, message, meta, errors}`; no error codes, no RFC 9457.
- No rate-limit headers and no `429` are documented, so back off on any non-2xx you did not expect.
