---
name: advanceai-document-verification
description: Run the ADVANCE.AI Global Document Verification flow and read the OCR extraction and ID forgery verdict.
api: ADVANCE.AI Open API
operations:
- generateAccessToken
- authorizeDocumentVerificationLicense
- queryDocumentVerificationResult
source: https://doc.advance.ai/global_document_verification.html
generated: '2026-09-07'
method: generated
---

# Verify an identity document with ADVANCE.AI

Three steps, and the middle one is a mobile SDK capture, not an HTTP call. There is no way to submit
a document image to this flow over HTTP — the `IDVID` that addresses a result is minted by the SDK.

## Steps

1. **Token** — see the `advanceai-authenticate` skill.

2. **`authorizeDocumentVerificationLicense`**

   ```
   POST https://api.advance.ai/intl/openapi/face-identity/document-verification/v1/auth-license
   X-ACCESS-TOKEN: <token>
   Content-Type: application/json

   {"licenseEffectiveSeconds": 600, "applicationId": "appId1,appId2"}
   ```

   Both fields are optional. `licenseEffectiveSeconds` defaults to 600 and caps at 86400;
   `applicationId` restricts the license to named applications. Take `data.license` and
   `data.expireTimestamp`.

   `ACCESS_DENIED` or `SERVICE_DISABLED` means the account is not entitled to this product. Do not
   retry — raise it commercially.

3. **Mobile SDK capture (not an API call).** The Global IQA SDK — Android via the Maven coordinate
   `ai.advance.mobile-sdk.android:global-iqa`, iOS via the xcframework bundle — presents the license
   and captures the document. It returns an `IDVID` (a UUID). Cordova and React Native wrappers are
   documented for Android.

4. **`queryDocumentVerificationResult`**

   ```
   POST https://api.advance.ai/intl/openapi/face-identity/document-verification/v1/query
   X-ACCESS-TOKEN: <token>
   Content-Type: application/json

   {"IDVID": "ba959a16-a06c-4b82-ae1f-22452b5bbcf3", "resultType": "IMAGE_URL"}
   ```

   Note the field name is uppercase `IDVID` — it is the one field in this API that is not camelCase.
   `resultType` is `IMAGE_URL` (default, 24-hour link) or `IMAGE_BASE64`.

## Reading the result

`data` carries three things:

- **`image`** — the captured document, as a link or base64.
- **`OCR`** — the extracted fields. **The field set depends on the document type.** The documented
  example is an Indonesian KTP and returns `idNumber`, `fullName`, `fullNameLocal`, `expiryDate`,
  `state`, `city`, `district`, `subdistrict`, `fullAddress`, `gender`, `bloodType`, `religion`,
  `nationality`, plus an open `others` map holding document-specific extras such as `rtrw`,
  `occupation`, `birthPlaceBirthday` and `maritalStatus`. Do not hard-code this shape for other
  countries — treat every field as optional and read `others` defensively.
- **`idForgery`** — `result` is `"pass"` (not forged) or `"fail"` (forged). On `fail`, `detail`
  carries the reason. ADVANCE.AI's published fail reasons are: retake or screenshot; colour
  photocopy or cut-off corner; black-and-white photocopy; pasted face; modified NIK.

**Treat `idForgery.result: "fail"` as a hard stop.** It is a fraud signal, not a quality signal, and
it should not be routed to the same queue as a low-confidence OCR read.

## Errors

- `IDVID_NOT_EXISTS` — the identifier is wrong or the capture never completed. Not retryable.
- `PARAMETER_ERROR` — "Parameter should not be empty" or "Parameter error, please check your request
  whether has illegal parameters". Your bug.
- Shared Glossary codes apply: `SERVICE_BUSY` / `RETRY_LATER` (honour `Retry-After`),
  `OVER_QUERY_LIMIT`, `INSUFFICIENT_BALANCE`, `IAM_FAILED`. Full catalogue:
  `errors/advanceai-error-codes.yml`.

## Image quality

The SDK enforces capture quality, but the documented requirements are worth knowing when debugging:
PNG/JPG/JPEG, under 2 MB, 256x256 to 4096x4096, document readable and not tilted around 45 degrees,
unobscured, clean background.

## The image link expires in 24 hours

If you need to retain the document image for audit, copy it to your own storage the moment you
receive it. Re-querying returns a fresh link, but only while the record exists.
