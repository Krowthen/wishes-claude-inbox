# Claude Code Task — Wire Asset Generation Service to Local and Friend-GCP ComfyUI

## Status

The secure ComfyUI infrastructure is now deployed and verified.

This task is application integration only.

Claude must:
- inspect the existing Wishes asset-generation code
- wire the existing `execution_target` marker into actual dispatch
- preserve the local/remote security boundary
- build and test
- NOT deploy production resources
- NOT change IAM

---

# 1. Verified Production Infrastructure

## GCP Project

```text
Project ID:
wishes-506905

Project number:
910633976836

Region:
us-west1
```

## Friend Broker

```text
Service:
wishes-comfy-broker

URL:
https://wishes-comfy-broker-910633976836.us-west1.run.app

Runtime identity:
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
```

The broker:
- is private
- reads Cloudflare credentials from Secret Manager
- authenticates through Cloudflare Access
- reaches the friend's ComfyUI
- exposes no generic proxy

## Remote Worker

```text
Service:
wishes-comfy-worker

URL:
https://wishes-comfy-worker-910633976836.us-west1.run.app

Runtime identity:
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
```

The worker:
- is private
- has `roles/run.invoker` on the broker
- cannot read either Cloudflare Secret Manager secret
- has zero Cloudflare credential handling of its own
- successfully reaches `/system-stats` through the full chain

Verified chain:

```text
GCP caller
  -> wishes-comfy-worker
  -> Google ID token
  -> wishes-comfy-broker
  -> Secret Manager
  -> Cloudflare Access
  -> friend ComfyUI
```

---

# 2. Existing Application Marker

Claude previously added a minimal column to:

```text
asset_generation_queue
```

named:

```text
execution_target
```

Allowed values:

```text
local
friend_gcp
```

Default:

```text
local
```

This currently acts only as a marker.

The existing Node.js asset-generation service does NOT yet dispatch based on it.

That is the gap this task closes.

---

# 3. Required Final Architecture

## Local

```text
Node asset-generation service / local tooling
  -> execution_target=local
  -> local ComfyUI
  -> http://127.0.0.1:8188
```

Local requirements:
- no Google auth
- no Secret Manager
- no Cloudflare auth
- no worker
- no broker

## Friend / Remote

```text
Node asset-generation service running in an authorized GCP workload
  -> execution_target=friend_gcp
  -> wishes-comfy-worker
  -> wishes-comfy-broker
  -> Cloudflare Access
  -> friend ComfyUI
```

Remote requirements:
- asset service calls worker only
- asset service must NOT call broker directly
- asset service must NOT know Cloudflare credentials
- use Google workload identity / ADC
- no service-account JSON keys

---

# 4. First Step — Inspect Existing Code

Before editing, inspect:

- asset-generation queue schema and migrations
- Node.js asset-generation service
- queue worker/consumer
- asset status lifecycle
- existing ComfyUI integrations
- workflow serialization/loading
- output download/storage code
- retry/error handling
- existing HTTP client conventions
- GCP deployment/Terraform for the asset service
- existing runtime service account(s)
- tests
- local development configuration

Do not create duplicate services or duplicate queue consumers if an existing path can be extended.

Report in the final summary which Node service/process actually owns asset-generation queue execution.

---

# 5. Explicit Routing

Read `execution_target` from the queued generation record.

Allowed values:

```text
local
friend_gcp
```

Routing must be explicit.

Pseudo-code:

```ts
switch (job.execution_target) {
  case "local":
    return localComfyExecutor.execute(job);

  case "friend_gcp":
    return remoteWorkerExecutor.execute(job);

  default:
    throw new UnsupportedExecutionTargetError(...);
}
```

Do not infer target from URLs.

Do not silently fall back between local and remote.

---

# 6. Local Executor

Implement or adapt a Node local ComfyUI client/executor.

Default URL:

```text
http://127.0.0.1:8188
```

Configuration:

```text
LOCAL_COMFYUI_BASE_URL=http://127.0.0.1:8188
```

Expected ComfyUI API flow:

```text
POST /prompt
GET  /history/{prompt_id}
GET  /view
```

Reuse existing project workflow/output normalization where appropriate.

Local executor must not:
- request Google ID tokens
- import Secret Manager clients
- set Cloudflare headers
- call the worker
- call the broker

---

# 7. Friend-GCP Executor

Implement or adapt a Node remote-worker client.

Fixed production worker URL:

```text
https://wishes-comfy-worker-910633976836.us-west1.run.app
```

Configuration:

```text
COMFY_WORKER_URL=https://wishes-comfy-worker-910633976836.us-west1.run.app
COMFY_WORKER_AUDIENCE=https://wishes-comfy-worker-910633976836.us-west1.run.app
```

The client may use config overrides for tests/dev, but application callers must not be able to supply arbitrary upstream URLs through job payloads.

The friend-GCP executor calls only the worker endpoints:

```text
GET  /system-stats
POST /prompt
GET  /history/{prompt_id}
GET  /outputs/{prompt_id}
GET  /files/{prompt_id}/{output_index}
```

Never call `wishes-comfy-broker` directly from the Node asset service.

---

# 8. Node Google Authentication

For `friend_gcp`, use Google-supported ADC/workload identity to mint an ID token for:

```text
https://wishes-comfy-worker-910633976836.us-west1.run.app
```

Prefer the repository's existing Google Auth library if present.

If not present, use the official Node Google Auth library:

```text
google-auth-library
```

Conceptually:

```ts
const auth = new GoogleAuth();
const client = await auth.getIdTokenClient(workerAudience);
```

Then use the authenticated client to call the worker.

Do not:
- shell out to `gcloud`
- use a JSON key
- read a credential file from the repository
- persist ID tokens
- log ID tokens
- pass the worker token into queue rows

---

# 9. Caller IAM Is Human-Managed

Claude must inspect the existing GCP deployment configuration and identify the service account used by the Node asset-generation workload.

Claude must NOT change IAM.

In the final report provide:

```text
Expected production caller service account:
<email or unresolved>
```

If no existing dedicated workload identity exists, recommend a dedicated service account name, but do not create it.

The human administrator will later grant only:

```text
roles/run.invoker
```

on:

```text
wishes-comfy-worker
```

to that actual application runtime identity.

Do not grant the Node service direct access to:
- wishes-comfy-broker
- Secret Manager Cloudflare credentials
- worker service account impersonation
- broker service account impersonation

---

# 10. Queue Lifecycle

Preserve the current queue lifecycle and status semantics.

Do not invent a parallel queue.

Integrate target routing into the existing lifecycle.

A typical lifecycle may be:

```text
pending
-> inprogress
-> approved/completed OR failed/rejected
```

Use the actual existing state model rather than renaming states without cause.

Persist:
- execution target
- prompt/workflow identifier
- returned ComfyUI prompt ID where appropriate
- output asset references
- safe failure reason

Do not persist:
- Google ID tokens
- Authorization headers
- Cloudflare credentials
- cookies

---

# 11. Submission vs Polling

ComfyUI jobs may outlive a single short HTTP request.

Reuse the current queue worker's retry/polling model where possible.

For remote generation:

```text
POST worker /prompt
-> prompt_id

poll:
GET worker /history/{prompt_id}
or
GET worker /outputs/{prompt_id}

download:
GET worker /files/{prompt_id}/{output_index}
```

Do not keep one unnecessarily long HTTP request open if the existing queue architecture already supports asynchronous polling.

Respect existing timeout/retry behavior where sensible.

---

# 12. Output Storage

Preserve the existing asset-storage model.

Downloaded output from either target should flow through the same normalized storage/finalization path.

Do not create separate asset schemas solely for local vs friend output unless technically required.

Record provenance/target if the existing model supports metadata.

---

# 13. Failure Semantics

Create or reuse explicit errors such as:

```text
UnsupportedExecutionTarget
LocalComfyUnavailable
RemoteWorkerAuthenticationError
RemoteWorkerUnavailable
RemoteGenerationFailed
GenerationTimeout
InvalidWorkflow
OutputDownloadFailed
```

Requirements:
- local failure never retries through friend_gcp
- friend_gcp failure never retries through local
- authentication failure remains a remote failure
- no sensitive auth material in errors

---

# 14. Local Development Behavior

Default:

```text
execution_target=local
```

A local development environment must be able to run local generation without:
- gcloud
- Application Default Credentials
- Cloud Run
- Cloudflare
- Secret Manager

Do not instantiate Google Auth merely because the package is imported.

Only the `friend_gcp` path should request Google credentials.

---

# 15. Remote Development Behavior

If a developer explicitly selects:

```text
friend_gcp
```

outside an authorized GCP environment, fail clearly with a safe authentication/configuration error.

Do not silently use the developer's Owner credentials.

Do not add instructions requiring local users to run:

```text
gcloud auth application-default login
```

for the normal local workflow.

---

# 16. API Validation

Where requests create asset-generation jobs, validate:

```text
execution_target
```

against the allowed enum.

If omitted, preserve current DB default:

```text
local
```

Reject arbitrary strings.

Do not accept caller-provided:
- broker URL
- worker Authorization header
- Cloudflare headers
- Secret Manager secret IDs

---

# 17. Security Tests

Add regression tests proving:

1. local jobs call local ComfyUI only
2. local jobs request no Google ID token
3. local jobs do not call worker
4. `friend_gcp` jobs call worker only
5. `friend_gcp` jobs never call broker directly
6. remote path uses Google ID-token auth
7. worker audience equals configured worker service URL
8. arbitrary job payload cannot override worker URL
9. arbitrary job payload cannot inject Authorization header
10. no Cloudflare headers exist in Node remote client
11. no Cloudflare secret IDs exist in Node remote client
12. no Secret Manager dependency is introduced for this path
13. no silent local/remote failover
14. invalid execution target is rejected
15. ID tokens are never logged
16. Authorization headers are never logged
17. queue/database records never store auth tokens
18. local remains the default target
19. normal local unit tests need no Google credentials

---

# 18. Functional Tests

Mock external services.

Test local path:
- submit workflow
- pending history
- completion
- output download
- ComfyUI unavailable
- timeout

Test remote path:
- worker auth client creation
- prompt submission
- pending state
- completed outputs
- file download
- worker 401/403
- worker 5xx
- timeout
- malformed response

Test queue integration:
- default local
- explicit local
- explicit friend_gcp
- status changes
- output finalization
- failure behavior

Do not hit production worker or broker in unit tests.

---

# 19. Optional Integration Tests

Keep integration tests opt-in.

Suggested flags:

```text
COMFY_LOCAL_INTEGRATION_TEST=1
COMFY_REMOTE_INTEGRATION_TEST=1
```

Local integration may target:

```text
http://127.0.0.1:8188
```

Remote integration must only run from a properly authorized GCP workload.

Never make remote integration part of the standard local test suite.

---

# 20. Configuration Documentation

Document at minimum:

```text
COMFY_EXECUTION_TARGET=local
LOCAL_COMFYUI_BASE_URL=http://127.0.0.1:8188

COMFY_WORKER_URL=https://wishes-comfy-worker-910633976836.us-west1.run.app
COMFY_WORKER_AUDIENCE=https://wishes-comfy-worker-910633976836.us-west1.run.app
```

Use actual existing configuration naming where a project standard already exists.

Do not put credentials into `.env.example`.

---

# 21. Do Not Change Production Infrastructure

Claude must not:
- deploy the Node service
- deploy the worker
- deploy the broker
- alter Cloud Run IAM
- grant `roles/run.invoker`
- create service accounts
- read Secret Manager
- add Cloudflare secrets
- run Terraform apply
- create service-account keys

Terraform/code changes describing desired application configuration are allowed if consistent with the repository, but do not apply them.

---

# 22. Database

The `execution_target` column already exists.

Do not create a second target column.

If the current schema only has a raw text constraint and the project convention warrants a stronger enum/check constraint, evaluate it carefully but avoid unnecessary schema churn.

A fresh `reset-db.sh` previously validated the existing column.

If you modify database schema:
- include migration/reset consistency
- rerun DB validation
- report exact changes

---

# 23. Deployment Readiness

At completion, the application code should be ready for a human to:

1. build/deploy the Node asset service
2. identify its production runtime service account
3. grant that identity `roles/run.invoker` on `wishes-comfy-worker`
4. test one `friend_gcp` queue job
5. test one `local` queue job independently

Claude should provide the exact IAM command template with the discovered caller service account, but must not execute it.

---

# 24. Acceptance Criteria

Complete when:

- existing asset-generation queue is the single queue
- `execution_target` drives actual dispatch
- local target uses local ComfyUI
- friend_gcp target uses worker
- Node service never calls broker directly
- Node service contains no Cloudflare credential logic
- remote auth uses Google workload identity/ADC
- no local Google auth requirement
- no silent fallback
- outputs share the existing asset finalization path
- tests pass
- security regression tests pass
- DB validation passes if touched
- no production deployment occurred
- no IAM change occurred

---

# 25. Final Claude Report

Report:

1. exact Node service/process that consumes `asset_generation_queue`
2. files created
3. files modified
4. routing design implemented
5. local ComfyUI path
6. friend_gcp worker path
7. Google auth implementation
8. queue lifecycle integration
9. output-storage integration
10. tests run
11. test counts/results
12. security checks
13. DB validation
14. expected production caller service account email
15. exact human IAM command to grant caller `roles/run.invoker` on `wishes-comfy-worker`
16. exact build/deploy commands for the human, if this Node service is currently deployable
17. one local end-to-end test command
18. one remote end-to-end test procedure
19. unresolved gaps or assumptions

Do not access, request, print, or persist any Cloudflare credential.
