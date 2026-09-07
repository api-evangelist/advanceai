---
name: advanceai-face-comparison
description: Compare two face photographs with ADVANCE.AI and act on the similarity score without burning budget on avoidable failures.
api: ADVANCE.AI Open API
operations:
- generateAccessToken
- compareFaces
source: https://doc.advance.ai/face_recognition.html
generated: '2026-09-07'
method: generated
---

# Compare two faces with ADVANCE.AI

This is the only ADVANCE.AI flow that runs entirely over HTTP — no mobile SDK step. It is also the
one where careless retries cost real money.

## Cost warning, read before implementing

Four **failure** codes on this operation are tagged `pay` and bill you:

- `NO_FACE_DETECTED_FROM_FIRST_IMAGE`
- `NO_FACE_DETECTED_FROM_SECOND_IMAGE`
- `FIRST_IMAGE_LOW_QUALITY_FACE`
- `SECOND_IMAGE_LOW_QUALITY_FACE`

Retrying the same bad image bills again. Validate before you call, and never put these codes on a
retry path.

## Steps

1. Get a token — see the `advanceai-authenticate` skill.

2. **Validate both images locally first.** ADVANCE.AI's published requirements:
   - format PNG, JPG or JPEG
   - under 2 MB
   - at least 256x256 and at most 4096x4096

   Anything outside this returns `IMAGE_INVALID_FORMAT` or `IMAGE_INVALID_SIZE` — both are free, but
   they are also a wasted round trip.

3. Call `compareFaces`:

   ```
   POST https://api.advance.ai/openapi/face-recognition/v4/check
   X-ACCESS-TOKEN: <token>
   Content-Type: multipart/form-data

   firstImage=@first.jpg
   secondImage=@second.jpg
   ```

4. Read `code` — the HTTP status is always 200.

   On `SUCCESS`, `data.similarity` is a float from 0 to 100. `data.firstFace` and
   `data.secondFace` carry the bounding box (`left`, `top`, `right`, `bottom`) and detected
   `gender` of the matched face in each image.

## Acting on the score

ADVANCE.AI publishes these thresholds. Use them rather than inventing your own:

| similarity | suggestion |
|---|---|
| above 70 | pass |
| 55 to 70 | manual check |
| below 55 | reject, or verify identity another way |

Do not auto-approve inside the 55-70 band. ADVANCE.AI's own guidance is that it needs a human.

## Behaviour worth knowing

- If several faces are present, the **largest** face is compared. If your images may contain more
  than one person, that choice is not yours.
- Photos flipped 90, 180 or 270 degrees are accepted, but ADVANCE.AI states accuracy drops. Rotate
  to upright before sending if you can.
- No result is stored. There is no identifier to fetch this comparison again — the only handle you
  get is `transactionId`. Persist it with your own record, along with the score, before you move on.

## Errors

Handle the four billable codes above by rejecting the input, not by retrying. Also handle the shared
codes from the Glossary: `SERVICE_BUSY` and `RETRY_LATER` (honour the `Retry-After` header),
`OVER_QUERY_LIMIT` and `INSUFFICIENT_BALANCE` (not retryable — a commercial problem), `IAM_FAILED`
(refresh the token once). Full catalogue: `errors/advanceai-error-codes.yml`.
