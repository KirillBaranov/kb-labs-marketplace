# KB Labs Marketplace

Unified marketplace service for KB Labs. This repo owns plugin and package lifecycle operations such as install, uninstall, enable, disable, link, unlink, sync, doctor, and list.

## Overview

Marketplace is split into a small set of focused packages:

- [`@kb-labs/marketplace-app`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/apps/marketplace/package.json): Fastify entrypoint that runs the HTTP service
- [`@kb-labs/marketplace-api`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-api/package.json): routes, OpenAPI, observability surfaces
- [`@kb-labs/marketplace-core`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-core/package.json): domain logic for package lifecycle operations
- [`@kb-labs/marketplace-contracts`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-contracts/package.json): shared types and contracts
- [`@kb-labs/marketplace-npm`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-npm/package.json): npm/pnpm-backed source integration
- [`@kb-labs/marketplace-cli`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-cli/package.json): CLI-facing integration package

The service runs on port `5070` in local development.

## Development

From repo root:

```bash
pnpm install
pnpm build
pnpm test
pnpm type-check
```

Run the marketplace service directly:

```bash
cd /Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace
pnpm --filter @kb-labs/marketplace-app dev
```

Run it through the workspace dev manager:

```bash
cd /Users/kirillbaranov/Desktop/kb-labs-workspace
./scripts/kb-dev start marketplace
./scripts/kb-dev ready marketplace --timeout 30s
./scripts/kb-dev status --json | jq '.services.marketplace'
```

## HTTP Service

Default local base URL:

```text
http://localhost:5070
```

Primary runtime endpoints:

- `GET /health`
- `GET /ready`
- `GET /metrics`
- `GET /observability/describe`
- `GET /observability/health`

Marketplace API routes are registered from [`packages/marketplace-api/src/routes`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-api/src/routes) under the `/api/v1/marketplace` prefix. Health and observability surfaces stay at the service root.

Representative operations:

- `POST /api/v1/marketplace/install`
- `POST /api/v1/marketplace/uninstall`
- `POST /api/v1/marketplace/enable`
- `POST /api/v1/marketplace/disable`
- `POST /api/v1/marketplace/link`
- `POST /api/v1/marketplace/unlink`
- `POST /api/v1/marketplace/sync`
- `POST /api/v1/marketplace/update`
- `GET /api/v1/marketplace/list`
- `GET /api/v1/marketplace/doctor`

## Observability

Marketplace is already migrated to the platform observability contract.

Canonical surfaces:

- `GET /health` — cheap public liveness
- `GET /ready` — readiness gate
- `GET /metrics` — Prometheus-compatible metrics
- `GET /observability/describe` — versioned service descriptor
- `GET /observability/health` — structured runtime diagnostics

The service emits bounded marketplace domain operations such as:

- `marketplace.bootstrap`
- `marketplace.list`
- `marketplace.install`
- `marketplace.uninstall`
- `marketplace.enable`
- `marketplace.disable`
- `marketplace.link`
- `marketplace.unlink`
- `marketplace.sync`
- `marketplace.update`
- `marketplace.doctor`

Implementation entrypoint:

- [`packages/marketplace-api/src/server.ts`](/Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace/packages/marketplace-api/src/server.ts)

Verification:

```bash
curl http://localhost:5070/health
curl http://localhost:5070/ready
curl http://localhost:5070/observability/describe
curl http://localhost:5070/observability/health
curl http://localhost:5070/metrics
```

## Quality Checks

Useful local commands:

```bash
cd /Users/kirillbaranov/Desktop/kb-labs-workspace/platform/kb-labs-marketplace
pnpm build
pnpm test
pnpm type-check
```

Focused package checks:

```bash
pnpm --filter @kb-labs/marketplace-api build
pnpm --filter @kb-labs/marketplace-api test
pnpm --filter @kb-labs/marketplace-core build
pnpm --filter @kb-labs/marketplace-contracts build
```

## Notes

- Marketplace is part of the platform backend and should stay aligned with the shared observability and service bootstrap patterns.
- For cross-workspace service orchestration, prefer `kb-dev` over ad hoc background processes.
- If marketplace behavior changes, update this README together with route or observability changes.
