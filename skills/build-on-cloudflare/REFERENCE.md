# Cloudflare application reference

Use this file by branch. Confirm commands and configuration against the installed Wrangler version and current Cloudflare documentation before you apply them.

## Application shape

| Need | Starting shape | Selection signal |
| --- | --- | --- |
| Static files with no request-time logic | Workers Static Assets | The build output fully defines every public response. |
| Small site, webhook, or API | Worker with a small router or direct handlers | Request-time logic is small and UI state is limited. |
| Interactive product with pages that must be indexed | Framework with supported SSR on Workers | The product needs UI composition, routing, and server-rendered HTML. |
| Stateful coordination | Worker plus Durable Objects | Requests for one entity need ordered access or live coordination. |
| Durable multi-step operation | Worker plus Workflows | Work must resume after delays or failures across several steps. |

Prefer a Worker entry point for dynamic traffic and an explicit static asset directory for public files. Route only the required paths through Worker code.

Official references: [Workers](https://developers.cloudflare.com/workers/), [Static Assets](https://developers.cloudflare.com/workers/static-assets/), [Framework guides](https://developers.cloudflare.com/workers/framework-guides/), [Workflows](https://developers.cloudflare.com/workflows/).

## Data and work ownership

Select a service from the required guarantee, not from convenience.

| Requirement | Service | Design consequence |
| --- | --- | --- |
| Relational records and transactions | D1 | Use migrations. Model indexes and query limits. Use Sessions when read replication needs sequential consistency. |
| Read-heavy key/value data that can tolerate propagation delay | Workers KV | Treat values as cache, configuration, or derived snapshots. Keep revocation and payment truth elsewhere. |
| Strongly coordinated state for one named entity | Durable Objects | Choose an object ID that matches the coordination boundary. Keep cross-object operations explicit. |
| Files and large objects | R2 | Store metadata that needs relational queries in D1. Define retention and access rules. |
| Asynchronous work | Queues | Make consumers idempotent. Record poison-message and retry behavior. |
| Durable sequence with waits, retries, or approvals | Workflows | Make each step repeatable and keep external side effects idempotent. |
| Event and usage aggregates | Analytics Engine | Design indexes and blobs for the queries that the product needs. Account for sampling in queries. |

A binding is an infrastructure dependency. Convert it to a narrow application service at the composition root. This keeps request handlers and domain logic testable.

Official references: [D1](https://developers.cloudflare.com/d1/), [D1 read replication](https://developers.cloudflare.com/d1/best-practices/read-replication/), [KV consistency](https://developers.cloudflare.com/kv/concepts/how-kv-works/), [Durable Objects](https://developers.cloudflare.com/durable-objects/), [R2](https://developers.cloudflare.com/r2/), [Queues](https://developers.cloudflare.com/queues/), [Analytics Engine](https://developers.cloudflare.com/analytics/analytics-engine/).

## Configuration and environments

Use `wrangler.jsonc` or `wrangler.toml` as the deployment configuration source of truth. Include the generated Wrangler schema when the format supports it.

Maintain at least one non-production environment for a public product. Give each environment separate stateful resources, routes, and secret values. Wrangler environment keys and bindings can be non-inheritable, so inspect the effective configuration for every environment.

Configuration review:

- Worker name, entry point, compatibility date, and compatibility flags are intentional.
- Static asset directories contain only publishable files.
- Bindings name the correct resource for the selected environment.
- Preview and staging routes cannot write production data.
- Local secret files are ignored by Git.
- Identifier values can be committed when they are not credentials.
- Secret values use encrypted secrets, not plaintext `vars`.
- Generated binding types are current.

Useful commands:

```sh
npx wrangler types
npx wrangler dev --env staging
npx wrangler deploy --env staging
npx wrangler secret put SECRET_NAME --env staging
```

Run the equivalent package scripts when the project provides them.

Official references: [Wrangler configuration](https://developers.cloudflare.com/workers/wrangler/configuration/), [Wrangler environments](https://developers.cloudflare.com/workers/wrangler/environments/), [Secrets](https://developers.cloudflare.com/workers/configuration/secrets/).

## Caching and freshness

Classify each response before you cache it:

| Response class | Default policy |
| --- | --- |
| Personalized, authenticated, or authorization-dependent | `private, no-store` |
| Public HTML that changes | Short browser lifetime and a separately selected edge lifetime |
| Versioned static asset | Long public lifetime with immutable file names |
| Public API response | Cache only when the key includes every response variant and invalidation is defined |

Treat cache invalidation as part of the write path. Purge or replace cached data only after the authoritative write succeeds. If stale data is acceptable during an upstream failure, expose its age or stale state.

Check behavior with response headers and repeated requests. A local cache test does not prove edge behavior.

Official references: [Workers Cache API](https://developers.cloudflare.com/workers/runtime-apis/cache/), [Cache-Control](https://developers.cloudflare.com/cache/concepts/cache-control/), [Purge cache](https://developers.cloudflare.com/cache/how-to/purge-cache/).

## Identity and abuse controls

Use an established identity protocol or provider. Keep session and entitlement state in a store that meets the required revocation behavior.

For browser sessions:

- generate unpredictable session identifiers;
- set `HttpOnly`, `Secure`, `SameSite`, `Path`, and expiry attributes intentionally;
- validate authorization on each protected action;
- check the request origin for state-changing browser requests;
- revoke the server-side session on logout or account disablement.

For API credentials:

- show a raw credential only when the product requires it;
- store a one-way digest or a provider-managed credential;
- scope credentials and record revocation;
- apply limits by the identity that consumes capacity;
- return stable `401` and `429` responses with recovery information.

Protect login, credential issuance, expensive rendering, scraping, and write endpoints before public launch.

Official references: [Workers security model](https://developers.cloudflare.com/workers/reference/security-model/), [Rate Limiting binding](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/), [Cloudflare Turnstile](https://developers.cloudflare.com/turnstile/).

## Scheduled, queued, and external work

Use `ctx.waitUntil()` only for work that can finish after the response without a separate delivery guarantee. Use Queues or Workflows when work must survive retries or long delays.

For every scheduled or asynchronous handler:

- define a stable idempotency key;
- make partial progress recoverable;
- cap retries and define dead-letter handling where supported;
- log the event ID, attempt, duration, and outcome;
- preserve the last valid output when a source is temporarily unavailable;
- prevent an older deployment or delayed job from overwriting newer data.

For browser automation, select the Browser Rendering REST API or Workers binding from current limits and runtime needs. Cache expensive render results and set explicit navigation and execution timeouts.

Official references: [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/), [Queues delivery guarantees](https://developers.cloudflare.com/queues/reference/delivery-guarantees/), [Browser Rendering](https://developers.cloudflare.com/browser-rendering/).

## Public pages and machine consumers

Pages that need discovery must return useful HTML without client execution. Give each canonical page a unique title, description, canonical URL, social metadata, and content-specific structured data. Return a real `404` for an unknown entity.

Publish a sitemap and `robots.txt` for the production host. Send `X-Robots-Tag: noindex` and a restrictive robots policy from staging.

For a public API or agent surface:

- use stable field names and documented error shapes;
- include provenance and terms with exported data when attribution must survive copying;
- define CORS from the intended caller model;
- publish `llms.txt` when it gives machine users a useful map;
- expose MCP only when tools provide a better task interface than the existing API.

Test the raw HTTP response with JavaScript disabled.

## Observability

Separate operational signals from product signals.

- Use structured Workers logs for request and job diagnosis.
- Include a request or event ID, route, duration, outcome, and safe error code.
- Redact credentials, session values, and personal data.
- Use Analytics Engine or another analytics system for product events and aggregates.
- Add an external check for the health route and one critical path.
- Alert on user-visible failure or exhausted capacity, not on every logged error.

Remember that a request served before Worker execution might not produce a Worker log. Validate analytics placement against caching behavior.

Official references: [Workers Logs](https://developers.cloudflare.com/workers/observability/logs/workers-logs/), [Real-time logs](https://developers.cloudflare.com/workers/observability/logs/real-time-logs/), [Analytics Engine SQL](https://developers.cloudflare.com/analytics/analytics-engine/sql-api/).

## Payments and entitlements

Keep payment collection on a hosted or audited payment surface. Verify each callback signature from the raw request body. Process callback IDs idempotently.

Store entitlement truth in a system with suitable read-after-write and revocation behavior. A successful payment event grants access; refund, cancellation, dispute, and failed renewal events update or revoke it according to the product policy. Test the complete lifecycle in staging with test credentials.

Keep billing credentials in environment-specific secrets. Log provider event IDs and internal outcomes, but not payment details.

## Delivery gates

### Local gate

- Configuration validates with the installed Wrangler version.
- Generated types, static checks, and automated tests pass.
- External parsers use safe, redacted fixtures and test malformed input.
- Migration apply and rollback or forward-fix behavior is understood.
- The diff contains no credential values or unintended public assets.

### Staging gate

- The deployed commit is the candidate intended for production.
- Stateful bindings and secrets resolve to staging resources.
- Health, critical path, authentication, cache, and failure checks pass.
- Logs contain the expected test events and no new unsafe data.
- Staging cannot be indexed as production content.

### Production gate

- Required approval is recorded before the production write.
- The verified commit is promoted without an unreviewed rebuild or code change.
- Migrations run in the planned order.
- Live behavior and affected metrics are checked after propagation.
- Rollback or forward-fix ownership is clear.

Official references: [Cloudflare Workers CI/CD](https://developers.cloudflare.com/workers/ci-cd/), [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/), [Wrangler deploy](https://developers.cloudflare.com/workers/wrangler/commands/#deploy).
