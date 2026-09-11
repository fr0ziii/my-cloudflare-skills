---
name: migrate-to-cloudflare
description: Migrate existing applications and services to Cloudflare. Use when moving from Vercel, Netlify, AWS Lambda, a conventional Node.js server, or another platform; replacing a data or job service with a Cloudflare product; planning a partial migration; or preparing a safe traffic cutover and rollback.
license: MIT
metadata:
  author: David Iglesias Guerra
---

# Migrate to Cloudflare

Preserve observable behavior before you optimize for the target platform. Move ownership in reversible slices and keep the current source authoritative until a verified cutover changes it.

Use [REFERENCE.md](REFERENCE.md) for assessment tables, migration patterns, checks, and official documentation.

## Choose the route

- **Full migration:** Run all eight phases in order.
- **Partial migration:** Run Phases 1 and 2, select the phases that own the change, then run Phases 6 through 8.
- **Assessment only:** Run Phases 1 through 4 and deliver the plan without changing production.

For each phase, record decisions, evidence, risks, open questions, and rollback conditions. Mark exclusions with a reason. A phase is complete only when its exit condition is true.

## Phase 1: Inventory the current system

Read repository instructions, application code, infrastructure definitions, deployment workflows, and operational documentation. Inventory:

- routes, protocols, rendering, and critical user paths;
- runtimes, packages, build steps, and environment assumptions;
- databases, files, caches, queues, schedules, and state owners;
- identity, sessions, credentials, payments, and third-party integrations;
- domains, DNS, certificates, traffic controls, and deployment permissions;
- current traffic, latency, errors, capacity, cost, and recovery targets.

Mark facts separately from assumptions. Capture a baseline for behavior that the migration must preserve.

**Exit condition:** Every critical path has known dependencies, state ownership, deployment ownership, and baseline evidence.

## Phase 2: Prove runtime compatibility

Compare each runtime feature and dependency with the current Workers runtime, limits, and supported framework behavior. Inspect Node.js APIs, native modules, filesystem use, process lifecycle, sockets, database drivers, long requests, background work, and mutable global state.

Classify every item as `compatible`, `change required`, `blocked`, or `unknown`. Resolve critical unknowns with a small deployed compatibility probe instead of an architectural assumption.

**Exit condition:** No critical path depends on an untested unknown, and every incompatibility has a selected change or a documented blocker.

## Phase 3: Design the target architecture

Map each responsibility to the smallest suitable Cloudflare service. Define request routing, static assets, server rendering, bindings, data stores, coordination, asynchronous work, observability, and environment isolation.

Keep bindings at the composition root and pass narrow services into application modules. Preserve one source of truth for each kind of data. When `build-on-cloudflare` is available, use it to validate the target design.

**Exit condition:** Every target service has a stated invariant, ownership boundary, failure behavior, and separate staging resource.

## Phase 4: Divide the migration into reversible slices

Choose full replacement, incremental routing, or service-only migration. Prefer a vertical slice that moves one complete user path while the remaining traffic stays on the current system.

For every slice, define:

1. entry and exit traffic;
2. state authority before, during, and after the slice;
3. data-copy or compatibility needs;
4. release and observation steps;
5. abort thresholds;
6. rollback procedure.

Use flags or routing controls only when they have a clear removal step. Introduce dual writes only with conflict ownership, repair, and reconciliation rules.

**Exit condition:** Each slice can deploy, verify, and roll back without depending on a later slice.

## Phase 5: Migrate data and identity

Define schemas, transformations, backfills, validation, incremental synchronization, write cutover, and retention. Rehearse the process with representative staging data.

Compare counts, checksums, relationships, samples, and application-level invariants. Record rejected rows and make reruns idempotent. Protect credentials and personal data throughout the transfer.

Select an explicit session strategy: preserve compatible sessions, exchange them, or require reauthentication. Verify authorization and revocation after the transition.

**Exit condition:** A rehearsed process can move and reconcile the required state within the recovery and cutover limits.

## Phase 6: Verify the target under realistic traffic

Deploy the target to an isolated staging environment. Replay sanitized fixtures and critical requests. Compare status, headers, body semantics, side effects, latency, and logs.

Use shadow traffic only when it cannot repeat unsafe writes or expose private data. Test load, dependency failure, retries, stale-data behavior, and rollback. Run migrations before code that depends on them receives traffic.

**Exit condition:** Deployed evidence covers every changed boundary, and measured behavior meets the agreed migration thresholds.

## Phase 7: Cut over traffic safely

Obtain required approval for shared and production changes. Confirm domains, certificates, routes, DNS behavior, secrets, bindings, dashboards, alerts, and on-call ownership.

Shift traffic in measured steps when the platform and application permit it. At each step, compare errors, latency, saturation, business events, and data reconciliation against abort thresholds. Change write authority only at the planned transition point.

If a threshold fails, stop the shift and run the prepared rollback or forward-fix procedure. Remember that a code rollback does not roll back data or bound resources.

**Exit condition:** The target owns the planned traffic and writes, live metrics are healthy, and rollback remains available for the agreed observation period.

## Phase 8: Stabilize and retire the old path

Observe the target for the agreed soak period. Reconcile final state, investigate differences, and confirm that user, crawler, API, scheduled, and billing paths work where applicable.

Remove temporary routes, synchronization, flags, credentials, and migration jobs after their rollback need ends. Retain or delete old data according to the recovery, legal, and privacy policy. Remove old infrastructure only after traffic and state ownership are proven absent.

Report implementation, publication, staging, production, data authority, rollback readiness, and retirement as separate states.

**Exit condition:** The target is the verified system of record, temporary migration mechanisms are removed, and retained legacy resources have an owner and expiry condition.
