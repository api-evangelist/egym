# Egym

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
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

EGYM is a Munich-headquartered fitness and health technology company behind the connected-gym
platform used by commercial fitness operators worldwide — Smart Strength and Smart Flex
networked equipment, the EGYM Genius training engine, the EGYM Fitness Hub and Trainer apps,
and the EGYM Wellpass corporate fitness network.

EGYM runs a public developer portal at [developer.egym.com](https://developer.egym.com/)
covering eight machine-readable APIs across three integration audiences:

| API | Operations | Audience |
|---|---|---|
| [MMS API V2](https://developer.egym.com/mms-api-v2/apis/mms-v2) | 43 | Member management system vendors |
| [MMS API v1](https://developer.egym.com/mms-api-v2/gymapi/api) | 13 | Legacy MMS (v2 required for new work) |
| [Data Hub API](https://developer.egym.com/data-hub/api) | 8 | Analytics export (Enterprise Pack pilot) |
| Data Export API | 4 | Per-account export (alpha; MCP-only discovery) |
| [Equipment Vendor API (standalone)](https://developer.egym.com/equipment/partner-api) | 31 | Device-to-server equipment integration |
| [Equipment Vendor API (server-to-server)](https://developer.egym.com/equipment/equipment-vendor-api-server_august-2025) | 6 | Backend-to-backend measurement relay |
| [User Connect API](https://developer.egym.com/user-connect-api/docs/introduction) | 4 | Consented third-party app submissions |
| [Canonical GroupX Classes](https://developer.egym.com/mms-blueprints/canonical-classes) | 8 | Blueprint the MMS vendor implements |

Plus a prose-only [Pay with Wellpass](https://developer.egym.com/mms-api-v2/tutorials/pay-with-wellpass)
authorization surface, a documented webhook event catalog, and an official
[documentation MCP server](https://developer.egym.com/general/ai-agent-integration) at
`https://developer.egym.com/mcp` with a published AI-agent instruction page.

- https://egym.com/int
- https://developer.egym.com/
- https://www.hiive.com/securities/egym-stock
