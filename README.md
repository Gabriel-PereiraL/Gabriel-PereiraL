# Gabriel Pereira

Backend Engineer focused on transactional systems, integrations, and reliability.

**Primary engineering stacks: C# / .NET · Java / Spring Boot**

**Professional backend experience: PHP · MySQL**

I build and study backend systems where correctness under failure matters: payments, order workflows, messaging, idempotency, concurrency, authentication, recovery, and observability. My current engineering focus is C#/.NET and Java/Spring Boot. PHP is part of my professional backend experience.

I document what independent reviews found and how I treated each finding in my [audit remediation history](AUDIT_REMEDIATION_HISTORY.md). I keep open limitations explicit instead of presenting a passing pipeline as proof of every production property.

## Featured engineering projects

### [Fulfillment Hub — .NET](https://github.com/Gabriel-PereiraL/fulfillment-hub)

A transactional order, payment, and delivery platform. It explores atomic state changes, transactional outbox, SQS delivery, idempotent consumers, signed webhooks, provider reconciliation, concurrency control, and failure recovery in the .NET ecosystem.

### [Fulfillment Hub — Java / Spring Boot](https://github.com/Gabriel-PereiraL/fulfillment-hub-java)

An equivalent implementation of the fulfillment problem using idiomatic Java and Spring Boot choices. The workflow survives lost provider responses, duplicate and out-of-order events, abandoned processing leases, and HTTP idempotency crash windows. PostgreSQL integration tests and an end-to-end environment exercise the real persistence and messaging boundaries.

The two Fulfillment Hub repositories solve the same core problem in different ecosystems. They demonstrate transferable backend fundamentals rather than a line-by-line port.

### [LedgerLab — PHP / Laravel](https://github.com/Gabriel-PereiraL/ledger-lab)

A financial ledger API centered on consistency: concurrent spending protection, idempotency, reversals, signed webhooks, queues, reconciliation, and automated tests.

### [Orders & Shipping API — PHP](https://github.com/Gabriel-PereiraL/orders-shipping-api)

A framework-light API that makes architecture boundaries and external shipping failures explicit, with static analysis, unit and integration tests, containers, and CI.

## Professional background

My professional backend work uses PHP and MySQL in e-commerce, automotive catalog, and logistics systems. It includes external API integrations, authentication and authorization, catalog synchronization, pricing and availability rules, checkout and order workflows, signed webhooks, operational troubleshooting, testing, and deployment.

The C#/.NET and Java/Spring Boot systems above are engineering portfolio projects. They do not represent professional production experience in those stacks.

## Engineering evidence

- **Correctness:** atomic order and stock changes, optimistic concurrency, explicit state-transition policies, and durable idempotency records.
- **Integration:** REST APIs, HMAC-signed webhooks, external provider simulators, asynchronous messages, retries, and reconciliation.
- **Failure recovery:** stable provider keys, resumable unknown outcomes, owner-fenced leases, dead-letter queues, and bounded retry responsibility.
- **Operations:** structured logs, correlation IDs, OpenTelemetry, Prometheus metrics, health checks, Docker, Kubernetes manifests, and CI security gates.
- **Data:** PostgreSQL and MySQL, relational modeling, migrations, locking, and transaction boundaries.
- **Review and remediation:** adversarial concurrency, crash-window and state-machine findings tracked from reproduction through implementation and CI evidence.

## Technologies

**Primary focus:** C# · .NET · ASP.NET Core · Java · Spring Boot

**Professional experience:** PHP · MySQL · REST APIs · Webhooks

**Data and operations:** PostgreSQL · EF Core · JPA/Hibernate · SQS · Docker · Kubernetes · GitHub Actions · OpenTelemetry · Testcontainers
