---
name: build-on-cloudflare
description: Build and harden applications on Cloudflare Workers. Use when starting a Cloudflare application or when adding storage, caching, authentication, background work, observability, staging, CI, or a production release to an existing Worker.
license: MIT
metadata:
  author: David Iglesias Guerra
---

# Build on Cloudflare

Build the smallest complete product path. Add a Cloudflare service only when a product invariant selects it.

Use [REFERENCE.md](REFERENCE.md) for decisions, commands, checks, and official documentation.

## Choose the route

- **Full build:** Run all 12 phases in order.
- **Focused change:** Select the phases that own the change, inspect their prerequisites, and run Phase 11. Run Phase 12 only when a production release is in scope and authorized.

For every phase, record the decision, implementation, verification evidence, and deferred work. Mark an optional phase as `not applicable` with a reason. A phase is complete only when its exit condition is true.

## Foundation

### Phase 1: Define the product boundary

Inspect repository instructions, current behavior, deployment state, and package scripts. Define the users, product surfaces, rendering needs, domain plan, privacy boundary, indexing needs, and commercial model.

Resolve missing decisions with focused questions. Keep out-of-scope work explicit.

**Exit condition:** Each product concern has a decision or a documented exclusion.

### Phase 2: Select the runtime and scaffold

Choose static assets, a direct Worker, or a framework with supported server rendering. Prefer an official Cloudflare starter for a new project and preserve established conventions in an existing project.

Keep public files in an explicit asset directory. Keep bindings at the composition root and pass narrow services into inner modules. Build one request-to-response path before broad infrastructure work.

**Exit condition:** A representative path runs locally and the selected runtime has a stated reason.

### Phase 3: Isolate configuration and secrets

Use Wrangler configuration as the deployment source of truth. Define development, staging, and production behavior. Give stateful environments separate resources, routes, and secret values.

Generate binding types. Store secret values through Wrangler secrets or ignored local files. Use the installed Wrangler schema as the authority for configuration fields.

**Exit condition:** Effective configuration and binding ownership are known for every active environment, and no secret value is committed.

## Platform

### Phase 4: Assign data and work ownership

Inventory each record, blob, cache, coordination boundary, event, and long-running operation. Select a service from its consistency, query, retention, and delivery requirements.

Create schema changes as migrations. Define idempotency, recovery, and separate staging resources. Keep one source of truth for each kind of data.

**Exit condition:** Every stateful concern has one owner, one required guarantee, and one tested failure outcome.

### Phase 5: Design caching and freshness

Classify responses as private, public dynamic content, immutable assets, or cacheable API data. Define cache keys, browser and edge lifetimes, invalidation, and stale-data behavior.

Verify cache behavior through deployed response headers and repeated requests. Treat invalidation as part of the authoritative write.

**Exit condition:** No private variant can enter a shared cache, and every cached response has a freshness and invalidation rule.

### Phase 6: Add identity and abuse controls

Define authentication, session or credential storage, authorization, origin validation, revocation, and rate limits. Use an established identity protocol or provider.

Protect login, write, credential issuance, scraping, and expensive compute paths. Test invalid identity, insufficient permission, revoked access, and excess traffic.

**Exit condition:** Each protected action has explicit identity, authorization, revocation, and abuse behavior.

### Phase 7: Make background and external work reliable

Select `waitUntil`, Cron Triggers, Queues, Workflows, or Browser Rendering from the required delivery and execution behavior.

Make retries and repeated events safe. Define timeouts, retry limits, dead-letter or recovery behavior, and stale-result policy. Cache expensive external work when its freshness permits it.

**Exit condition:** Interrupted, delayed, duplicated, and failed work each has an intentional outcome.

## Surface

### Phase 8: Configure domains and discovery surfaces

Attach production and staging hosts to the correct environments. Prevent staging content from indexing.

For public pages, return useful server-rendered HTML, canonical metadata, structured data, real not-found responses, robots policy, and a sitemap. For machine consumers, define stable APIs, provenance, terms, CORS, `llms.txt`, and MCP only where they add value.

**Exit condition:** Human, crawler, and machine-consumer responses are correct without relying on unintended client behavior.

### Phase 9: Add observability and product analytics

Emit structured, redacted operational logs. Add health checks, request or event identifiers, product events, useful queries, external checks, and alerts for user-visible failure.

Account for traffic served before Worker execution when you choose where to measure an event.

**Exit condition:** A test request and a test failure can be found in operational signals, and a critical product event can be found in analytics.

### Phase 10: Connect payments and entitlements

Run this phase only for a paid product. Keep payment collection on a hosted or audited surface. Verify callbacks from the raw request body and process provider event IDs idempotently.

Model entitlement grant, renewal, cancellation, refund, dispute, payment failure, and revocation. Test the full lifecycle with staging credentials.

**Exit condition:** Repeated and out-of-order billing events converge on the intended entitlement state.

## Release

### Phase 11: Prove the release candidate in staging

Run generated types, static checks, tests, migration checks, and a Wrangler build or dry run. Use safe fixtures for external inputs and scan the diff for credentials and unintended public files.

After required approval, deploy the candidate to staging. Verify resource isolation, migrations, health, the critical path, changed failure paths, logs, cache behavior, authentication, and indexing controls as applicable.

**Exit condition:** Evidence from the deployed staging commit covers every changed boundary.

### Phase 12: Release and operate production

Obtain required production approval. Promote the same verified commit through the repository workflow, run migrations in the planned order, and observe the release.

Verify live behavior and affected metrics after propagation. Keep rollback or forward-fix ownership clear. Report local completion, staging verification, production verification, and deferred controls as separate states.

**Exit condition:** Production behavior is verified, operational ownership is clear, and the final report distinguishes every delivery state.
