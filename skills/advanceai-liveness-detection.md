---
name: advanceai-liveness-detection
description: Run the full ADVANCE.AI liveness detection flow — signature, license, SDK capture, result, video evidence and PII deletion.
api: ADVANCE.AI Open API
operations:
- generateAccessToken
- generateLivenessSignatureId
- authorizeLivenessLicense
- getLivenessDetectionResult
- getLivenessVideo
- clearLivenessPiiData
source: https://doc.advance.ai/liveness_detection.html
generated: '2026-09-07'
method: generated
---

# Run an ADVANCE.AI liveness detection

Six steps, and **step 3 is not an HTTP call**. A human has to look at a phone camera. You cannot
complete this flow autonomously; you orchestrate around a mobile capture your app performs.

## Steps

1. **Token** — see the `advanceai-authenticate` skill.

2. **`generateLivenessSignatureId`**

   ```
   POST https://api.advance.ai/liveness/ext/v1/generate-signature-id
   X-ACCESS-TOKEN: <token>
   Content-Type: application/json

   {"productLevel": "STANDARD", "livenessType": "DISTANT_NEAR"}
   ```

   Both fields are optional and apply only to Advanguard Liveness Detection Standard/Pro on SDK
   above 4.0.0. `productLevel` is `STANDARD` (basic compliance, high pass rate) or `PRO` (strong
   compliance, enhanced accuracy). `livenessType` defaults to `DISTANT_NEAR`.

   Take `data.signatureId`. It is **single-use** — get a fresh one for every detection.

3. **`authorizeLivenessLicense`**

   ```
   POST https://api.advance.ai/openapi/liveness/v1/auth-license
   X-ACCESS-TOKEN: <token>
   Content-Type: application/json

   {"licenseEffectiveSeconds": 600, "applicationId": "appId1,appId2"}
   ```

   Default lifetime 600 seconds, maximum 86400. `ACCESS_DENIED` or `SERVICE_DISABLED` here means the
   account is not entitled to this product — a commercial problem, not a retryable one.

   **A license cannot be revoked.** It expires and that is the only way it ends. Keep
   `licenseEffectiveSeconds` as short as your capture UX allows.

4. **Mobile SDK capture (not an API call).** The Android or iOS Liveness SDK presents the license
   and runs the capture. SDK v4.x.x is the only maintained line — v1, v2 and v3 are marked "No
   Further Updates". v4 supports SILENT, ACTION, DISTANT_NEAR and DISTANT_NEAR_ACTION methods plus
   device security and deepfake detection. The SDK returns a `livenessId`.

5. **`getLivenessDetectionResult`**

   ```
   POST https://api.advance.ai/openapi/liveness/v3/detection-result
   X-ACCESS-TOKEN: <token>
   Content-Type: application/json

   {"livenessId": "...", "resultType": "IMAGE_URL"}
   ```

   Pass `livenessId` or `signatureId` — they cannot both be empty. `resultType` is `IMAGE_URL`
   (default) or `IMAGE_BASE64`.

   Read `data.livenessScore`, a double from 0 to 100. ADVANCE.AI's published guidance:

   | livenessScore | suggestion |
   |---|---|
   | above 50 | pass — normal behaviour |
   | below 50 | manual check — abnormal behaviour |

   When the score is 0, `data.attackType` classifies it: `1` presentation attack, `2` injection
   attack, `3` unsure. `attackSubType` is currently always null.

   Codes to handle: `LIVENESS_ID_NOT_EXISTED`, `RESULT_NOT_FOUND` (the resource was not found or
   has been deleted), `PARAMETER_ERROR`.

6. **`getLivenessVideo`** — optional evidence.

   ```
   GET https://api.advance.ai/liveness/ext/v1/get-video?livenessId=...
   X-ACCESS-TOKEN: <token>
   ```

   Two preconditions: SDK above 4.0.0, and **video recording is not enabled by default** — ADVANCE.AI
   must switch it on for your account first. Without that you get `VIDEO_NOT_FOUND`.
   `pricingStrategy` is documented as deprecated on this operation and always returns FREE.

7. **`clearLivenessPiiData`** — only when you mean it.

   ```
   GET https://api.advance.ai/liveness/ext/v1/clear-data?livenessId=...
   X-ACCESS-TOKEN: <token>
   ```

   **This is irreversible.** ADVANCE.AI documents no restore, no undo, no soft delete and no
   recovery window. Once cleared, the images, the video and the result are gone. Never call this on
   an automated path without an explicit human decision, and never infer a grace period — none is
   published.

## Download everything you need within 24 hours

Every URL this flow returns — `detectionResult`, `imageFarUrl`, `imageNearUrl`, `auditImageUrl`,
`videoUrl` — expires after 24 hours. Re-query for a fresh link, or copy the asset into your own
storage while it is live. `auditImageUrl` is a zip and is null unless configured in the SDK.

## No idempotency

There is no `Idempotency-Key` and no deduplication. A retried `authorizeLivenessLicense` issues a
second license; a retried `getLivenessDetectionResult` bills again. Key your own retries on your own
identifiers and treat every write as at-most-once.
