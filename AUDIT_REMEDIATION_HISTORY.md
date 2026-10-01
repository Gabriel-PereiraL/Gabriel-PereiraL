# Audit remediation history

I use independent and adversarial reviews to test the claims I make about my portfolio. This document records what each finding exposed, what I changed, and what evidence I used. I keep limitations visible when a review identified a risk that I have not yet reproduced or closed.

## First audit — F01–F15

| Finding | What I found | What I changed |
|---|---|---|
| F01 | Cancellation could reach a provider before resource ownership was established. | I moved provider work behind the authorized local transaction and a durable `OrderCancelled` event. An integration test proves another customer cannot change stock, order state, outbox, or provider state. |
| F02 | A lost payment-creation response left no provider ID and no recovery path. | I persist `Submitting`/`Unknown`, retain the original idempotency key, and reconcile attempts without a provider ID. |
| F03 | Delivery recovery discarded the saved quote and did not reliably adopt duplicates. | I reload the persisted quote, renew it only when needed, retain the recovery identity, and adopt a compatible duplicate response. |
| F04 | Runtime persistence paths could bypass the domain transition rules. | I introduced a shared temporal state policy and applied it to payment and delivery provider results, including late and stale responses. |
| F05 | Order commit and HTTP idempotency completion had a crash window. | I linked the reservation to the order inside the business transaction and reconstruct the response when final response persistence was interrupted. |
| F06 | A consumer could perform its effect before durable deduplication. | I acquire an owner-fenced database claim before work and complete or release it only with the current owner token. |
| F07 | Webhook processing had no recoverable owner lease. | I added owner, lease, retry scheduling, expired-claim recovery, and conditional completion. |
| F08 | Reconciliation could repeat snapshots and hide failures. | I lock the row, make repeated snapshots a no-op, and emit structured failure logs. |
| F09 | Permanent delivery rejection remained eligible for repeated processing. | I persist a terminal failure, cancel locally, release stock idempotently, and create durable refund/cancellation obligations. |
| F10 | A required outbox event without a handler could be acknowledged. | I make a missing required handler fail visibly and enter the retry/failure path. |
| F11 | Correlation, traces, metrics, and reconciliation logging were incomplete. | I added structured reconciliation logs, correlation capture, W3C trace propagation through outbox/SQS, provider metrics, OTLP configuration, and worker health signals. |
| F12 | Local PostgreSQL was ephemeral and the worker lacked useful health signalling. | I added persistent storage, heartbeat-based probes, resource/security settings, and termination grace. I do not present this as proof of a production Kubernetes deployment. |
| F13 | Sistema-portaria contained fixed database and login credentials. | I moved database configuration to environment variables and replaced fixed passwords with PBKDF2 hashes. I kept historical rotation as an explicit operational responsibility. |
| F14 | CI used mutable dependencies and did not exercise every image or the complete flow. | I pinned third-party Actions, added Maven Wrapper, restricted permissions, added E2E, Gitleaks, CodeQL, and Trivy gates for API, worker, and simulator images. |
| F15 | Processing leases and queue visibility did not have a clear ownership budget. | I added fenced owners, renewals, bounded batches, single-message consumption, and a declared 300-second visibility budget. |

## Adversarial audit — F16–F31

| Finding | What I found | What I changed |
|---|---|---|
| F16 | A delivery create response could revive an order cancelled during the remote call. | I revalidate the locked order around remote work and preserve cancellation while recording the required remote compensation. |
| F17 | Permanent delivery rejection cancelled an order without returning stock. | I moved idempotent stock restoration and the compensation obligation into the cancellation transaction. |
| F18 | A late `Delivered` snapshot could overwrite a cancelled order. | I apply provider snapshots through the shared state policy and preserve the cancelled terminal decision. |
| F19 | Provider `Returned`/`Cancelled` states did not converge the order and its compensations. | I added the authorized local transition, stock release, and durable refund obligation. |
| F20 | A late permanent payment response could replace an already settled payment. | I made settled states monotonic and separated conflict, uncertainty, and definitive rejection handling. |
| F21 | A crash after the fifth webhook claim could leave an unrecoverable `Processing` row. | I separate an interrupted claim from a completed failed attempt and recover or explicitly park expired work. |
| F22 | An `Ignored` webhook was not terminal when redelivered. | I treat `Ignored` and `Processed` as acknowledged terminal outcomes. |
| F23 | A permanent quote rejection left a paid order repeatedly eligible. | I classify permanent quote failures and converge the order through cancellation, stock restoration, and refund scheduling. |
| F24 | An ordinary customer could read Prometheus metrics. | I restricted the endpoint to the administrative policy and covered the access matrix. |
| F25 | Address construction outside cleanup could leave an idempotency key stuck. | I moved domain construction inside the reservation cleanup boundary. |
| F26 | A concurrent first idempotency reservation could leak a database conflict as HTTP 500. | I normalize the persistence race into the existing idempotency read/replay path. |
| F27 | .NET could commit an order before persisting its idempotent response. | I added a durable customer/key association, a unique partial index, short execution leases, long replay retention, and response reconstruction. |
| F28 | .NET can create a new delivery identity after an expired quote even when the previous remote outcome is unknown. | I documented this as an open recovery limitation. I do not claim that this scenario is closed until a lost-response/expired-quote test proves adoption with one remote delivery. |
| F29 | My profile metadata did not reflect Java as a primary engineering focus. | I rewrote the profile around C#/.NET and Java/Spring Boot and describe PHP as professional backend experience. |
| F30 | Orders & Shipping can overwrite a confirmed aggregate with a stale quote snapshot during a concurrent write. | I documented this as an open concurrency limitation. I do not claim compare-and-swap protection until the repository has a version check and a reproducing integration test. |
| F31 | The Java provider simulator used a non-atomic get/create/put sequence. | I made same-key creation atomic and added 64-way concurrency coverage for payment and delivery creation. |

## Reaudit — F32–F42

| Finding | What I found | What I changed |
|---|---|---|
| F32 | SQS treated a busy claim like completed work and completion did not enforce affected rows. | I model `Acquired`, `Busy`, and `Completed` separately, delete only after true completion, and require owned completion to update one row. |
| F33 | Runtime transitions still diverged from domain rules. | I reject `Pending → Refunded` unless payment is settled and route delivery create/cancel results through the temporal policy. |
| F34 | Cancellation during delivery quotation could still be followed by remote creation. | I re-read the paid state after the quote and stop before remote creation when cancellation won. |
| F35 | Simulator refunds accepted invalid amounts and did not replay by idempotency key. | I require a key, validate `0 < refund <= payment`, and replay the same refund result. |
| F36 | Outbox correlation existed as columns without complete capture and restoration. | I capture request correlation/trace context, send it as SQS attributes, and restore the context while consuming. |
| F37 | The Java snapshot had an incomplete failing CI run. | I fixed the concurrent assertion, updated vulnerable runtime dependencies, rebuilt all images, and obtained green build, integration, E2E, Trivy, Gitleaks, and CodeQL runs. |
| F38 | .NET acknowledged failed webhooks too early. | I keep them eligible for bounded retries up to five attempts and return failure so queue redelivery controls the next attempt. |
| F39 | .NET used one expiration window for execution ownership and response replay. | I separated a two-minute execution lease from 24-hour response retention and recover the committed order by customer/key. |
| F40 | LedgerLab could persist an inbox event but lose the queue dispatch. | I redispatch known pending duplicates and added a scheduled scan for unprocessed inbox rows. |
| F41 | PHP workflows referenced mutable third-party Actions. | I pinned the LedgerLab and Orders & Shipping workflow dependencies to reviewed commit SHAs. |
| F42 | Sistema-portaria checked and inserted open visits in separate database operations. | I lock the visitor and perform check/insert in one transaction, then added a MariaDB generated column and unique index for one open visit per visitor. |

## Verification and published changes

I published the remediations in their owning repositories instead of copying implementation detail into this profile:

- [Fulfillment Hub — Java](https://github.com/Gabriel-PereiraL/fulfillment-hub-java): unit tests, 28 PostgreSQL/LocalStack integration tests, full E2E, three image scans, Gitleaks, and CodeQL passed on the final remediation SHA.
- [Fulfillment Hub — .NET](https://github.com/Gabriel-PereiraL/fulfillment-hub): Release build, 245 local tests, environment E2E in CI, image/security gates, and CodeQL passed.
- [LedgerLab](https://github.com/Gabriel-PereiraL/ledger-lab): PHP style, static analysis, migrations, and test workflow passed after durable inbox recovery changes.
- [Orders & Shipping API](https://github.com/Gabriel-PereiraL/orders-shipping-api): CI passed with pinned workflow dependencies.
- [Sistema-portaria](https://github.com/Gabriel-PereiraL/Sistema-portaria): Python compilation checks passed; existing databases must apply `schema_hardening.sql` during deployment.

I distinguish a green automated gate from broader production proof. These projects demonstrate mechanisms and executable scenarios; they do not claim real payment traffic, cloud scale, high availability, or operational history in C# and Java.
