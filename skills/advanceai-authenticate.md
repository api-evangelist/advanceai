---
name: advanceai-authenticate
description: Obtain and cache an ADVANCE.AI access token, and use it correctly on every other call.
api: ADVANCE.AI Open API
operations:
- generateAccessToken
source: https://doc.advance.ai/global_document_verification.html
generated: '2026-09-07'
method: generated
---

# Authenticate against the ADVANCE.AI Open API

Every other ADVANCE.AI operation needs an `X-ACCESS-TOKEN` header. This skill gets one.

## Before you start

You need an `accessKey` and a `secretKey`. They are issued per account and read from the ADVANCE.AI
Websaas platform under **Account > Account Management**. There is no self-service issuance; access
is arranged through https://advance.ai/book-free-demo/. Never send the `secretKey` — it is only ever
used to compute a hash.

## Steps

1. Take the current time as a **13-digit epoch milliseconds** value. Call it `timestamp`.
   ADVANCE.AI accepts roughly 300 seconds of clock skew; outside that you get
   `PARAMETER_ERROR` / "Timestamp error".

2. Compute `signature` as the SHA256 hex digest of the three values concatenated in this exact
   order, with no separator: `accessKey + secretKey + timestamp`.

3. Call `generateAccessToken`:

   ```
   POST https://api.advance.ai/openapi/auth/ticket/v1/generate-token
   Content-Type: application/json

   {"accessKey": "...", "signature": "...", "timestamp": 1648785145789, "periodSecond": 3600}
   ```

   `periodSecond` is optional — default 3600, minimum 60, maximum 86400.

4. **Read `code`, not the HTTP status.** The response is HTTP 200 whether it worked or not.
   - `SUCCESS` — take `data.token` and `data.expiredTime`.
   - `PARAMETER_ERROR` — the message says which: "Parameter should not be empty", "Timestamp
     error", or "Signature error". All three are your bug. Do not retry unchanged.
   - `ACCOUNT_DISABLED` — commercial problem, not a technical one. Contact the sales manager.
   - `CLIENT_ERROR` — you sent a malformed HTTP request. Check the path, verb and content type.

5. Cache the token against `expiredTime`. Send it on every other operation as
   `X-ACCESS-TOKEN: <token>`.

## Rules that matter

- **One token covers everything.** The same token is valid for document verification, face
  comparison and liveness. Do not mint one per service.
- **Reissue does not revoke.** Requesting a new token leaves the old one working until it expires,
  so you can refresh ahead of time with no cutover gap. Refresh at ~80% of lifetime.
- **`IAM_FAILED` on a business call usually means the token expired.** Refresh once and retry. If
  it persists, the message distinguishes an expired token from an entitlement problem — "Account
  not authorized for this country" and "Account not authorized for this domain" are entitlement,
  not authentication, and retrying will never fix them.
- If your infrastructure is in mainland China, ADVANCE.AI advises routing via VPN; the service is
  deployed outside China and you will otherwise see packet loss and timeouts.
