# Cloudflare application reference

Use the sections for the active phases. Confirm commands and configuration against the installed Wrangler version and current Cloudflare documentation before you apply them.

When you fetch an official reference URL, request its Markdown representation:

```http
Accept: text/markdown
```

For example: `curl -H 'Accept: text/markdown' <URL>`.

Keep these rules across all phases:

- Make public files explicit.
- Keep one source of truth for each kind of data.
- Isolate staging from production.
- Commit configuration and keep secret values encrypted or local.
- Prefer a small, direct design over an unnecessary abstraction.
- Verify deployed behavior instead of treating a successful command as proof.

## Phase 1: Product boundary

Record the following facts before selecting services:

| Concern | Required decision |
| --- | --- |
| Product | Primary user, core task, public and private surfaces |
| Delivery | Site, application, API, scheduled process, or combination |
| Rendering | Static, server-rendered, interactive, or API-only |
| Identity | Anonymous, browser session, API credential, organization, or combination |
| Data | Consistency, query, retention, residency, and recovery needs |
| Discovery | Search indexing, social previews, API documentation, agent access |
| Commercial model | Free, subscription, one-time payment, metered API, or out of scope |
| Release | Environments, approval boundary, target branch, rollback owner |

A custom domain is not a scaffold dependency. A new product can prove its first deployment on a `workers.dev` host, then attach production and staging domains when the routes are ready.

## Phase 2: Runtime and scaffold

| Need | Starting shape | Selection signal |
| --- | --- | --- |
| Static files with no request-time logic | Workers Static Assets | Build output defines every response. |
| Small site, webhook, or API | Direct Worker or small router | Request-time logic is limited and UI state is small. |
| Interactive pages that need indexing | Framework with supported SSR | The product needs UI composition and server-rendered HTML. |
| Stateful coordination | Worker plus Durable Objects | Requests for one entity need ordered access or live coordination. |
| Durable multi-step operation | Worker plus Workflows | Work must resume across waits or failures. |

Start a new project with the current official scaffold:

```sh
npm create cloudflare@latest
```

Keep browser assets in the configured static asset directory. Put command-line jobs and maintenance tools in separate entry points from the Worker runtime.

Treat bindings as infrastructure dependencies. Convert them to domain-focused services at the composition root. Inner modules must receive only the capabilities they use.

Checks:

```sh
npx wrangler dev
npx wrangler deploy --dry-run
```

Use project scripts when they wrap these commands.

Official references: [Workers](https://developers.cloudflare.com/workers/), [Static Assets](https://developers.cloudflare.com/workers/static-assets/), [Framework guides](https://developers.cloudflare.com/workers/framework-guides/), [Workflows](https://developers.cloudflare.com/workflows/).

## Phase 3: Environments, configuration, and secrets

Use `wrangler.jsonc` or `wrangler.toml` as the deployment configuration source of truth. Include the Wrangler schema when the format and installed version support it.

Configuration review:

- Worker name, entry point, compatibility date, and compatibility flags are intentional.
- Asset directories contain only publishable files.
- Dynamic route patterns run Worker code only where needed.
- Bindings resolve to the correct resource in each environment.
- Preview and staging cannot write production state.
- Local secret files are ignored by Git.
- Identifier values are distinguished from credentials.
- Secret values use encrypted secrets instead of plaintext `vars`.
- Generated binding types match the effective configuration.

Wrangler environment fields and bindings can be non-inheritable. Inspect each environment instead of assuming it inherits top-level values.

Useful commands:

```sh
npx wrangler types
npx wrangler dev --env staging
npx wrangler secret put SECRET_NAME --env staging
npx wrangler deploy --env staging
```

Official references: [Wrangler configuration](https://developers.cloudflare.com/workers/wrangler/configuration/), [Wrangler environments](https://developers.cloudflare.com/workers/wrangler/environments/), [Secrets](https://developers.cloudflare.com/workers/configuration/secrets/).

## Phase 4: Data and work ownership

Select services from required guarantees:

| Requirement | Service | Design consequence |
| --- | --- | --- |
| Relational records and transactions | D1 | Use migrations. Design indexes and query limits. Use Sessions when read replication needs sequential consistency. |
| Read-heavy key/value data that tolerates propagation delay | Workers KV | Use for cache, configuration, or derived snapshots. Keep revocation and payment truth elsewhere. |
| Coordinated state for one named entity | Durable Objects | Match the object ID to the coordination boundary. Keep cross-object work explicit. |
| Files and large objects | R2 | Keep queryable relational metadata in D1. Define retention and access. |
| Asynchronous delivery | Queues | Make consumers idempotent. Define retry and dead-letter behavior. |
| Multi-step work with waits or recovery | Workflows | Make steps repeatable and external side effects idempotent. |
| Product-event aggregates | Analytics Engine | Design indexes and blobs for required queries. Account for sampling. |

For every binding, record:

1. the invariant that selects it;
2. the data or work it owns;
3. consistency and failure behavior;
4. retention and recovery behavior;
5. staging and production resource names.

Create a migration before the first D1 schema change. Test it against representative data and define whether recovery uses rollback or a forward fix.

Official references: [D1](https://developers.cloudflare.com/d1/), [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/), [D1 read replication](https://developers.cloudflare.com/d1/best-practices/read-replication/), [KV consistency](https://developers.cloudflare.com/kv/concepts/how-kv-works/), [Durable Objects](https://developers.cloudflare.com/durable-objects/), [R2](https://developers.cloudflare.com/r2/), [Queues](https://developers.cloudflare.com/queues/), [Analytics Engine](https://developers.cloudflare.com/analytics/analytics-engine/).

## Phase 5: Caching and performance

Classify every response before caching it:

| Response class | Default policy |
| --- | --- |
| Personalized or authorization-dependent | `private, no-store` |
| Public HTML that changes | Short browser lifetime and a separately selected edge lifetime |
| Content-addressed or versioned asset | Long public lifetime with immutable names |
| Public API response | Shared cache only when the key covers every variant and invalidation is defined |

A cache design must specify:

- all key inputs, including locale, encoding, and public variants;
- browser and edge lifetimes;
- invalidation after an authoritative write;
- behavior while content is stale;
- behavior when the origin or upstream source fails;
- the metric that proves the cache helps.

Purge or replace data only after its source-of-truth write succeeds. Mark served fallback data as stale when age affects user trust.

Check deployed headers with repeated requests. Local cache behavior is not evidence of edge behavior.

Official references: [Workers Cache API](https://developers.cloudflare.com/workers/runtime-apis/cache/), [Cache-Control](https://developers.cloudflare.com/cache/concepts/cache-control/), [Purge cache](https://developers.cloudflare.com/cache/how-to/purge-cache/).

## Phase 6: Identity, authorization, and abuse

Use an established identity protocol or provider. Store sessions and entitlements in a service that meets the required revocation behavior.

Browser-session checks:

- Session identifiers are unpredictable.
- Cookies set `HttpOnly`, `Secure`, `SameSite`, `Path`, and expiry intentionally.
- Each protected action validates identity and authorization.
- State-changing browser requests validate their origin.
- Logout and account disablement revoke server-side access.

API-credential checks:

- Stored credentials use a one-way digest or provider-managed storage.
- Credentials have scope, ownership, creation, and revocation records.
- Limits apply to the identity that consumes capacity.
- `401` and `429` responses have stable bodies and recovery information.

Apply abuse controls to login, write, credential issuance, scraping, rendering, and other expensive routes. Add Turnstile only where a human challenge fits the interaction.

Official references: [Workers security model](https://developers.cloudflare.com/workers/reference/security-model/), [Rate Limiting binding](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/), [Turnstile](https://developers.cloudflare.com/turnstile/).

## Phase 7: Scheduled, queued, and browser work

Choose the mechanism from the delivery requirement:

| Requirement | Mechanism |
| --- | --- |
| Finish short non-critical work after a response | `ctx.waitUntil()` |
| Run work on a time schedule | Cron Trigger |
| Deliver asynchronous items with retries | Queue |
| Resume a durable sequence across waits and failures | Workflow |
| Execute or capture a browser page | Browser Rendering |

For each handler:

- derive a stable idempotency key;
- make partial progress recoverable;
- cap retries and define terminal failure handling;
- log event ID, attempt, duration, and outcome;
- preserve the last valid result when temporary failure permits it;
- prevent delayed work from replacing newer output.

For Browser Rendering, select the REST API or Worker binding from current runtime and limit requirements. Set navigation and execution timeouts. Cache results when freshness permits it. Test each external target because bot protection and page behavior vary.

Official references: [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/), [Queues delivery guarantees](https://developers.cloudflare.com/queues/reference/delivery-guarantees/), [Workflows](https://developers.cloudflare.com/workflows/), [Browser Rendering](https://developers.cloudflare.com/browser-rendering/).

## Phase 8: Domains, indexing, and machine interfaces

Map each host to one environment. Keep production and staging resources separate. Give staging responses an `X-Robots-Tag: noindex` header and a restrictive `robots.txt`.

Pages that need discovery must return useful HTML without client execution. Each canonical page needs:

- a unique title and description;
- a canonical URL;
- social preview metadata and an image;
- content-specific structured data;
- internal links that make the page reachable;
- a real `404` response for an unknown entity.

Production must publish an intentional `robots.txt` and sitemap. Verify raw HTML and response status with an HTTP client.

For a public API or agent surface:

- keep field names and error shapes stable;
- document authentication, limits, and terms;
- include provenance with exported data when attribution must survive copying;
- set CORS from the intended caller model;
- publish `llms.txt` when it provides a useful machine-readable map;
- add MCP only when tools offer a better task interface than the API.

Official references: [Custom Domains](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/), [Static Assets routing](https://developers.cloudflare.com/workers/static-assets/routing/), [Cloudflare Agents MCP](https://developers.cloudflare.com/agents/model-context-protocol/).

## Phase 9: Observability and analytics

Separate operational signals from product signals.

Operational signals:

- structured logs with request or event ID, route, duration, outcome, and safe error code;
- redaction of credentials, sessions, payment details, and personal data;
- a health route that checks only dependencies needed for its stated health level;
- an external check for health and one critical user path;
- alerts for sustained user-visible failure or exhausted capacity.

Product signals:

- an event allowlist and stable event names;
- dimensions that answer defined product questions;
- queries that account for Analytics Engine sampling;
- client-side measurement when edge caching prevents Worker execution;
- a retention and privacy policy.

Verification requires one successful request, one controlled failure, and one product event visible in the selected systems.

Official references: [Workers Logs](https://developers.cloudflare.com/workers/observability/logs/workers-logs/), [Real-time logs](https://developers.cloudflare.com/workers/observability/logs/real-time-logs/), [Analytics Engine](https://developers.cloudflare.com/analytics/analytics-engine/), [Analytics Engine SQL](https://developers.cloudflare.com/analytics/analytics-engine/sql-api/).

## Phase 10: Payments and entitlements

Use a hosted or audited payment surface so application code does not handle raw card data.

Billing-handler checks:

- Verify the provider signature against the raw request body.
- Deduplicate by provider event ID.
- Keep payment and webhook credentials in environment-specific secrets.
- Log event IDs and safe internal outcomes, not payment details.
- Make entitlement updates safe under retries and out-of-order delivery.

Model the complete lifecycle: initial payment, renewal, payment failure, cancellation, refund, dispute, and revocation. Store entitlement truth in a service that provides the required read and revocation behavior.

Test the lifecycle in staging with provider test credentials. Include an unpaid or failed path, repeated callbacks, and a revoked entitlement.

## Phase 11: Tests, CI, migrations, and staging

### Local gate

- Effective configuration validates with the installed Wrangler version.
- Generated binding types are current.
- Static checks and automated tests pass.
- External parsers use safe, redacted fixtures and reject malformed or implausible results.
- Migration order and rollback or forward-fix behavior are understood.
- A Wrangler build or dry run succeeds.
- The diff contains no credential values or unintended public assets.

### CI gate

- CI runs deterministic checks without production credentials.
- Deployment uses a reviewed commit, not uncommitted output.
- Staging migrations run before code that requires them receives traffic.
- A failed staging smoke check stops promotion.

### Staging gate

- The deployed commit is the intended production candidate.
- Bindings and secrets resolve to staging resources.
- Health and one critical path pass against the deployed host.
- Changed authentication, cache, background, billing, and failure paths pass where applicable.
- Logs contain the expected tests and no unsafe data.
- Staging cannot be indexed as production content.

Official references: [Workers CI/CD](https://developers.cloudflare.com/workers/ci-cd/), [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/), [Wrangler deploy](https://developers.cloudflare.com/workers/wrangler/commands/#deploy).

## Phase 12: Production release and operation

### Release gate

- Required approval covers the production write and its deployment effect.
- The verified staging commit is the commit being promoted.
- Production resources, routes, and secret names are correct.
- Migrations run in the planned order.
- Rollback or forward-fix steps and ownership are clear.

### Live verification

- Allow expected propagation time.
- Check the production health route and critical path.
- Confirm the changed cache, identity, indexing, billing, or background behavior.
- Inspect operational signals and affected product metrics.
- Send a test alert when alert delivery changed.

Report these states separately:

1. local implementation and checks;
2. commit and publication state;
3. staging deployment and evidence;
4. production deployment and evidence;
5. deferred controls with the condition that activates each one.

A successful Git operation is not production evidence. A successful deployment command is not live-behavior evidence.
