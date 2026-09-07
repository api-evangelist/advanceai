# Advanceai

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

# ADVANCE.AI

ADVANCE.AI is the digital identity verification, KYC/KYB, AML, compliance and risk-management
business unit of **Advance Intelligence Group**, headquartered in Singapore and founded in 2016.
Its AdvanGuard product line covers identity verification, document verification, face
authentication and Know Your Business checks for banking and financial services, fintech, crypto,
payments, e-commerce and the sharing economy, with a strong footprint across Southeast Asia and
other emerging markets.

- Website — https://advance.ai/
- API documentation — https://doc.advance.ai/
- API host — https://api.advance.ai
- Status — https://status.advance.ai/
- Request access — https://advance.ai/book-free-demo/

## What ADVANCE.AI publishes

A public **Open API** with nine documented operations across four services: token authentication,
Global Document Verification (SDK licensing, OCR field extraction, ID forgery detection), face
comparison, and liveness detection (signature, licensing, result, video evidence, and programmatic
PII deletion). Plus first-party Android and iOS capture SDKs distributed from ADVANCE.AI's own
Nexus Maven repository and object storage.

Two properties of this API matter more than anything else in it:

1. **Every response is HTTP 200.** Success, authentication failure, quota exhaustion and server
   error all return 200, and the real outcome lives in the JSON envelope field `code`. A client
   that branches on HTTP status reads every failure as a success.
2. **Billing is per response code, and four failure codes are chargeable.** Each documented status
   code carries a `free` or `pay` tag. On face comparison, "no face detected" and "face quality too
   low" both bill — so a blind retry loop on a bad image costs money every attempt.

## Not published by ADVANCE.AI

Recorded here so an absence reads as measured rather than missing. Each was probed on 2026-09-07.

- **No OpenAPI or Swagger.** `/openapi.json`, `/openapi.yaml`, `/swagger.json`, `/v1/openapi.json`,
  `/api-docs`, `/docs` and `/redoc` all return 404 on `api.advance.ai` and on `doc.advance.ai`.
  The specification in `openapi/` was **authored by API Evangelist from the published
  documentation** and is marked as such in its own provenance block. It is not a provider artifact.
- **No `/.well-known/` document of any kind** on any host — see `well-known/` for the full probe
  table, including a soft-200 catch-all on `app.advance.ai` that answers every path with an error
  envelope and would otherwise register as six false positives.
- **No A2A agent card**, **no MCP server**, **no GraphQL, gRPC, WSDL or AsyncAPI surface**, and
  **no webhooks or events**.
- **No server-side client library** on npm, PyPI, Maven Central, NuGet, RubyGems, Packagist,
  crates.io or pkg.go.dev, and no ADVANCE.AI GitHub organization.
- **No published pricing and no self-service sign-up.** The site has no pricing page; access is
  sales-led through a demo request.
- **No idempotency mechanism, no reversal operation, no sandbox and no dated API changelog.**

## Notable findings

- **`clearLivenessPiiData` is irreversible.** ADVANCE.AI offers a programmatic PII erasure endpoint
  — an uncommon and genuinely useful privacy affordance — but documents no restore, no soft delete
  and no recovery window. Recorded in `conventions/` and gated behind human confirmation in the
  skills.
- **The published reference is a subset of the running surface.** The status page lists a
  production component named `curp-info-check` (CURP is the Mexican national identity number) that
  has no page in the public API reference.
- **Certifications are badge images, not text.** BSI, iBeta Level 1 and iBeta Level 2 certificate
  images are displayed on the Security & Compliance page with no certificate number, scope or
  expiry in machine-readable form. This profile records that the badges are displayed and does not
  assert what they cover. ISO 27001, PCI DSS and HIPAA appear on that page only as generic examples
  of regulatory standards and are **not** credited as certifications ADVANCE.AI holds.
- **A security contact exists but is not discoverable by machine.** `security@advance.ai` is
  published in HTML prose; `/.well-known/security.txt` returns 404 everywhere. Serving RFC 9116
  would make an existing commitment machine-readable at near-zero cost.
- **The iOS SDK distribution is unpinned** — the download URL contains the literal path segment
  `latest` and carries no version, so a consumer cannot tell which build they received.

## Related records in this network

ADVANCE.AI is one business unit of Advance Intelligence Group, which also operates Atome Financial
and Ginee. The parent company is profiled separately at `all/advance-intelligence-group/`, and that
record was itself enriched from this same `doc.advance.ai` surface. A third stub exists at
`all/advance.ai/`. **These records overlap and should be reconciled by a human** — see
`x-parent-company` and `x-duplicate-candidates` in `apis.yml`.
