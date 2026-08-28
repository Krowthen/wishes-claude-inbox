# Claude Code Task — Build Secure ComfyUI Broker for GCP

## IMPORTANT: Security Boundary

This is the single Claude Code task document for the ComfyUI integration.

Assume all older ComfyUI task documents are obsolete.

Claude must **build and test the code**, but must **NOT deploy the secret-bearing broker to Google Cloud** and must **NOT access Google Secret Manager**.

The first deployment is a human-admin action.

Reason:

> Any identity that can deploy arbitrary code to a Cloud Run service running as the secret-reading service account can effectively cause that code to read or expose the secrets. Therefore Claude must not have deploy/update permission on the broker service.

After deployment, Claude/application callers will have only `roles/run.invoker`.

---

# Deployment Configuration

Use these exact non-secret values:

```text
GCP_PROJECT_ID=wishes-506905
GCP_PROJECT_NUMBER=910633976836
GCP_REGION=us-west1

COMFYUI_BASE_URL=https://comfyui.triumphcoding.net

ARTIFACT_REGISTRY_REPOSITORY=wishes-services

BROKER_SERVICE_NAME=wishes-comfy-broker
BROKER_SERVICE_ACCOUNT=wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com

AGENT_SERVICE_ACCOUNT=wishes-claude-agent@wishes-506905.iam.gserviceaccount.com

CF_ACCESS_CLIENT_ID_SECRET_ID=comfy-cf-access-client-id
CF_ACCESS_CLIENT_SECRET_SECRET_ID=comfy-cf-access-client-secret
```

---

# Human Infrastructure Already Completed

The human administrator has already completed the following.

Do not recreate these resources unless inspection shows they are missing.

## APIs enabled

```text
artifactregistry.googleapis.com
cloudbuild.googleapis.com
iam.googleapis.com
iamcredentials.googleapis.com
run.googleapis.com
secretmanager.googleapis.com
```

## Service accounts created

```text
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
wishes-claude-agent@wishes-506905.iam.gserviceaccount.com
```

## Secret Manager containers created

```text
comfy-cf-access-client-id
comfy-cf-access-client-secret
```

Each has an enabled secret version containing the real Cloudflare credential.

Claude must not retrieve those versions.

## Secret IAM configured

Only the broker service account has:

```text
roles/secretmanager.secretAccessor
```

on the two ComfyUI Cloudflare secrets.

The agent service account does not have Secret Manager access.

## Artifact Registry created

```text
us-west1-docker.pkg.dev/wishes-506905/wishes-services
```

## Human deployer permission

The human administrator is allowed to attach:

```text
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
```

to the Cloud Run service.

---

# Target Architecture

```text
Claude / application
       │
       │ authenticated Google request
       │ roles/run.invoker only
       ▼
┌──────────────────────────────┐
│ Cloud Run                    │
│ wishes-comfy-broker          │
│                              │
│ runtime identity:            │
│ wishes-comfy-broker          │
└──────────────┬───────────────┘
               │
               │ Secret Manager API
               ▼
┌──────────────────────────────┐
│ Google Secret Manager        │
│                              │
│ comfy-cf-access-client-id    │
│ comfy-cf-access-client-secret│
└──────────────┬───────────────┘
               │
               │ Cloudflare Access headers
               ▼
┌──────────────────────────────┐
│ Cloudflare Access            │
│ Service Auth                 │
└──────────────┬───────────────┘
               ▼
https://comfyui.triumphcoding.net
               │
               ▼
            ComfyUI
```

---

# Hard Security Requirements

Claude must never:

```text
read the real secret versions
print the real secret versions
request that the user paste credentials
store credentials in .env
store credentials in source code
store credentials in Terraform
store credentials in Terraform state input
create service-account JSON keys
use browser Cloudflare cookies
disable TLS verification
make the broker unauthenticated
grant itself Secret Manager access
grant itself broker deployment permission
deploy/update the broker service
```

Claude must not execute commands such as:

```bash
gcloud secrets versions access ...
```

Claude must not create or use:

```text
CF_ACCESS_CLIENT_ID=<real value>
CF_ACCESS_CLIENT_SECRET=<real value>
```

---

# Threat Model

Assume an autonomous coding agent may:

- read repository files
- run tests
- execute shell commands in its workspace
- inspect environment variables exposed to it

Therefore the design must ensure the agent can use the broker without possession of the upstream Cloudflare credential.

Also assume:

> deployment permission to the secret-bearing broker is equivalent to indirect secret access.

For that reason Claude does not receive Cloud Run Admin/Developer deployment permissions for this service.

---

# Part 1 — Repository Inspection

Inspect the repository and follow existing conventions.

Determine:

- Python packaging structure
- existing API framework
- test framework
- Docker conventions
- Terraform structure
- logging conventions
- CI conventions

Prefer integration with existing architecture.

Do not create duplicate frameworks if equivalent infrastructure already exists.

---

# Part 2 — Broker Service

Implement the broker in the repository.

Preferred structure if no equivalent exists:

```text
services/
└── comfy_broker/
    ├── __init__.py
    ├── app.py
    ├── config.py
    ├── credentials.py
    ├── comfy_client.py
    ├── models.py
    ├── outputs.py
    ├── errors.py
    └── README.md
```

Use FastAPI unless repository conventions specify another framework.

Use `httpx` for upstream requests if consistent with project dependencies.

---

# Part 3 — Configuration

Runtime configuration:

```text
GCP_PROJECT_ID=wishes-506905
COMFYUI_BASE_URL=https://comfyui.triumphcoding.net

CF_ACCESS_CLIENT_ID_SECRET_ID=comfy-cf-access-client-id
CF_ACCESS_CLIENT_SECRET_SECRET_ID=comfy-cf-access-client-secret

COMFYUI_REQUEST_TIMEOUT_SECONDS=30
COMFYUI_CREDENTIAL_CACHE_TTL_SECONDS=300
```

The credential values themselves are never environment variables.

Validate that:

```text
COMFYUI_BASE_URL uses https://
the URL is normalized without trailing slash
secret IDs are non-empty
```

---

# Part 4 — Google Secret Manager Provider

Implement:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class CloudflareCredentials:
    client_id: str
    client_secret: str
```

and a provider abstraction such as:

```python
class CredentialProvider:
    def get_credentials(self) -> CloudflareCredentials:
        ...
```

Production implementation:

```text
GoogleSecretManagerCredentialProvider
```

Use:

```python
google.cloud.secretmanager.SecretManagerServiceClient
```

Read:

```text
projects/wishes-506905/secrets/comfy-cf-access-client-id/versions/latest

projects/wishes-506905/secrets/comfy-cf-access-client-secret/versions/latest
```

Use Application Default Credentials.

In Cloud Run, ADC will resolve to:

```text
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com
```

Do not use a service-account key file.

## Credential cache

Cache credentials only in process memory.

Suggested TTL:

```text
300 seconds
```

Never persist them to disk.

Never log them.

Allow the cache to be replaced/mockable in tests.

---

# Part 5 — Upstream ComfyUI Client

Implement an internal client for:

```text
https://comfyui.triumphcoding.net
```

Every upstream request must internally inject:

```text
CF-Access-Client-Id
CF-Access-Client-Secret
```

using the Secret Manager provider.

The caller must never control or override those headers.

Supported upstream operations:

```text
GET  /system_stats
POST /prompt
GET  /history/{prompt_id}
GET  /view
```

Do not create a generic upstream request mechanism.

---

# Part 6 — Broker API

Expose:

```text
GET  /health
GET  /system-stats

POST /prompt
GET  /history/{prompt_id}
GET  /outputs/{prompt_id}
GET  /files/{prompt_id}/{output_index}
```

No generic proxy endpoint is allowed.

---

# GET /health

Must not contact:

```text
Secret Manager
Cloudflare
ComfyUI
```

Response:

```json
{
  "status": "ok",
  "service": "wishes-comfy-broker"
}
```

---

# GET /system-stats

Call:

```text
GET https://comfyui.triumphcoding.net/system_stats
```

through Cloudflare Access.

Return safe JSON.

Normalize upstream auth failures.

Do not return Cloudflare headers.

---

# POST /prompt

Accept:

```json
{
  "prompt": {},
  "client_id": "optional"
}
```

Requirements:

- prompt must be a JSON object
- generate a UUID if client_id is missing
- caller cannot specify upstream URL
- caller cannot specify upstream Cloudflare headers

Forward to:

```text
POST https://comfyui.triumphcoding.net/prompt
```

Expected response:

```json
{
  "prompt_id": "...",
  "number": 12,
  "node_errors": {}
}
```

Require `prompt_id`.

Surface node validation errors safely.

---

# GET /history/{prompt_id}

Validate prompt identifier using a conservative allowlist such as:

```regex
^[A-Za-z0-9._-]{1,128}$
```

Do not permit traversal or arbitrary path construction.

Forward only to:

```text
/history/{prompt_id}
```

---

# GET /outputs/{prompt_id}

Normalize ComfyUI history into:

```json
{
  "prompt_id": "...",
  "status": "completed",
  "outputs": [
    {
      "index": 0,
      "node_id": "9",
      "media_type": "image",
      "filename": "ComfyUI_00001_.png",
      "subfolder": "",
      "folder_type": "output",
      "download_path": "/files/PROMPT_ID/0"
    }
  ]
}
```

Do not assume node `9`.

Iterate all output nodes.

Support pending:

```json
{
  "prompt_id": "...",
  "status": "pending",
  "outputs": []
}
```

Support failed execution when ComfyUI exposes failure status.

---

# GET /files/{prompt_id}/{output_index}

Do not accept filename/path from the caller.

Derive file information from trusted prompt history:

```text
prompt_id
  ↓
history
  ↓
normalized outputs
  ↓
output_index
  ↓
filename/subfolder/type
  ↓
upstream /view
```

Forward only history-derived values to:

```text
GET /view
```

Stream output bytes when practical.

---

# Part 7 — Safe Cloudflare Failure Handling

Detect:

```text
401
403
302/login redirect
HTML login page when JSON expected
```

Normalize to:

```json
{
  "error": "upstream_authentication_failed",
  "message": "Cloudflare Access rejected the broker credentials."
}
```

Never include:

```text
Client ID
Client Secret
Cloudflare request headers
Cloudflare cookies
CF_Authorization
```

---

# Part 8 — Logging

Allowed:

```text
request ID
prompt ID
route
duration
status code
upstream status
output count
```

Forbidden:

```text
secret values
authorization token
Google ID token
Cloudflare headers
cookies
complete workflow body by default
```

Redact at minimum:

```text
authorization
cookie
set-cookie
cf-access-client-id
cf-access-client-secret
```

---

# Part 9 — Agent-Facing Client

Create:

```text
clients/comfy_broker_client.py
```

It must authenticate to the private Cloud Run broker using Google identity.

Suggested interface:

```python
class ComfyBrokerClient:
    def health(self) -> dict: ...
    def system_stats(self) -> dict: ...
    def submit_workflow(self, workflow: dict, client_id: str | None = None) -> str: ...
    def get_history(self, prompt_id: str) -> dict: ...
    def get_outputs(self, prompt_id: str) -> dict: ...
    def download_output(self, prompt_id: str, output_index: int, destination): ...
    def wait_for_completion(self, prompt_id: str) -> dict: ...
    def execute_workflow(self, workflow: dict) -> dict: ...
```

It must contain no Cloudflare authentication logic.

---

# Part 10 — Cloud Run Authentication

The deployed broker will require IAM authentication.

The caller identity will eventually be:

```text
wishes-claude-agent@wishes-506905.iam.gserviceaccount.com
```

The caller will receive only:

```text
roles/run.invoker
```

on the broker service.

Application calls should obtain a Google ID token with audience equal to the Cloud Run service URL.

Prefer Google authentication libraries.

Do not shell out to `gcloud` from production application code.

Do not log bearer tokens.

---

# Part 11 — Example Workflow Flow

Create:

```text
examples/comfyui_generate_example.py
```

Flow:

```text
1. authenticate to broker
2. GET /health
3. GET /system-stats
4. load API-format ComfyUI workflow
5. mutate explicit node inputs if configured
6. POST /prompt
7. receive prompt_id
8. poll /outputs/{prompt_id}
9. detect completed/failed
10. download outputs through /files
```

Do not leave one HTTP request open for the whole generation.

---

# Workflow Mutation Helper

Implement an explicit helper:

```python
def set_node_input(
    workflow: dict,
    node_id: str,
    input_name: str,
    value,
) -> None:
    try:
        workflow[node_id]["inputs"][input_name] = value
    except KeyError as exc:
        raise KeyError(
            f"Workflow node/input not found: {node_id}.{input_name}"
        ) from exc
```

Do not fuzzy-guess workflow nodes in production.

---

# Part 12 — CLI

Create:

```text
scripts/comfy.py
```

Commands:

```bash
python scripts/comfy.py health
python scripts/comfy.py stats
python scripts/comfy.py submit WORKFLOW.json
python scripts/comfy.py history PROMPT_ID
python scripts/comfy.py outputs PROMPT_ID
python scripts/comfy.py download PROMPT_ID 0 ./output.png
python scripts/comfy.py run WORKFLOW.json --output-dir ./generated
```

The CLI calls only the broker.

No Cloudflare credentials are accepted.

---

# Part 13 — Container

Create a Cloud Run-compatible production image.

Requirements:

```text
minimal Python base
non-root user where practical
no credentials baked in
no .env copied
listen on $PORT
production server
```

Inside Cloud Run, bind:

```text
0.0.0.0:$PORT
```

This is correct because Cloud Run itself remains IAM protected.

---

# Part 14 — Infrastructure as Code

Create/update Terraform that describes the desired infrastructure state.

IMPORTANT:

> Claude may write Terraform, but must not run `terraform apply` for the secret-bearing broker.

Terraform may reference existing resources or import/state-manage them according to repository conventions.

Existing resources:

```text
project: wishes-506905
region: us-west1

service accounts:
  wishes-comfy-broker
  wishes-claude-agent

secrets:
  comfy-cf-access-client-id
  comfy-cf-access-client-secret

artifact registry:
  wishes-services
```

Terraform must never contain a real secret payload.

Terraform must never create a real `google_secret_manager_secret_version` with credential data.

Terraform should describe:

```text
Cloud Run broker service
broker runtime service account
environment configuration
service-level IAM
Secret Manager access bindings if appropriate
Artifact Registry reference
```

Do not grant `allUsers`.

Do not grant `allAuthenticatedUsers`.

Do not grant Secret Manager access to the agent.

Do not grant deployment/update permissions to the agent.

---

# Part 15 — Human Deployment Script

Create a human-admin deployment script or documented commands.

Suggested:

```text
scripts/deploy_comfy_broker.sh
```

IMPORTANT:

Claude may WRITE and TEST the script structure.

Claude must NOT execute the production deployment.

The script must:

```text
build image
push image to:
us-west1-docker.pkg.dev/wishes-506905/wishes-services

deploy:
wishes-comfy-broker

region:
us-west1

runtime service account:
wishes-comfy-broker@wishes-506905.iam.gserviceaccount.com

COMFYUI_BASE_URL:
https://comfyui.triumphcoding.net

require authentication
```

It must never:

```text
read Cloudflare secret payloads
print secret payloads
grant unauthenticated access
grant Claude deploy permission
```

---

# Part 16 — Tests

Use the repository's standard framework.

At minimum test:

## Secret provider

1. correct secret resource names
2. no secret values logged
3. cache TTL
4. cache disabled/mockable
5. safe Secret Manager failures

## Upstream ComfyUI

6. Cloudflare headers injected internally
7. caller cannot override headers
8. `/system_stats`
9. `/prompt`
10. missing `prompt_id`
11. node validation error
12. validated history identifier
13. 401 handling
14. 403 handling
15. HTML login handling
16. redirect handling
17. secret absent from exception strings

## Broker

18. health does not contact secrets/upstream
19. system stats uses upstream
20. prompt validation
21. client ID generation
22. prompt ID traversal rejection
23. outputs pending
24. outputs completed
25. multiple output nodes
26. file metadata derives from history
27. invalid output index
28. no generic proxy route

## Agent client

29. correct Google ID-token audience
30. bearer token sent
31. no Cloudflare headers
32. pending polling
33. completed polling
34. failed execution
35. timeout
36. download

## Security regression

37. no real Cloudflare secret variables in application config
38. Terraform contains no secret payload
39. agent has no Secret Manager IAM binding
40. no unauthenticated Cloud Run IAM
41. agent has no deploy/update IAM permission
42. application client has no upstream Cloudflare credential dependency

---

# Part 17 — Static Secret Checks

Integrate with repository secret scanning if present.

At minimum flag accidental code/config patterns such as:

```text
CF_ACCESS_CLIENT_SECRET=<literal>
CF-Access-Client-Secret: <literal>
```

Do not reject documentation placeholders.

---

# Part 18 — README

Document:

```text
Claude/application
    ↓
private broker

broker
    ↓
Google Secret Manager

broker
    ↓
Cloudflare Access

Cloudflare
    ↓
ComfyUI
```

Include this explicit warning:

> Do not grant the Claude/application identity Secret Manager access or deployment permission to the secret-bearing broker. Either permission would weaken the credential isolation boundary.

---

# Part 19 — Human-Only Actions After Claude Finishes

Claude must finish by providing exact commands for a human administrator to perform these actions.

Claude must not execute them.

Human actions will include:

```text
1. review code and Terraform
2. build/push production image
3. deploy first Cloud Run revision as wishes-comfy-broker
4. confirm Cloud Run requires authentication
5. run /health
6. run /system-stats
7. grant wishes-claude-agent roles/run.invoker on this broker only
8. verify wishes-claude-agent cannot access Secret Manager
9. run one test ComfyUI workflow
10. download generated output
```

---

# Acceptance Criteria

Claude's task is complete when:

- broker code is implemented
- Cloudflare credentials are read only by production broker credential provider
- no real secret is required for unit tests
- no generic proxy exists
- application-facing client uses Google auth
- output polling/downloading is implemented
- production container exists
- IaC is prepared
- human-only deployment instructions are prepared
- tests pass
- static/security checks pass
- Claude has not accessed the real secrets
- Claude has not deployed the broker
- Claude has not changed IAM permissions
- Claude has not granted itself additional GCP privileges

---

# Final Claude Report

Report:

1. files created
2. files modified
3. tests run
4. test results
5. static/security checks
6. container build command
7. intended image URI
8. human deployment command
9. expected Cloud Run service configuration
10. exact human `/health` verification command
11. exact human `/system-stats` verification command
12. exact human command to grant `roles/run.invoker` to `wishes-claude-agent`
13. exact human command to verify the agent cannot read the secrets
14. example workflow execution command
15. remaining assumptions or blockers

Never print or request the real Cloudflare Client ID or Client Secret.
