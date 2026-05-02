# Presto Foundation (presto-foundation)

The Presto Foundation is a Linux Foundation project supporting Presto, an open source distributed SQL query engine for big data analytics. Founded by Facebook (Meta), Uber, Twitter, and Alibaba, Presto enables interactive analytics across diverse data sources at massive scale, with a vendor-neutral governance model and an active ecosystem of contributors and integrations.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/presto-foundation/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Consumer
- **Access:** 3rd-Party

## Tags

- Analytics, Big Data, Distributed SQL, Linux Foundation, Open Source, Query Engine, SQL

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-28

## APIs

### Presto Client REST API

The Presto Client REST API is the HTTP protocol used by Presto clients to submit SQL queries to a Presto coordinator and stream back results. It centers on POST /v1/statement to submit a query, GET on the returned nextUri to fetch subsequent result batches, and DELETE on nextUri to cancel a running query. Session context such as user, source, catalog, schema, time zone, and language is conveyed through X-Presto-* headers, and supports authentication mechanisms including Kerberos, LDAP, password files, OAuth 2.0, and custom authenticators.

**Human URL:** [https://prestodb.io/docs/current/develop/client-protocol.html](https://prestodb.io/docs/current/develop/client-protocol.html)

#### Tags

- Analytics, Big Data, Client Protocol, REST API, SQL

#### Properties

- [Documentation](https://prestodb.io/docs/current/develop/client-protocol.html)
- [REST API Reference](https://prestodb.io/docs/current/rest.html)
- [GitHub Repository](https://github.com/prestodb/presto)

### Presto Coordinator REST API

The Presto Coordinator REST API exposes resources for inspecting and managing a running Presto cluster, including Node, Query, Stage, Statement, and Task resources. These endpoints are served by the coordinator process and are used by clients, monitoring tools, and the Presto worker protocol to coordinate distributed query execution and observe cluster health.

**Human URL:** [https://prestodb.io/docs/current/rest.html](https://prestodb.io/docs/current/rest.html)

#### Tags

- Analytics, Big Data, Cluster Management, REST API, SQL

#### Properties

- [Documentation](https://prestodb.io/docs/current/rest.html)
- [Node Resource](https://prestodb.io/docs/current/rest/node.html)
- [Query Resource](https://prestodb.io/docs/current/rest/query.html)
- [Stage Resource](https://prestodb.io/docs/current/rest/stage.html)
- [Statement Resource](https://prestodb.io/docs/current/rest/statement.html)
- [Task Resource](https://prestodb.io/docs/current/rest/task.html)

## Common Properties

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
- [Linux Foundation](https://www.linuxfoundation.org/projects/case-studies/presto/)

## Maintainers

**FN:** Kin Lane

**Email:** info@apievangelist.com
