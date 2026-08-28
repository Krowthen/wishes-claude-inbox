# 17 — Operations Runbook

## Objective

This document defines the operational procedures for managing the Wishes AI Asset Pipeline after deployment.

Unlike the implementation documents, this runbook is intended for day-to-day operations.

Its purpose is to ensure the Asset Pipeline remains:

- Stable
- Recoverable
- Observable
- Maintainable
- Scalable

This document should be used by developers, administrators, and future automation systems.

---

# Operational Goals

The production system should always maintain:

- Maximum uptime
- Recoverable failures
- Consistent asset quality
- Deterministic publication
- Complete audit history

No manual action should permanently modify published assets.

---

# Service Inventory

The production asset platform consists of:

```text
API Gateway

↓

Asset Service

↓

Asset Workers

↓

PostgreSQL

↓

Redis

↓

ComfyUI

↓

Generated Asset Storage

↓

Unity Client
```

Every service must expose:

- Health
- Ready
- Version
- Metrics

---

# Service Startup Order

Always start services in the following order.

```text
PostgreSQL

↓

Redis

↓

Storage

↓

Workflow Validation

↓

Asset Service

↓

Asset Workers

↓

ComfyUI

↓

API Gateway
```

Validation occurs after every startup.

---

# Service Shutdown Order

Shutdown in reverse order.

```text
API Gateway

↓

Workers

↓

Asset Service

↓

ComfyUI

↓

Redis

↓

PostgreSQL
```

Workers should finish active jobs before terminating.

---

# Daily Operational Checklist

Verify:

- Database healthy
- Redis healthy
- ComfyUI healthy
- Storage available
- Queue processing
- No failed workers
- Published storage writable
- Workflow validation passing

---

# Hourly Health Checks

Automatically verify:

```text
Database

Redis

Asset Service

Workers

ComfyUI

Storage

Workflow Registry

GPU

Disk Space
```

Log every result.

---

# Queue Monitoring

Monitor:

```text
Queued Jobs

Running Jobs

Failed Jobs

Retry Count

Average Wait Time

Average Generation Time
```

Alert thresholds:

```text
Queue > 500 jobs

Worker Offline

Average Wait > 10 minutes

Failure Rate > 10%
```

---

# Worker Health

Each worker should publish:

```text
Worker UUID

Current Job

Heartbeat

Memory Usage

CPU

GPU

Jobs Completed

Failures
```

Workers missing heartbeat for more than:

```text
60 seconds
```

should be marked offline.

---

# ComfyUI Monitoring

Monitor:

- Availability
- Queue depth
- Memory usage
- GPU utilization
- Model load failures
- Workflow failures

Alert on:

```text
Workflow Failure

GPU Out of Memory

No Response

Repeated Timeouts

Output Missing
```

---

# Storage Monitoring

Track:

Pending

Approved

Published

Rejected

Archive

Temp

Monitor:

- total size
- growth rate
- free disk space
- file count

Alert when:

```text
Storage > 80%

Disk Full

Directory Missing

Permission Failure
```

---

# Database Monitoring

Track:

Connections

Locks

Slow Queries

Migration Version

Replication Status

Future:

Read replicas

Failover

Backups

---

# Backup Schedule

Back up:

```text
PostgreSQL

Approved Assets

Published Assets

Templates

Workflow Registry

Configuration
```

Suggested schedule:

```text
Database

Daily

Assets

Daily Incremental

Weekly Full

Configuration

Every Change
```

---

# Restore Procedure

Database:

1. Stop services
2. Restore backup
3. Run validation
4. Restart services

Assets:

1. Restore storage
2. Validate hashes
3. Validate metadata
4. Resume publication

---

# Disaster Recovery

## Database Failure

Actions:

Stop workers.

Restore backup.

Validate schema.

Restart.

---

## Storage Failure

Actions:

Disable publication.

Restore storage.

Validate checksums.

Resume workers.

---

## ComfyUI Failure

Actions:

Pause queue.

Restart ComfyUI.

Validate workflows.

Resume workers.

---

## Worker Failure

Actions:

Worker heartbeat timeout.

Mark worker offline.

Return claimed jobs to queue.

Launch replacement worker.

---

# Asset Recovery

If generated asset missing:

Locate metadata.

Locate job.

Locate source.

Determine:

```text
Regenerate

Restore

Archive

Delete
```

Never manually recreate metadata.

---

# Workflow Recovery

If workflow becomes invalid:

Disable workflow.

Notify administrators.

Reject new requests.

Existing assets remain valid.

---

# Queue Recovery

If queue becomes stuck:

Identify blocked worker.

Return orphaned jobs.

Resume processing.

Never delete queued jobs without audit.

---

# Publication Recovery

If publication fails:

Leave asset approved.

Retry publication.

Do not regenerate.

Publication is deterministic.

---

# Manual Operations

Supported manual actions:

Approve

Reject

Archive

Republish

Retry Generation

Retry Publication

Replace Portrait

Rebuild Metadata

Reindex Storage

Every manual action must create an audit record.

---

# Scheduled Maintenance

Weekly:

Validate workflows.

Validate storage.

Clean temp directory.

Check rejected asset growth.

Review failed jobs.

Monthly:

Verify backups.

Test restore.

Review performance.

Update documentation.

Quarterly:

Dependency updates.

Workflow review.

Security audit.

Capacity planning.

---

# Performance Monitoring

Collect:

Queue latency

Generation duration

Review duration

Publication duration

Renderer duration

Worker utilization

GPU utilization

Storage throughput

Database latency

---

# Capacity Planning

Monitor trends:

Asset growth

Daily generations

Approval rate

Storage growth

GPU utilization

Database size

Worker throughput

Scale before bottlenecks occur.

---

# Security Operations

Verify:

No secrets committed.

Workflow directory unchanged.

Model directory protected.

Published assets immutable.

API authentication active.

Review permissions enforced.

---

# Audit Requirements

Every operation records:

Timestamp

Actor

Service

Object

Object UUID

Action

Result

Duration

Error

Audit records are immutable.

---

# Operational Metrics

Recommended dashboards:

Generation Dashboard

Review Dashboard

Publication Dashboard

Storage Dashboard

GPU Dashboard

Database Dashboard

Worker Dashboard

System Health Dashboard

---

# Incident Response

Incident priorities:

```text
P1

Production Down

P2

Generation Stopped

P3

Publication Failure

P4

Workflow Failure

P5

Minor Operational Issue
```

Each incident should produce:

Root Cause

Timeline

Resolution

Follow-up Tasks

---

# Maintenance Windows

During maintenance:

Pause queue.

Finish active jobs.

Disable publication.

Perform maintenance.

Validate.

Resume workers.

Resume publication.

---

# Release Procedure

Deployment:

1. Backup
2. Apply migrations
3. Validate
4. Deploy Asset Service
5. Deploy Workers
6. Validate ComfyUI
7. Run smoke tests
8. Resume production

Never deploy directly to production without validation.

---

# Production Acceptance

Operations are considered production-ready when:

- Monitoring operational
- Alerting operational
- Backups verified
- Restore tested
- Queue recovery validated
- Worker recovery validated
- Storage recovery validated
- Documentation complete
- On-call procedures documented
- Smoke tests successful

---

# Long-Term Operations Vision

The operational platform should eventually become largely self-managing.

Future automation should:

- Detect failures
- Recover workers
- Validate workflows
- Scale GPU resources
- Optimize queue scheduling
- Recommend prompt improvements
- Detect identity drift
- Predict storage expansion

Automation should **never** bypass the human review and publication process.

The role of operations evolves from manual intervention to oversight, while preserving deterministic behavior, traceability, and complete auditability across the entire Wishes Asset Pipeline.