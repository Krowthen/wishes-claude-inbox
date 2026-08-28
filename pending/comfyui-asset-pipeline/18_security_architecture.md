# 18 — Security Architecture

## Objective

This document defines the complete security architecture for the Wishes AI Asset Pipeline.

The objective is to ensure that every asset, workflow, AI model, and service is protected from unauthorized access while maintaining a development workflow that remains efficient for a small team and scalable to enterprise deployment.

Security must be considered at every layer of the system rather than being implemented as an afterthought.

---

# Security Principles

The Wishes Asset Pipeline follows these principles:

- Default Deny
- Least Privilege
- Defense in Depth
- Immutable Published Assets
- Complete Auditability
- Explicit Authorization
- Secure by Default
- Zero Trust Between Services
- Secrets Never Stored in Source Control

---

# Security Domains

The system is divided into distinct trust boundaries.

```text
Internet
    │
    ▼
API Gateway
    │
    ▼
Authenticated Services
    │
    ▼
Internal Asset Services
    │
    ▼
Generation Infrastructure
    │
    ▼
Storage
```

No component should bypass its assigned trust boundary.

---

# Trust Zones

## Zone 1 — Public

Accessible:

- API Gateway
- Authentication

Never expose:

- ComfyUI
- Database
- Workers
- Storage
- Workflow JSON
- Models

---

## Zone 2 — Internal Services

Contains:

- Asset Service
- World Simulation
- Battle Service
- Chat Service

Only authenticated internal traffic.

---

## Zone 3 — Generation Network

Contains:

- ComfyUI
- Asset Workers
- GPU Nodes

Never publicly routable.

---

## Zone 4 — Data

Contains:

- PostgreSQL
- Redis
- Asset Storage
- Models
- Templates

Only internal services may access.

---

# Authentication

Future authentication provider:

```text
OpenID Connect

OAuth2

JWT
```

Every request should resolve:

```text
Identity

↓

Permissions

↓

Action
```

---

# Authorization

Permissions should be role-based.

Suggested roles:

```text
Viewer

Artist

Reviewer

Publisher

Administrator

System
```

---

## Viewer

Allowed:

- Browse assets
- View metadata
- Download published assets

Denied:

- Generate
- Review
- Publish

---

## Artist

Allowed:

- Queue generation
- Submit revisions
- View pending assets

Denied:

- Publish
- Delete

---

## Reviewer

Allowed:

- Approve
- Reject
- Revise
- Replace portrait

Denied:

- Modify workflow registry

---

## Publisher

Allowed:

- Publish approved assets
- Republish assets

Denied:

- Modify prompts
- Modify workflows

---

## Administrator

Full access.

Including:

- Workflow management
- Configuration
- Maintenance
- Recovery

---

## System

Internal service account.

Used by:

- Workers
- Scheduled jobs
- Cleanup
- Validation

Never interactive.

---

# API Security

Every endpoint should validate:

Authentication

↓

Authorization

↓

Input Validation

↓

Business Rules

↓

Execution

↓

Audit

---

# Request Validation

Every request should validate:

UUID

↓

Ownership

↓

Permissions

↓

Object State

↓

Action Allowed

---

# Input Validation

Reject:

- Invalid UUID
- Missing fields
- Oversized payloads
- Invalid enums
- Invalid workflow references
- Unknown asset roles

Never trust client input.

---

# Rate Limiting

Public endpoints:

Recommended:

```text
100 requests / minute
```

Generation endpoints:

Lower limits.

Review endpoints:

Moderate limits.

Internal services:

No public rate limiting.

---

# CSRF Protection

Required for browser-based administrative interfaces.

Use:

- SameSite cookies
- CSRF tokens

Future:

OIDC session integration.

---

# CORS

Only trusted origins.

Never:

```text
*
```

Production origins explicitly configured.

---

# Secrets

Never store:

Passwords

API Keys

JWT Secrets

Database Credentials

Cloud Credentials

In:

Git

Workflow JSON

Templates

Logs

Metadata

---

# Secret Sources

Preferred:

Environment Variables

↓

Secret Manager

↓

Container Secrets

↓

Runtime Injection

---

# Database Security

Least privilege accounts.

Suggested:

```text
migration

application

readonly

backup
```

Never run production services as database owner.

---

# Redis Security

Internal only.

Require authentication in production.

Disable dangerous commands where appropriate.

---

# ComfyUI Security

ComfyUI is an internal component.

Never expose publicly.

Access only through Asset Workers.

Disable:

- Public uploads
- Arbitrary workflow execution
- Administrative interfaces

---

# Workflow Security

Workflow JSON is trusted code.

Treat as executable configuration.

Requirements:

- Version controlled
- Reviewed
- Signed (future)
- Immutable in production

Workers should reject unknown workflow versions.

---

# Prompt Security

Never execute prompt text.

Prompts are data.

Escape prompt values before logging.

Limit maximum prompt size.

Reject binary data.

---

# File Upload Security

Validate:

Extension

↓

MIME Type

↓

Image Decode

↓

Dimensions

↓

Maximum Size

↓

Virus Scan (future)

Never trust file extensions alone.

---

# Storage Security

Pending:

Writable

Approved:

Writable by review process

Published:

Read-only

Archive:

Append only

Models:

Read-only

---

# Published Assets

Published assets are immutable.

Corrections require:

New Version

↓

Review

↓

Publication

Never overwrite published files.

---

# Path Security

Never build paths manually.

Use:

Path Builder Service

Reject:

```text
../

Absolute paths

Invalid separators
```

Prevent path traversal.

---

# Logging Security

Never log:

Passwords

JWT

API Keys

Prompt secrets

Database credentials

Session cookies

Sensitive values should be redacted.

---

# Audit Logging

Every privileged action records:

Timestamp

User

Role

Action

Object

Result

Duration

Client IP

Correlation ID

Audit logs are immutable.

---

# Worker Security

Workers:

Stateless

Authenticated

Limited permissions

No direct user interaction

Can only:

Read pending jobs

Write generated assets

Update owned jobs

---

# GPU Security

GPU servers should:

Remain internal

Run dedicated workloads

Avoid shared public access

Limit model modification

---

# Dependency Security

Before deployment:

Audit:

npm packages

Container images

OS packages

Known CVEs

Future:

Automatic dependency scanning.

---

# Supply Chain Security

Validate:

Workflow sources

Model sources

LoRA sources

Third-party templates

Only approved assets enter production.

---

# Backup Security

Encrypt:

Database backups

Asset backups

Configuration backups

Store separately from production.

---

# Encryption

In Transit:

TLS

At Rest:

Filesystem encryption where appropriate.

Future:

Object storage encryption.

---

# Monitoring

Alert on:

Repeated authentication failures

Privilege escalation

Unexpected workflow execution

Unauthorized publication

Storage access anomalies

Configuration changes

---

# Incident Response

Security incidents:

Contain

↓

Preserve Logs

↓

Identify Cause

↓

Patch

↓

Recover

↓

Review

Never destroy evidence.

---

# Compliance

Maintain:

Audit history

Asset history

Publication history

Review history

Workflow history

Configuration history

Every decision should be traceable.

---

# Future Security Enhancements

Planned:

- Hardware security modules
- Signed workflows
- Signed metadata
- Object integrity hashes
- Tamper detection
- Immutable audit ledger
- Multi-factor authentication
- Fine-grained permissions
- SSO integration
- Automated vulnerability scanning

---

# Acceptance Criteria

The Wishes Asset Pipeline security architecture is complete when:

1. Every service authenticates appropriately.
2. Every privileged action is authorized.
3. ComfyUI remains inaccessible from public networks.
4. Published assets are immutable.
5. Secrets never exist in source control.
6. Every administrative action is audited.
7. Workflow execution is controlled.
8. Storage permissions follow least privilege.
9. Security failures generate alerts.
10. The architecture supports future enterprise security requirements without redesign.