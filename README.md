# Presto Foundation (presto-foundation)

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

The Presto Foundation is a Linux Foundation project supporting Presto, an open source distributed SQL query engine for big data analytics. Founded by Facebook (Meta), Uber, Twitter, and Alibaba, Presto enables interactive analytics across diverse data sources at massive scale, with a vendor-neutral governance model and an active ecosystem of contributors and integrations.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/presto-foundation/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/presto-foundation/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Consumer
- **Access:** 3rd-Party

## Tags

- Analytics
- Big Data
- Distributed SQL
- Linux Foundation
- Open Source
- Query Engine
- SQL

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-28

## APIs

### Presto Client REST API

The Presto Client REST API is the HTTP protocol used by Presto clients to submit SQL queries to a Presto coordinator and stream back results. It centers on POST /v1/statement to submit a query, GET on the returned nextUri to fetch subsequent result batches, and DELETE on nextUri to cancel a running query. Session context such as user, source, catalog, schema, time zone, and language is conveyed through X-Presto-* headers, and supports authentication mechanisms including Kerberos, LDAP, password files, OAuth 2.0, and custom authenticators.

- **Human URL:** [https://prestodb.io/docs/current/develop/client-protocol.html](https://prestodb.io/docs/current/develop/client-protocol.html)

#### Tags

- Analytics
- Big Data
- Client Protocol
- REST API
- SQL

#### Properties

- [Documentation](https://prestodb.io/docs/current/develop/client-protocol.html)
- [R E S T  A P I  Reference](https://prestodb.io/docs/current/rest.html)
- [GitHub Repository](https://github.com/prestodb/presto)
- [Postman Collection](collections/presto-foundation.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/presto-foundation.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Presto Coordinator REST API

The Presto Coordinator REST API exposes resources for inspecting and managing a running Presto cluster, including Node, Query, Stage, Statement, and Task resources. These endpoints are served by the coordinator process and are used by clients, monitoring tools, and the Presto worker protocol to coordinate distributed query execution and observe cluster health.

- **Human URL:** [https://prestodb.io/docs/current/rest.html](https://prestodb.io/docs/current/rest.html)

#### Tags

- Analytics
- Big Data
- Cluster Management
- REST API
- SQL

#### Properties

- [Documentation](https://prestodb.io/docs/current/rest.html)
- [Node  Resource](https://prestodb.io/docs/current/rest/node.html)
- [Query  Resource](https://prestodb.io/docs/current/rest/query.html)
- [Stage  Resource](https://prestodb.io/docs/current/rest/stage.html)
- [Statement  Resource](https://prestodb.io/docs/current/rest/statement.html)
- [Task  Resource](https://prestodb.io/docs/current/rest/task.html)
- [Postman Collection](collections/presto-foundation.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/presto-foundation.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/presto-foundation)
- [Portal](https://prestodb.io/)
- [Documentation](https://prestodb.io/docs/current/)
- [Foundation](https://prestodb.io/foundation/)
- [Blog](https://prestodb.io/blog/)
- [Community](https://prestodb.io/community/)
- [Events](https://prestodb.io/events/)
- [Resources](https://prestodb.io/resources/)
- [GitHub Organization](https://github.com/prestodb)
- [Slack](https://communityinviter.com/apps/prestodb/prestodb)
- [Twitter](https://twitter.com/prestodb)
- [YouTube](https://www.youtube.com/c/PrestoDB)
- [Linux  Foundation](https://www.linuxfoundation.org/projects/case-studies/presto/)

## Maintainers

**FN:** Kin Lane
**Email:** info@apievangelist.com
