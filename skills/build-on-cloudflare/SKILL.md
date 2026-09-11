---
name: build-on-cloudflare
description: Build and harden applications on Cloudflare Workers. Use when starting a Cloudflare application or when adding storage, caching, authentication, background work, observability, staging, CI, or a production release to an existing Worker.
license: MIT
metadata:
  author: David Iglesias Guerra
---

# Build on Cloudflare

Build a small, deployable path first. Add each Cloudflare product only when an application invariant requires it.

Read the relevant sections of [REFERENCE.md](REFERENCE.md) before you select services or write configuration.

## 1. Establish the delivery target

Inspect the repository, its instructions, package scripts, Wrangler configuration, and current deployment workflow. Determine:

- the product surface: site, application, API, scheduled process, or a combination;
- the rendering model: static, server-rendered, interactive, or API-only;
- the environments and their release order;
- the data, consistency, retention, and latency requirements;
- the authentication, abuse, privacy, indexing, and billing boundaries.

Ask focused questions for decisions that the repository and request do not answer. This step is complete when each item has an explicit answer or is marked out of scope.

## 2. Select the minimum architecture

Choose the application shape and one service for each data invariant. Use the decision tables in `REFERENCE.md`.

Keep bindings at the application composition root. Pass domain-focused services into inner modules instead of passing the complete Worker environment.

For each selected service, state:

1. the requirement that selects it;
2. the data or work it owns;
3. its failure and consistency behavior;
4. its separate staging and production resources.

This step is complete when every service has a stated reason and no two services own the same source of truth.

## 3. Build a vertical slice

For a new project, use an official Cloudflare starter that fits the selected rendering model. For an existing project, preserve its framework and conventions.

Build one path from request to response:

- commit Wrangler configuration and resource bindings;
- isolate staging resources from production resources;
- create schema changes as migrations;
- declare secret names in configuration when the installed Wrangler version supports it;
- store secret values with Wrangler secrets and local ignored environment files;
- return typed, stable responses at the public boundary.

Use the installed Wrangler version and its generated schema as the authority for configuration fields. This step is complete when the slice runs locally with representative data and has an automated check.

## 4. Add operational controls

Apply only the controls required by the active product branches:

- public caching and explicit private response rules;
- session validation, authorization, origin checks, and abuse limits;
- idempotency for queues, scheduled jobs, billing callbacks, and retries;
- structured logs, health checks, and product metrics;
- complete server-rendered metadata for pages that must be indexed;
- stable API documentation and attribution fields for machine consumers.

Test the failure path for each control. This step is complete when a failed dependency, invalid identity, repeated event, and stale response each have an intentional outcome where applicable.

## 5. Verify the release candidate

Run the repository checks. Include type generation, type checks, tests, migration checks, and a Wrangler build or dry run when the project supports them.

After required approval, deploy the candidate to staging. Verify the deployed system, not only the command result:

- exercise the health route and one critical user path;
- confirm staging uses staging resources and secrets;
- inspect logs for the test requests;
- confirm cache, authentication, robots, and error behavior as applicable;
- run pending migrations before code that depends on them receives traffic.

This step is complete when evidence from staging covers every changed boundary.

## 6. Release and report

Promote the same verified commit through the repository's release workflow. Obtain explicit approval before a production write when the operating environment requires it.

After release, verify the production behavior and affected metrics. Report local completion, staging verification, and production verification as separate states. List deferred controls and the condition that will make each one necessary.
