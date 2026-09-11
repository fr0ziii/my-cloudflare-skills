# Cloudflare migration reference

Use the sections for the active migration phases. Confirm product behavior, limits, commands, and configuration against current first-party documentation.

When you fetch a Cloudflare documentation URL, request its Markdown representation:

```http
Accept: text/markdown
```

For example: `curl -H 'Accept: text/markdown' <URL>`.

## Phase 1: Inventory and baseline

Build the inventory from code and deployed evidence. Do not treat an architecture document as current proof.

| Area | Capture |
| --- | --- |
| User paths | Entry point, route, protocol, response contract, latency target, owner |
| Compute | Runtime, region, duration, concurrency, memory, CPU, package, native dependency |
| State | Store, schema, owner, size, growth, consistency, backup, retention |
| Background work | Trigger, delivery guarantee, retry, idempotency, timeout, dead letter |
| Identity | Provider, session format, credential store, authorization, revocation |
| Network | Domain, DNS, certificate, origin, egress allowlist, private connectivity |
| Operations | Deploy path, approval, logs, metrics, alert, incident and rollback owner |
| Commercial | Provider callback, entitlement source, metering, reconciliation |

Measure at least request volume, p50/p95/p99 latency, error classes, dependency latency, resource use, and cost for the critical paths. Record the measurement window and query so later comparisons use the same definition.

Inventory hidden coupling such as shared databases, hard-coded origins, trusted proxy headers, IP allowlists, scheduled maintenance, and external callbacks.

## Phase 2: Compatibility assessment

Use this matrix for each dependency:

| Current assumption | Workers question | Typical action |
| --- | --- | --- |
| Node.js API | Is the exact API fully supported for the selected compatibility date? | Test it in the Workers runtime. Replace partial or stub behavior. |
| Native addon | Can the package run without a native binary? | Select a Web API, WebAssembly, pure JavaScript, or remote service alternative. |
| Local filesystem | Does the code expect durable local files? | Bundle read-only assets or move durable files to R2. |
| Long-lived process | Does correctness depend on one process staying alive? | Move durable state and work to Durable Objects, Queues, or Workflows. |
| Mutable global state | Can another isolate handle the next request? | Treat globals as disposable optimization only. |
| Database connection | Does the driver and network path work within runtime limits? | Use D1, Hyperdrive, HTTP access, or a supported driver after a deployed test. |
| Background callback | Must work complete after the response or survive failure? | Select `waitUntil`, Queues, or Workflows from the required guarantee. |
| Server listener | Can a Fetch handler represent the route and protocol? | Adapt the application to request handlers or a supported framework adapter. |
| WebSocket state | Do connections need shared coordination? | Evaluate Durable Objects as the connection and state boundary. |
| Provider middleware | Which headers, rewrites, redirects, or edge rules does it imply? | Reproduce the observable contract with Worker routes and explicit code. |

A successful bundle is not compatibility proof. Exercise imported APIs and external connections in a deployed probe.

Official references: [Node.js compatibility](https://developers.cloudflare.com/workers/runtime-apis/nodejs/), [Workers runtime APIs](https://developers.cloudflare.com/workers/runtime-apis/), [Workers limits](https://developers.cloudflare.com/workers/platform/limits/), [Framework guides](https://developers.cloudflare.com/workers/framework-guides/), [Local development](https://developers.cloudflare.com/workers/local-development/).

## Phase 3: Target mapping

These mappings are candidates, not automatic equivalents. Confirm semantics before selection.

| Source responsibility | Cloudflare candidate | Main comparison |
| --- | --- | --- |
| Serverless or edge function | Workers | Runtime APIs, duration, CPU, region, request and response behavior |
| Static hosting | Workers Static Assets | Routing, redirects, headers, build output, fallback behavior |
| Relational database | D1 or existing database through Hyperdrive | SQL support, transactions, consistency, location, driver behavior |
| Key/value or configuration | Workers KV | Propagation delay and stale-read tolerance |
| Coordinated entity state | Durable Objects | Object key, ordering, storage, location, throughput |
| Object storage or S3 | R2 | API compatibility, metadata, lifecycle, access, consistency, transfer |
| Message queue | Queues | Delivery, retry, ordering assumptions, batching, dead letters |
| Scheduled event | Cron Triggers | Schedule semantics, overlap, idempotency, duration |
| Step function or durable job | Workflows | Step replay, waits, retries, limits, external side effects |
| API gateway or reverse proxy | Worker routing | Authentication, rate limits, headers, body limits, observability |
| WebSocket coordinator | Durable Objects | Connection ownership, hibernation, reconnect, state recovery |

For the selected design, complete the 12 phases in `build-on-cloudflare` when that skill is installed. Keep this migration skill focused on compatibility, transition, and cutover.

Official references: [Workers](https://developers.cloudflare.com/workers/), [Static Assets](https://developers.cloudflare.com/workers/static-assets/), [D1](https://developers.cloudflare.com/d1/), [Hyperdrive](https://developers.cloudflare.com/hyperdrive/), [Workers KV](https://developers.cloudflare.com/kv/), [Durable Objects](https://developers.cloudflare.com/durable-objects/), [R2](https://developers.cloudflare.com/r2/), [Queues](https://developers.cloudflare.com/queues/), [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/), [Workflows](https://developers.cloudflare.com/workflows/).

## Phase 4: Migration patterns

| Pattern | Use when | Main risk |
| --- | --- | --- |
| Full replacement | The system is small, stateless, and easy to restore | One cutover contains all uncertainty |
| Incremental routing | Routes or tenants can move independently | Old and new paths can drift |
| Service extraction | One store, job, or capability has a clear boundary | Cross-platform latency and split ownership |
| Read path first | Reads can compare safely before writes move | New reads can hide source inconsistency |
| Parallel environment | A complete target can run without production writes | Test traffic can differ from real traffic |

A slice record must contain:

```text
Name:
User path:
Current owner:
Target owner:
State authority before/during/after:
Data movement:
Traffic control:
Verification query:
Abort thresholds:
Rollback steps:
Removal condition:
```

### Authority progression

Use explicit authority states:

1. The source accepts writes and is authoritative.
2. A repeatable copy builds target state.
3. Safe reads compare source and target.
4. Incremental synchronization closes the lag, if required.
5. A controlled transition changes write authority.
6. The target is authoritative and the source is read-only.
7. The source is retired after the rollback window.

Dual writes add two possible truths. Use them only when the system defines conflict resolution, retry behavior, reconciliation, and repair ownership.

## Phase 5: Data and identity migration

### Data plan

For each dataset, record:

- source and target schema;
- field and type transformation;
- source-of-truth transitions;
- initial-copy tool and expected duration;
- incremental-change mechanism, if needed;
- encryption and access boundaries;
- validation queries and acceptable difference;
- rejected-record handling;
- rerun and resume behavior;
- retention and deletion dates.

Validate more than row counts. Compare checksums, null rates, relationships, aggregates, recent writes, and samples selected independently of migration order.

For R2 migrations from supported object stores, evaluate Super Slurper or Sippy against the cutover model. Verify object metadata, permissions, lifecycle rules, and missing or changed objects after transfer.

For D1, generate a compatible schema and rehearse import or batch operations within current limits. Use migrations for target schema changes. Keep backfill state separate from application requests.

Official references: [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/), [D1 import and export](https://developers.cloudflare.com/d1/best-practices/import-export-data/), [R2 data migration](https://developers.cloudflare.com/r2/data-migration/), [Super Slurper](https://developers.cloudflare.com/r2/data-migration/super-slurper/), [Sippy](https://developers.cloudflare.com/r2/data-migration/sippy/).

### Identity plan

Choose one session transition:

| Strategy | Benefit | Cost |
| --- | --- | --- |
| Preserve session format | Users stay signed in | Target must validate the old format and revocation source |
| Exchange old session | Limits legacy compatibility | Requires a secure one-time transition path |
| Require reauthentication | Simplest trust boundary | Creates user disruption and support load |

Test login, logout, expiry, revocation, account disablement, role changes, CSRF controls, and API credential limits on the target.

## Phase 6: Target verification

Build a comparison set from sanitized production shapes and recorded edge cases. For each request, compare:

- status and redirect chain;
- required headers, cookies, and cache policy;
- body schema or rendered meaning;
- database and external side effects;
- authentication and authorization outcome;
- latency and dependency calls;
- logs, metrics, and trace correlation.

Shadow only safe requests. Block writes, callbacks, emails, payments, and destructive jobs unless the shadow environment replaces each side effect with a controlled test sink.

Failure tests must cover dependency timeout, malformed input, partial data, duplicate event, exhausted limit, stale cache, and target rollback where applicable.

Exit with a signed comparison report that lists accepted differences and unresolved blockers.

Official references: [Workers testing](https://developers.cloudflare.com/workers/testing/), [Workers observability](https://developers.cloudflare.com/workers/observability/), [Preview URLs](https://developers.cloudflare.com/workers/configuration/previews/).

## Phase 7: Traffic cutover

### Readiness check

- The approved commit is deployed to the intended environment.
- Production bindings and secrets resolve to production resources.
- Target data is reconciled inside the agreed tolerance.
- Domain validation and certificates are ready.
- External callbacks use the planned target URL.
- Dashboards show source and target separately.
- Alerts and rollback access have been tested.
- One person owns the cutover decision and one owns data integrity.

### Traffic steps

When gradual delivery is available:

1. Route an internal or allowlisted probe.
2. Route a small percentage or low-risk segment.
3. Hold for the defined sample size and duration.
4. Compare technical and business thresholds.
5. Increase traffic only after the gate passes.
6. Change write authority at its named step.
7. Continue until the target owns the planned traffic.

DNS changes can have resolver and client cache delay. Record previous values, lower TTL early when appropriate, and keep the old origin able to serve during the transition window.

Workers gradual deployments split code versions, but bindings and external state do not roll back with code. Keep schema changes backward compatible across all active versions.

Official references: [Custom Domains](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/), [Versions and deployments](https://developers.cloudflare.com/workers/versions-and-deployments/), [Gradual deployments](https://developers.cloudflare.com/workers/versions-and-deployments/gradual-deployments/), [Rollbacks](https://developers.cloudflare.com/workers/versions-and-deployments/rollbacks/).

## Phase 8: Stabilization and retirement

Define the soak period from traffic cycles and business risk, not from convenience. Include peak traffic, scheduled work, billing callbacks, cache expiry, session expiry, and data reconciliation windows where applicable.

Retirement check:

- Source traffic is zero or limited to an intentional fallback.
- Source writes are disabled.
- Final data reconciliation passes.
- Temporary synchronization and migration jobs are stopped.
- Temporary flags and routes are removed.
- Legacy credentials and firewall access are revoked.
- Old alerts and dashboards are archived or removed.
- Retained backups have access, expiry, and deletion owners.
- Old infrastructure removal has explicit approval.
- Cost and performance are compared with the baseline.

### Final report

Report these states separately:

1. compatibility assessment;
2. target architecture;
3. data migration and authority;
4. staging verification;
5. traffic cutover;
6. production verification;
7. rollback readiness;
8. legacy retirement.

A deployed Worker does not prove migration completion. Completion requires verified traffic, state authority, and removal or explicit ownership of the old path.
