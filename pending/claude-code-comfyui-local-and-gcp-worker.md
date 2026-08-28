# Claude Code Task — Complete Local + GCP Remote ComfyUI Routing

## Status

This is the single Claude Code task document for the next phase of the ComfyUI integration.

Assume the secure friend broker already exists and is deployed successfully.

Do not use older ComfyUI task documents as implementation instructions unless they are explicitly referenced by current repository code.

Claude must build/test code only. Human administrators perform production IAM changes and deployments.

---

# 1. Final Architecture

There are two completely separate ComfyUI execution paths.

## Local Path

```text
Claude Code running locally
        │
        │ direct local HTTP
        ▼
http://127.0.0.1:8188
        │
        ▼
Local ComfyUI
```

Requirements:

- no Google authentication
- no Cloudflare authentication
- no Google Secret Manager access
- no Cloud Run dependency
- intended for local development / local generation

## Friend / Remote Path

```text
GCP application / remote execution service
        │
        │ authenticated Google request
        ▼
wishes-comfy-worker
        │
        │ Google ID token
        ▼
wishes-comfy-broker
        │
        │ Secret Manager + CF service token
        ▼
Cloudflare Access
        │
        ▼
https://comfyui.triumphcoding.net
```

Requirements:

- Claude Code running locally must NOT directly invoke the friend broker
- Claude Code on a VM also does not need direct friend-broker access by default
- only the GCP remote worker should invoke the friend broker
- worker has no Secret Manager access
- broker retains all Cloudflare credential handling

---

# 2. Existing Production Broker

The following is already deployed and verified.

```text
Project:
wishes-506905

Project number:
910633976836

Region:
us-west1

Broker Cloud Run service:
wishes-comfy-broker

Broker URL:
https://wishes-comfy-broker-910633976836.us-west1.run.app

Broker runtime identity:
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com

Friend ComfyUI:
https://comfyui.triumphcoding.net
```

The broker already supports:

```text
GET  /health
GET  /system-stats
POST /prompt
GET  /history/{prompt_id}
GET  /outputs/{prompt_id}
GET  /files/{prompt_id}/{output_index}
```

The broker has already been verified to:

- read the Cloudflare Client ID and Client Secret from Secret Manager
- authenticate through Cloudflare Access
- reach the friend's ComfyUI `/system_stats`
- remain private
- expose no generic proxy

Do not reimplement Cloudflare credential handling in the worker.

---

# 3. Identity Change Being Made by Human Administrator

A dedicated GCP service account will become the only normal workload identity allowed to invoke the friend broker:

```text
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
```

It will receive:

```text
roles/run.invoker
```

on:

```text
wishes-comfy-broker
```

It will NOT receive:

```text
roles/secretmanager.secretAccessor
roles/run.admin
roles/run.developer
roles/iam.serviceAccountUser on the broker identity
roles/iam.serviceAccountTokenCreator on the broker identity
```

The old direct broker invocation grant to:

```text
wishes-claude-agent@wishes-506905.iam.gserviceaccount.com
```

will be removed by the human administrator.

Claude must not perform these IAM changes.

---

# 4. Objective

Implement:

1. a shared ComfyUI execution abstraction
2. a local ComfyUI provider
3. a GCP remote worker service
4. a remote worker client intended for trusted GCP application services
5. configuration-based routing
6. tests proving local and remote credential boundaries remain separate
7. human-only container/deployment instructions

---

# 5. Repository Inspection

Before editing:

- inspect existing `server/comfy-broker`
- inspect current asset-generation APIs/services
- inspect current ComfyUI client utilities
- inspect database/asset queue integration
- inspect Docker/Terraform conventions
- inspect tests
- reuse existing broker response models where practical

Do not create duplicate implementations of functionality already present.

---

# 6. Shared Execution Model

Create or adapt a provider abstraction.

Example:

```python
from typing import Protocol


class ComfyProvider(Protocol):
    def system_stats(self) -> dict:
        ...

    def submit_workflow(
        self,
        workflow: dict,
        client_id: str | None = None,
    ) -> str:
        ...

    def get_history(self, prompt_id: str) -> dict:
        ...

    def get_outputs(self, prompt_id: str) -> dict:
        ...

    def download_output(
        self,
        prompt_id: str,
        output_index: int,
        destination,
    ):
        ...

    def execute_workflow(self, workflow: dict) -> dict:
        ...
```

Use existing project interfaces if equivalent structures already exist.

---

# 7. Target Enum / Routing

Add an explicit execution target.

Suggested values:

```text
local
friend_gcp
```

Example:

```python
class ComfyExecutionTarget(str, Enum):
    LOCAL = "local"
    FRIEND_GCP = "friend_gcp"
```

Do not infer the target based on URL.

Do not silently fall back from remote to local or vice versa.

A failed remote request must fail as remote.

---

# 8. Local ComfyUI Provider

Implement:

```text
LocalComfyProvider
```

Default URL:

```text
http://127.0.0.1:8188
```

Configuration:

```text
LOCAL_COMFYUI_BASE_URL=http://127.0.0.1:8188
```

Supported local operations:

```text
GET  /system_stats
POST /prompt
GET  /history/{prompt_id}
GET  /view
```

Expose the same normalized provider behavior used by the remote path.

Local provider must contain:

```text
NO Cloudflare headers
NO Google ID tokens
NO Secret Manager code
NO GCP service-account logic
```

Local Claude Code should be able to use this provider without any GCP authentication.

---

# 9. Local API Flow

Expected local flow:

```text
workflow
   ↓
POST http://127.0.0.1:8188/prompt
   ↓
prompt_id
   ↓
GET /history/{prompt_id}
   ↓
outputs
   ↓
GET /view
```

Reuse normalization behavior from the existing broker where possible.

Do not copy Cloudflare-specific code into the local implementation.

---

# 10. GCP Remote Worker

Create a separate service:

```text
wishes-comfy-worker
```

Suggested repository location:

```text
server/comfy-worker/
```

Adapt naming/structure to repository conventions.

The remote worker does NOT connect directly to the friend's ComfyUI.

It connects only to:

```text
https://wishes-comfy-broker-910633976836.us-west1.run.app
```

---

# 11. Worker Runtime Identity

Production Cloud Run runtime service account:

```text
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com
```

The worker obtains Google credentials through Cloud Run workload identity / Application Default Credentials.

Do not use service-account JSON files.

Do not include GCP credentials in environment variables.

---

# 12. Worker-to-Broker Authentication

Use a Google-signed ID token whose audience is:

```text
https://wishes-comfy-broker-910633976836.us-west1.run.app
```

Send:

```text
Authorization: Bearer <Google ID token>
```

Prefer `google-auth` libraries.

Do not shell out to `gcloud` in production code.

Do not log ID tokens.

---

# 13. Worker Configuration

Non-secret production configuration:

```text
GCP_PROJECT_ID=wishes-506905
GCP_REGION=us-west1

COMFY_BROKER_URL=https://wishes-comfy-broker-910633976836.us-west1.run.app
COMFY_BROKER_AUDIENCE=https://wishes-comfy-broker-910633976836.us-west1.run.app

COMFY_REQUEST_TIMEOUT_SECONDS=30
COMFY_EXECUTION_TIMEOUT_SECONDS=1800
COMFY_POLL_INTERVAL_SECONDS=1
```

No Cloudflare values exist in the worker.

No Secret Manager secret IDs are necessary in the worker.

---

# 14. Worker API

Expose a private Cloud Run API.

At minimum:

```text
GET  /health
GET  /system-stats

POST /prompt
GET  /history/{prompt_id}
GET  /outputs/{prompt_id}
GET  /files/{prompt_id}/{output_index}
```

These may mirror the broker contract intentionally.

No generic proxy route is allowed.

---

# 15. Worker /health

Must not call the broker.

Example:

```json
{
  "status": "ok",
  "service": "wishes-comfy-worker"
}
```

---

# 16. Worker /system-stats

Flow:

```text
caller
   ↓
wishes-comfy-worker
   ↓ Google ID token
wishes-comfy-broker
   ↓
friend ComfyUI
```

Return the broker's safe normalized result.

---

# 17. Worker Workflow Submission

`POST /prompt`

Accept the same safe request structure as the broker:

```json
{
  "prompt": {},
  "client_id": "optional"
}
```

Forward only to the fixed broker URL.

Caller cannot select an arbitrary upstream URL.

Caller cannot pass arbitrary Google auth headers through to the broker.

---

# 18. Worker History / Outputs / Files

Implement:

```text
GET /history/{prompt_id}
GET /outputs/{prompt_id}
GET /files/{prompt_id}/{output_index}
```

The worker should call the matching broker endpoints.

For `/files`, stream broker output to the caller when practical.

Do not expose broker authentication tokens.

---

# 19. Remote Worker Client

Create an application-facing client such as:

```text
clients/comfy_remote_worker_client.py
```

This is intended for application code running in GCP.

It should:

- authenticate to `wishes-comfy-worker` using Google ID token
- not know the broker's Cloudflare credential
- not talk directly to the friend broker

Suggested interface should match `ComfyProvider`.

---

# 20. Important Local Boundary

The normal local Claude Code path must not instantiate `RemoteComfyWorkerClient` automatically.

Local configuration should default to:

```text
COMFY_EXECUTION_TARGET=local
```

The remote path must be explicitly requested by application logic.

Do not require local users to run:

```text
gcloud auth application-default login
```

just to use local ComfyUI.

---

# 21. GCP Application Routing

For application code running in GCP:

```text
target=local
```

is valid only if that runtime actually has a local ComfyUI instance.

Otherwise:

```text
target=friend_gcp
```

uses:

```text
RemoteComfyWorkerClient
```

Do not route `friend_gcp` directly to `wishes-comfy-broker`.

---

# 22. Asset Queue Integration

If the repository already has an asset-generation queue/API, integrate the target explicitly.

Suggested field:

```text
execution_target
```

Allowed initial values:

```text
local
friend_gcp
```

If a provider/target field already exists, reuse it instead of adding a duplicate column.

Persist enough information to determine which executor owns the job.

Do not encode Cloudflare/GCP credentials in job records.

---

# 23. Local Claude Usage

Provide a local CLI/example.

Example:

```bash
python scripts/comfy.py   --target local   run workflow_api.json   --output-dir ./generated
```

This must work without Google credentials when local ComfyUI is available.

---

# 24. GCP Remote Usage

Provide a GCP-side example.

Conceptually:

```python
provider = get_comfy_provider(
    target="friend_gcp"
)

result = provider.execute_workflow(workflow)
```

The code should assume it is running under a GCP identity authorized to invoke `wishes-comfy-worker`.

Do not teach local Claude to impersonate this identity.

---

# 25. Error Isolation

Use distinct errors.

Examples:

```text
LocalComfyUnavailable
RemoteWorkerAuthenticationError
RemoteWorkerUnavailable
RemoteBrokerFailure
RemoteComfyExecutionFailed
WorkflowValidationFailed
ExecutionTimeout
```

A local connection failure must not silently attempt friend GCP.

A remote failure must not silently attempt local ComfyUI.

---

# 26. Security Tests

Add regression tests proving:

1. local provider adds no Cloudflare headers
2. local provider requests no Google token
3. local provider has no Secret Manager dependency
4. worker adds Google ID token to broker calls
5. worker contains no Cloudflare Client ID/Secret dependency
6. worker contains no Secret Manager dependency
7. caller cannot override worker's broker URL
8. caller cannot inject broker Authorization header
9. no generic proxy endpoint exists
10. local default target is `local`
11. remote target is explicit
12. friend remote provider calls worker, not broker directly
13. errors never contain Google ID tokens
14. worker Terraform does not grant Secret Manager access
15. worker Terraform does not grant itself deployment/admin permissions
16. worker Terraform remains private
17. no `allUsers`
18. no `allAuthenticatedUsers`

---

# 27. Functional Tests

Mock all external services.

Test:

```text
Local:
- health/stats equivalent
- prompt submit
- pending history
- completed history
- output normalization
- file download

Remote worker:
- /health
- broker token audience
- /system-stats
- /prompt
- /history
- /outputs
- /files
- broker 401/403 propagation
- timeout
```

Do not hit production broker in unit tests.

---

# 28. Optional Integration Tests

Add separate opt-in integration modes.

Local:

```text
COMFY_LOCAL_INTEGRATION_TEST=1
```

Remote:

```text
COMFY_REMOTE_INTEGRATION_TEST=1
```

Remote integration tests must run only in a properly authorized GCP environment.

Never require remote integration tests for normal local test runs.

---

# 29. Container

Create a production container for:

```text
wishes-comfy-worker
```

Requirements:

- minimal image
- non-root where practical
- no credentials baked in
- bind `$PORT`
- Cloud Run compatible
- no `.env` copied

---

# 30. Worker Terraform

Add Terraform for `wishes-comfy-worker`.

IMPORTANT:

Claude writes but does not apply production Terraform.

Desired service:

```text
name:
wishes-comfy-worker

region:
us-west1

runtime service account:
wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com

authentication:
required
```

Do not grant the worker any Secret Manager permission.

Do not grant public access.

Do not grant the old `wishes-claude-agent` identity invocation access automatically.

Caller IAM for the worker is intentionally configured by the human after deciding which GCP application service should invoke it.

---

# 31. Human-Only Deployment Script

Create or document commands to:

1. build image
2. push to `wishes-services`
3. deploy private Cloud Run worker
4. attach `wishes-comfy-worker` runtime identity
5. configure broker URL
6. verify `/health`
7. verify `/system-stats`

Claude must not run production deployment.

---

# 32. Do Not Modify Broker Security

Do not:

- give worker Secret Manager access
- add Cloudflare values to worker
- make broker public
- make worker public
- grant local Claude broker invocation permission
- create service-account key files
- add a generic forwarding endpoint

---

# 33. Deployment Image

Use:

```text
us-west1-docker.pkg.dev/wishes-506905/wishes-services/wishes-comfy-worker
```

Tag strategy may follow repository convention.

For initial deployment an acceptable tag is:

```text
initial
```

---

# 34. README Architecture

Document explicitly:

```text
LOCAL
Claude Code → Local ComfyUI

REMOTE
GCP app → Comfy Worker → Comfy Broker → Cloudflare → Friend ComfyUI
```

Also include:

> The local Claude Code environment does not require credentials for the friend ComfyUI path. Friend access is intentionally isolated to GCP workloads.

---

# 35. Future VM Claude

Document that moving Claude Code to a VM does not automatically change this boundary.

Default VM behavior:

```text
VM Claude
   ├── local ComfyUI on VM, if present
   └── application/job submission path for friend generation
```

Do not grant VM Claude direct broker access unless a future architecture explicitly requires it.

---

# 36. Acceptance Criteria

Complete when:

- shared Comfy provider abstraction exists
- local provider works without GCP
- local target defaults to `local`
- remote worker service is implemented
- remote worker authenticates to broker using Google ID token
- worker has no Secret Manager dependency
- worker has no Cloudflare credential dependency
- friend remote provider calls worker instead of broker
- asset routing supports explicit target where appropriate
- local and remote paths never silently fail over
- tests pass
- security tests pass
- container is buildable
- Terraform validates
- human deployment commands are documented
- Claude has not deployed production worker
- Claude has not changed IAM

---

# 37. Final Claude Report

Report:

1. files created
2. files modified
3. provider/routing design implemented
4. local ComfyUI URL/config
5. worker URL expected after deployment
6. worker image URI
7. tests run
8. test results
9. security checks
10. Terraform validation
11. exact human build command
12. exact human deploy command
13. exact human `/health` command
14. exact human `/system-stats` command
15. any asset-queue schema changes
16. remaining human-only IAM actions
17. unresolved assumptions

Do not access, print, or request any Cloudflare credentials.
