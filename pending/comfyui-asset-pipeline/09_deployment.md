# 09 — Deployment & Infrastructure Specification

## Objective

Define the complete deployment architecture for the Wishes AI Asset Pipeline.

This document covers local development, production deployment, ComfyUI integration, storage layout, service orchestration, monitoring, security, scaling, disaster recovery, and operational procedures.

The deployment architecture must support deterministic asset generation while remaining modular and horizontally scalable.

---

# Deployment Goals

The deployment must:

- Support local development.
- Support production deployment.
- Keep ComfyUI isolated from public access.
- Allow multiple asset workers.
- Scale independently of the game server.
- Preserve deterministic asset generation.
- Support future GPU expansion.

---

# High-Level Architecture

```text
                Unity Client
                     │
                     ▼
             API Gateway (Fastify)
                     │
     ┌───────────────┼────────────────┐
     ▼               ▼                ▼
 Character API   Asset Service   World Services
                     │
                     ▼
              PostgreSQL
                     │
                     ▼
           Asset Generation Queue
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
 Asset Worker 1             Asset Worker N
        │                         │
        └────────────┬────────────┘
                     ▼
                 ComfyUI
                     │
                     ▼
          Generated Asset Storage
```

---

# Core Services

## API Gateway

Responsibilities:

- Authentication
- Routing
- Rate limiting
- Logging
- Public API

Never expose ComfyUI directly.

---

## Asset Service

Responsibilities:

- Queue generation requests
- Review workflow
- Prompt building
- Asset metadata
- Storage management
- Publication

---

## Asset Workers

Workers execute queued jobs.

Responsibilities:

- Claim queued jobs
- Inject workflow values
- Submit to ComfyUI
- Normalize outputs
- Update database

Workers should be stateless.

---

## ComfyUI

Responsibilities:

- Execute workflows
- Produce generated images

ComfyUI should never communicate directly with clients.

---

# Recommended Local Directory Structure

```text
wishes-game/
├── server/
│   └── asset-service/
├── generated-assets/
│   ├── pending/
│   ├── approved/
│   ├── published/
│   ├── rejected/
│   └── archive/
├── models/
│   ├── checkpoints/
│   ├── loras/
│   ├── controlnets/
│   └── vae/
└── workflows/
```

---

# Environment Variables

```text
ASSET_SERVICE_PORT=3300
DATABASE_URL=
COMFYUI_BASE_URL=http://127.0.0.1:8188
ASSET_STORAGE_ROOT=
ASSET_WORKER_POLL_MS=2000
ASSET_MAX_CONCURRENT_JOBS=1
LOG_LEVEL=info
```

Configuration should be environment-driven. Never hardcode local paths.

---

# ComfyUI Deployment

Recommended:

- Dedicated machine or container
- Local-only HTTP endpoint
- GPU acceleration
- Version-controlled workflows
- Read-only workflow directory during runtime

Startup sequence:

1. Load models.
2. Validate workflows.
3. Verify output directories.
4. Accept requests.

---

# Storage Strategy

Directory layout:

```text
generated-assets/
  pending/
  approved/
  published/
  rejected/
  archive/
```

Every asset version receives its own directory.

Never overwrite published assets.

---

# Docker Recommendation

Recommended containers:

```text
postgres
redis
asset-service
api-gateway
comfyui
```

Future:

```text
nginx
grafana
prometheus
```

---

# Scaling Strategy

Scale independently:

- API Gateway
- Asset Service
- Asset Workers

Normally keep one ComfyUI instance per GPU.

If multiple GPUs exist:

```text
Worker -> Dispatcher -> GPU Pool
```

---

# Queue Strategy

Database-backed queue for milestone one.

Future options:

- Kafka
- SQS
- RabbitMQ

Workers must use:

```sql
FOR UPDATE SKIP LOCKED
```

to prevent duplicate claims.

---

# Monitoring

Track:

- Queue length
- Average generation time
- Failed jobs
- GPU utilization
- Storage growth
- Workflow failures

Health endpoints:

```text
/health
/ready
/live
```

---

# Logging

Every state transition should be logged.

Examples:

```text
JobQueued
JobClaimed
WorkflowSubmitted
WorkflowCompleted
AssetApproved
AssetPublished
```

Structured JSON logging is recommended.

---

# Security

Never expose:

- ComfyUI
- Workflow directories
- Model directories
- Raw generation metadata

Protect review/publish APIs with authentication in production.

---

# Backup Strategy

Back up:

- PostgreSQL
- Approved assets
- Published assets
- Workflows
- Templates

Pending assets may be recreated.

---

# Disaster Recovery

If ComfyUI fails:

- Mark jobs failed.
- Preserve prompt metadata.
- Allow retry.

If storage fails:

- Abort publication.
- Preserve review status.

---

# Deployment Phases

Phase 1

- Local development
- Single GPU
- Single worker

Phase 2

- Multiple workers
- Central storage

Phase 3

- Multiple GPUs
- Distributed generation

Phase 4

- Production HA deployment
- Monitoring
- Alerting

---

# Operational Checklist

Before deployment:

- Database migrated
- Workflows validated
- Models present
- Storage writable
- Health checks passing

After deployment:

- Generate portrait
- Approve portrait
- Generate full body
- Publish asset
- Verify Unity can consume output

---

# Acceptance Criteria

1. Services start in correct order.
2. Health endpoints succeed.
3. Workers process queued jobs.
4. ComfyUI remains internal.
5. Assets stored in normalized layout.
6. Published assets remain immutable.
7. Environment configuration controls deployment.
8. Multiple workers supported.
9. Monitoring available.
10. Deployment reproducible across environments.
