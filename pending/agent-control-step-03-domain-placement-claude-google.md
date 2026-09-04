# Step 03 — Dedicated Agent Control Domain Placement

Created: 2026-09-04
Priority: Critical
Mode: design-and-cost-approval-package
Assigned Agent: claude-google
Allow Edit: planning/docs only

Depends on:
- `pending/agent-control-step-02-security-domain-reconciliation-claude-google.md`

References:
- `pending/reference-agent-control-platform-revised-approved-design.md`
- `pending/agent-control-platform-master-deployment-plan.md`

## Objective

Using Step 02 live-state/security findings, produce the concrete hosting/repository/database placement for the independent `agent-control` platform. No live resource creation occurs in this step.

## Required recommendation

Compare and recommend:

### Repository
- dedicated `agent-control` repository (preferred);
- temporary isolated bootstrap path only if repository provisioning is not immediately available;
- migration/extraction plan for any bootstrap code.

### GCP project boundary
Compare:
- dedicated GCP project for the control platform (preferred for multi-user/multi-project isolation);
- existing management/shared project with strict independent IAM (temporary option);
- explicitly reject application/project-specific placement when it creates cross-project authority coupling.

### Database
Recommend dedicated Cloud SQL PostgreSQL instance and `agent_control` database sizing/topology for initial load. Show lower-cost bootstrap alternatives but identify security/isolation tradeoffs.

### Event transport
Recommend secure internal event transport for outbox dispatch and near-real-time runtime push. Compare approved Redis/Memorystore vs Pub/Sub/other existing platform capability as appropriate. Agent runtimes never connect directly to the event backend.

### Gateway / runtime bridge
Define:
- service names;
- region;
- ingress/auth model;
- outbound agent bridge connectivity;
- private DB/event networking;
- availability/scale-to-zero implications;
- local/Google runtime reachability.

### Identity / secrets
Define:
- Human OIDC provider boundary (no password DB initial release);
- agent device enrollment/short-lived platform token boundary;
- service identities;
- Secret Manager namespace/labels/access for only server-side integration secrets;
- no external provider credential centralization.

## Cost package

Provide estimated monthly standing/light-use cost ranges for each placement choice, with separate line items for:
- Cloud SQL;
- event transport/Redis;
- Cloud Run/API;
- NAT/egress if needed;
- Secret Manager/logging/storage;
- any identity provider charges where known/configurable.

Use live pricing/quotas from authorized environment where available; otherwise clearly mark estimates requiring Human/live verification.

## Security comparison

Score each placement on:
- tenant isolation;
- blast radius;
- IAM complexity;
- credential exposure risk;
- network isolation;
- recovery/rebuild;
- cost;
- operational complexity.

## Human Gate

End with one recommended target architecture and explicit decisions requiring Human approval.

Do not create projects, databases, IAM bindings, secrets, network resources, repositories or service deployments in this step.