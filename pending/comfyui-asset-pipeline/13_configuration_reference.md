# 13 — Configuration Reference

## Objective

This document defines the complete configuration system used by the Wishes AI Asset Pipeline.

The goals of the configuration system are:

- Eliminate hard-coded values.
- Allow every environment to be configured independently.
- Keep secrets outside source control.
- Support local development, staging, and production.
- Allow future cloud deployment without code changes.

Every configurable value should originate from one of the following sources:

1. Environment Variables
2. Configuration Files
3. Database Configuration
4. Runtime Defaults

Configuration precedence must always be:

```text
Environment Variable
        ↓
Configuration File
        ↓
Database Configuration
        ↓
Application Default
```

---

# Design Principles

Configuration should be:

- Explicit
- Version controlled (except secrets)
- Environment-specific
- Strongly typed
- Validated during startup
- Immutable after application startup unless explicitly designed otherwise

---

# Configuration Categories

The Asset Pipeline configuration is divided into:

```text
Application
Database
Redis
Storage
Asset Pipeline
ComfyUI
Rendering
Workflows
Workers
Logging
Security
Monitoring
Performance
Feature Flags
Unity Integration
```

---

# Environment Files

Recommended layout:

```text
.env
.env.local
.env.development
.env.test
.env.production
```

Never commit:

- passwords
- API keys
- tokens
- connection strings containing credentials

---

# Application Configuration

```text
APP_NAME=Wishes Asset Service
APP_ENV=development
APP_PORT=3300
APP_HOST=0.0.0.0
APP_LOG_LEVEL=info
```

---

# Database Configuration

```text
DATABASE_HOST=
DATABASE_PORT=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_SSL=
DATABASE_POOL_SIZE=
```

Startup validation:

- Connection succeeds
- Required migrations applied
- Required extensions installed

---

# Redis Configuration

```text
REDIS_HOST=
REDIS_PORT=
REDIS_PASSWORD=
REDIS_DATABASE=
```

Future use:

- distributed locking
- cache
- session storage

---

# Asset Storage Configuration

```text
ASSET_STORAGE_ROOT=
ASSET_PENDING_PATH=
ASSET_APPROVED_PATH=
ASSET_PUBLISHED_PATH=
ASSET_REJECTED_PATH=
ASSET_ARCHIVE_PATH=
ASSET_TEMP_PATH=
```

Storage paths should always resolve relative to:

```text
ASSET_STORAGE_ROOT
```

---

# ComfyUI Configuration

```text
COMFYUI_BASE_URL=
COMFYUI_TIMEOUT_MS=
COMFYUI_HISTORY_POLL_MS=
COMFYUI_MAX_PARALLEL_REQUESTS=
COMFYUI_OUTPUT_DIRECTORY=
```

Optional future:

```text
COMFYUI_CLUSTER_ENABLED=
COMFYUI_DISPATCH_MODE=
```

---

# Workflow Configuration

```text
WORKFLOW_DIRECTORY=
WORKFLOW_MANIFEST_DIRECTORY=
WORKFLOW_VALIDATE_ON_STARTUP=true
```

Startup should fail if validation is enabled and required workflows are invalid.

---

# Prompt Configuration

```text
PROMPT_GLOBAL_STYLE=
PROMPT_NEGATIVE_DEFAULT=
PROMPT_MAX_LENGTH=
PROMPT_ENABLE_IDENTITY_BLOCK=true
```

Global styles should be stored separately from workflow definitions.

---

# Rendering Configuration

```text
CARD_RENDER_WIDTH=
CARD_RENDER_HEIGHT=

THUMBNAIL_SIZE=

ICON_SIZE=

SPRITE_TILE_WIDTH=

SPRITE_TILE_HEIGHT=
```

Renderer configuration should remain deterministic.

---

# Worker Configuration

```text
WORKER_ENABLED=true

WORKER_COUNT=1

WORKER_POLL_INTERVAL_MS=2000

WORKER_HEARTBEAT_MS=5000

WORKER_RETRY_LIMIT=3

WORKER_TIMEOUT_MS=900000
```

Future:

```text
GPU_AFFINITY

QUEUE_PRIORITY
```

---

# Queue Configuration

```text
QUEUE_BATCH_SIZE=

QUEUE_MAX_PENDING=

QUEUE_RETRY_DELAY_MS=

QUEUE_CLEANUP_DAYS=
```

---

# Logging Configuration

```text
LOG_LEVEL=

LOG_FORMAT=json

LOG_INCLUDE_STACKTRACE=true

LOG_SQL=false
```

Recommended production format:

JSON structured logging.

---

# Security Configuration

```text
ENABLE_AUTH=true

JWT_SECRET=

JWT_EXPIRATION=

ALLOW_ANONYMOUS_REVIEW=false
```

Future:

```text
OIDC_PROVIDER
```

---

# Feature Flags

Recommended feature flags:

```text
ENABLE_SPRITES

ENABLE_CARD_RENDERER

ENABLE_PORTRAIT_REFINEMENT

ENABLE_BATCH_RENDER

ENABLE_CHARACTER_LORA

ENABLE_IDENTITY_SCORING
```

Every experimental feature should be feature flagged.

---

# Monitoring Configuration

```text
ENABLE_METRICS=true

ENABLE_HEALTH_ENDPOINTS=true

ENABLE_PERFORMANCE_LOGGING=true
```

Future:

```text
PROMETHEUS

GRAFANA

OTEL
```

---

# Performance Configuration

```text
MAX_UPLOAD_SIZE_MB

MAX_IMAGE_DIMENSION

MAX_QUEUE_DEPTH

MAX_RENDER_THREADS
```

---

# Unity Configuration

```text
UNITY_EXPORT_PROFILE=game

UNITY_ASSET_ROOT=

UNITY_METADATA_VERSION=
```

Unity should consume only published assets.

---

# Configuration Service

Implement a dedicated ConfigurationService.

Responsibilities:

- Load configuration
- Validate configuration
- Expose strongly typed values
- Prevent duplicate parsing
- Fail fast on invalid values

No service should read process environment variables directly.

---

# Startup Validation

During startup validate:

- Database
- Redis
- Storage
- Workflow directory
- Workflow manifests
- ComfyUI connectivity
- Required templates
- Required fonts
- Required frames

Application startup should fail immediately if required resources are missing.

---

# Runtime Configuration Objects

Recommended structure:

```text
ApplicationConfig

DatabaseConfig

StorageConfig

WorkflowConfig

RenderingConfig

WorkerConfig

LoggingConfig

SecurityConfig

MonitoringConfig
```

---

# Validation Rules

Every configuration value should validate:

Examples:

```text
Port

1-65535

Worker Count

>=1

Poll Interval

>0

Timeout

>0

Directory Exists

true

URL Valid

true
```

---

# Configuration Versioning

Configuration schema should include:

```text
schema_version
```

Future upgrades should support migration.

---

# Secrets Management

Production secrets should originate from:

- Environment variables
- Secret manager
- Container secrets

Never:

- commit secrets
- log secrets
- return secrets through APIs

---

# Operational Overrides

Operational overrides should be possible for:

- worker count
- logging level
- queue interval
- ComfyUI endpoint
- storage location

without code changes.

---

# Testing Configuration

Provide:

```text
.env.test
```

Configured for:

- isolated database
- isolated storage
- mock ComfyUI
- reduced timeouts

---

# Configuration Documentation

Every configuration value should document:

- name
- description
- default
- valid range
- required
- environment
- restart required

---

# Acceptance Criteria

Configuration implementation is complete when:

1. No runtime constants are hardcoded.
2. Startup validates required configuration.
3. Invalid configuration prevents startup.
4. Configuration is strongly typed.
5. Secrets remain outside source control.
6. Runtime services use ConfigurationService.
7. Feature flags control experimental systems.
8. Configuration supports local, staging, and production.
9. Environment changes require no code modification.
10. Configuration documentation remains synchronized with implementation.