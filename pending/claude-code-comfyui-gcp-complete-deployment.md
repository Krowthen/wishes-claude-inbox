# Claude Code Deployment Task — Secure GCP Broker for Remote ComfyUI Behind Cloudflare Access

## Status

This document is the **single source of instructions for Claude Code** for this deployment.

Assume all earlier ComfyUI API, Cloudflare, broker, and credential-management task documents have been removed. Do not depend on them.

The human administrator will separately perform all credential-bearing and console/account operations. Claude must not request, print, store, or inspect the real Cloudflare Access Client ID or Client Secret.

---

# 1. Goal

Build and deploy a secure API broker on Google Cloud that lets an AI agent/application submit workflows to a friend's ComfyUI instance that is already protected by Cloudflare Access.

Target architecture:

```text
Claude / application workload
        │
        │ Google-authenticated request
        │ Cloud Run Invoker only
        ▼
┌──────────────────────────────────┐
│ GCP Cloud Run                    │
│ wishes-comfy-broker              │
│                                  │
│ approved ComfyUI API only        │
│ no generic HTTP proxy            │
└──────────────┬───────────────────┘
               │
               │ broker runtime service account
               ▼
┌──────────────────────────────────┐
│ Google Secret Manager            │
│                                  │
│ comfy-cf-access-client-id        │
│ comfy-cf-access-client-secret    │
└──────────────┬───────────────────┘
               │
               │ inject headers internally
               ▼
┌──────────────────────────────────┐
│ Cloudflare Access                │
│ Service Auth policy              │
└──────────────┬───────────────────┘
               ▼
┌──────────────────────────────────┐
│ Friend's ComfyUI                 │
│ HTTPS only                       │
└──────────────────────────────────┘
```

Claude/application callers must never need:

```text
CF-Access-Client-Id
CF-Access-Client-Secret
```

The broker alone is authorized to read those values.

---

# 2. Security Invariants

These are mandatory.

## 2.1 Never store real secret material in the repository

Never put real Cloudflare credentials in:

```text
.env
.env.local
.env.production
CLAUDE.md
README.md
Terraform .tf files
Terraform tfvars
Terraform state inputs
Dockerfile
docker-compose files
source code
test fixtures
shell scripts
CI files
example files
command-line arguments
logs
exception messages
```

Example placeholders are allowed.

## 2.2 Terraform must not manage secret payloads

Terraform may create:

```text
google_secret_manager_secret
```

resources and IAM bindings.

Terraform must **not** create real:

```text
google_secret_manager_secret_version
```

resources containing Cloudflare credentials.

Reason: secret payloads managed by Terraform can be retained in Terraform state.

The human administrator will add secret versions separately.

## 2.3 Separate identities

Create/use two distinct service accounts:

```text
wishes-comfy-broker
wishes-claude-agent
```

Responsibilities:

```text
wishes-comfy-broker
    CAN:
      - run the broker
      - access exactly the two Comfy Cloudflare secrets

    MUST NOT:
      - have broad project admin roles


wishes-claude-agent
    CAN:
      - invoke the Cloud Run broker

    MUST NOT:
      - access either Cloudflare secret
      - impersonate wishes-comfy-broker
      - administer Cloud Run
      - administer Secret Manager
```

## 2.4 Broker must remain private

Cloud Run must require authentication.

Do not use:

```text
--allow-unauthenticated
```

Do not grant:

```text
allUsers
allAuthenticatedUsers
```

Cloud Run invocation should be controlled with IAM.

## 2.5 No generic proxy

Do not expose:

```text
/proxy
/request
/forward
/fetch-url
```

or any endpoint allowing arbitrary upstream hosts, methods, headers, or URLs.

The broker may talk only to the configured ComfyUI base URL and only through explicitly supported operations.

## 2.6 HTTPS upstream only

`COMFYUI_BASE_URL` must be `https://`.

Reject `http://` in deployed configurations.

Do not disable TLS certificate verification.

## 2.7 Never expose upstream Cloudflare headers

Inbound users must not be able to provide, override, retrieve, or inspect:

```text
CF-Access-Client-Id
CF-Access-Client-Secret
```

---

# 3. Human-Supplied Non-Secret Configuration

Claude may expect these values to be supplied by the human administrator or Terraform variables:

```text
GCP_PROJECT_ID
GCP_REGION
COMFYUI_BASE_URL
```

Suggested default region if the human has not standardized another region:

```text
us-west1
```

Do not guess the real friend hostname.

Use a placeholder such as:

```text
https://comfy-api.example.com
```

until the human supplies the actual Cloudflare-protected ComfyUI URL.

---

# 4. Google Cloud Resources

Infrastructure-as-code should create the following.

## APIs

Ensure these APIs are enabled:

```text
run.googleapis.com
secretmanager.googleapis.com
artifactregistry.googleapis.com
cloudbuild.googleapis.com
iam.googleapis.com
iamcredentials.googleapis.com
```

If existing project conventions manage APIs elsewhere, follow repository conventions instead of duplicating resources.

## Service Accounts

Create:

```text
wishes-comfy-broker@PROJECT_ID.iam.gserviceaccount.com
wishes-claude-agent@PROJECT_ID.iam.gserviceaccount.com
```

Use descriptive display names.

## Secret Manager secret containers

Create metadata containers only:

```text
comfy-cf-access-client-id
comfy-cf-access-client-secret
```

Recommended automatic replication unless the repository has an explicit regional secret policy.

Do not add actual secret versions.

## Secret IAM

Grant:

```text
roles/secretmanager.secretAccessor
```

to:

```text
wishes-comfy-broker
```

on each of the two secrets individually.

Do not grant Secret Manager access project-wide if secret-level IAM can be used.

Do not grant the role to:

```text
wishes-claude-agent
```

## Artifact Registry

Create or reuse an Artifact Registry Docker repository.

Suggested name:

```text
wishes-services
```

Follow existing project conventions if one already exists.

## Cloud Run

Create:

```text
wishes-comfy-broker
```

Requirements:

```text
authentication required
runtime service account = wishes-comfy-broker
region = configured GCP region
HTTPS managed by Cloud Run
minimum instances = 0 by default
maximum instances = configurable
```

Suggested initial resource profile:

```text
CPU: 1
Memory: 512Mi
Concurrency: 20
Request timeout: 300s
Min instances: 0
Max instances: 5
```

Do not block a Cloud Run HTTP request for an entire ComfyUI generation.

Submission and polling must remain separate operations.

## Cloud Run Invoker IAM

Grant:

```text
roles/run.invoker
```

on `wishes-comfy-broker` to:

```text
wishes-claude-agent
```

Do not give `wishes-claude-agent` Cloud Run Admin.

---

# 5. Repository Structure

Adapt to existing project structure rather than creating duplicates.

Preferred layout if no equivalent exists:

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

clients/
└── comfy_broker_client.py

examples/
├── comfyui_generate_example.py
└── workflows/
    └── README.md

infra/
└── comfy_broker/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── versions.tf
    └── README.md

scripts/
├── deploy_comfy_broker.sh
├── verify_comfy_broker.py
└── comfy.py

tests/
└── comfy_broker/
    └── ...
```

If the repository has existing Terraform modules, Python packaging, test directories, or deployment conventions, integrate with them.

Do not create a parallel architecture unnecessarily.

---

# 6. Broker Implementation

Use Python unless the repository already standardizes another backend language for services.

Preferred framework:

```text
FastAPI
```

Preferred server:

```text
uvicorn
```

Use a persistent HTTP client/session for outbound ComfyUI calls.

`httpx` is preferred with FastAPI.

---

# 7. Configuration Model

Broker configuration must include:

```text
GCP_PROJECT_ID
COMFYUI_BASE_URL
CF_ACCESS_CLIENT_ID_SECRET_ID=comfy-cf-access-client-id
CF_ACCESS_CLIENT_SECRET_SECRET_ID=comfy-cf-access-client-secret

COMFYUI_REQUEST_TIMEOUT_SECONDS=30
COMFYUI_POLL_HINT_SECONDS=1
```

Do not provide the secret values themselves through environment variables.

Validate:

```text
COMFYUI_BASE_URL starts with https://
no trailing slash after normalization
secret IDs are non-empty
project ID is non-empty
```

---

# 8. Secret Manager Credential Provider

Implement a Google Secret Manager-backed credential provider.

Suggested data model:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class CloudflareCredentials:
    client_id: str
    client_secret: str
```

Suggested interface:

```python
class CredentialProvider:
    def get_credentials(self) -> CloudflareCredentials:
        ...
```

Implementation:

```text
GoogleSecretManagerCredentialProvider
```

Use:

```python
google.cloud.secretmanager.SecretManagerServiceClient
```

Read:

```text
projects/{project_id}/secrets/comfy-cf-access-client-id/versions/latest

projects/{project_id}/secrets/comfy-cf-access-client-secret/versions/latest
```

Use Application Default Credentials.

On Cloud Run this must rely on the attached `wishes-comfy-broker` service account.

Do not use a service-account JSON key.

## Caching

Cache credentials in memory for a short bounded period so the broker does not call Secret Manager for every outbound request.

Suggested TTL:

```text
300 seconds
```

Requirements:

- no secret values in cache logs
- expiration causes re-read
- process restart clears cache
- no disk persistence

Support disabling the cache in tests.

---

# 9. Upstream ComfyUI Client

Create an internal ComfyUI client.

It must always add:

```text
CF-Access-Client-Id: <secret value>
CF-Access-Client-Secret: <secret value>
```

These headers are internal only.

Supported upstream endpoints:

```text
GET  /system_stats
POST /prompt
GET  /history/{prompt_id}
GET  /view
```

Optional if required later:

```text
POST /upload/image
POST /interrupt
```

Do not expose unsupported upstream endpoints automatically.

---

# 10. Broker Public API

Initial broker endpoints:

```text
GET  /health
GET  /system-stats

POST /prompt
GET  /history/{prompt_id}
GET  /outputs/{prompt_id}
GET  /files/{prompt_id}/{output_index}
```

The broker is private at the Cloud Run IAM layer.

---

# 11. GET /health

Purpose:

```text
verify broker container/process health
```

It must not contact Secret Manager or ComfyUI.

Example:

```json
{
  "status": "ok",
  "service": "wishes-comfy-broker"
}
```

Do not expose version control commit information unless existing service conventions require it.

---

# 12. GET /system-stats

Call upstream:

```text
GET {COMFYUI_BASE_URL}/system_stats
```

Return a safe JSON representation.

If Cloudflare responds with authentication failure, normalize the response.

Example:

```json
{
  "error": "upstream_authentication_failed",
  "message": "Cloudflare Access rejected the broker credentials."
}
```

Do not return Cloudflare request headers.

---

# 13. POST /prompt

Accept:

```json
{
  "prompt": {
    "...": "ComfyUI API-format workflow"
  },
  "client_id": "optional UUID/string"
}
```

Requirements:

- `prompt` must be a JSON object
- generate UUID if `client_id` is omitted
- reject oversized input using a reasonable request/body limit if supported by the framework/deployment
- do not accept arbitrary upstream URL or headers

Forward:

```text
POST {COMFYUI_BASE_URL}/prompt
```

with:

```json
{
  "prompt": {},
  "client_id": "..."
}
```

Normalize successful response:

```json
{
  "prompt_id": "...",
  "number": 12,
  "node_errors": {}
}
```

If `prompt_id` is missing, raise a broker upstream-protocol error.

If `node_errors` is non-empty, return a 422-style validation response containing safe ComfyUI validation details.

---

# 14. GET /history/{prompt_id}

Validate `prompt_id`.

Allow only a conservative identifier character set.

For example:

```regex
^[A-Za-z0-9._-]{1,128}$
```

Forward only to:

```text
GET {COMFYUI_BASE_URL}/history/{prompt_id}
```

Return normalized JSON.

Do not permit path traversal.

---

# 15. GET /outputs/{prompt_id}

Retrieve history and normalize generated outputs.

Do not assume a fixed output node.

Inspect every node in the completed history result.

At minimum support:

```json
{
  "outputs": {
    "9": {
      "images": [
        {
          "filename": "ComfyUI_00001_.png",
          "subfolder": "",
          "type": "output"
        }
      ]
    }
  }
}
```

Normalized output model:

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

Also handle execution status/failure information if present in the ComfyUI history schema.

If history is not ready:

```json
{
  "prompt_id": "...",
  "status": "pending",
  "outputs": []
}
```

Use an appropriate HTTP success code for a valid pending state.

---

# 16. GET /files/{prompt_id}/{output_index}

Purpose:

download a generated output through the broker.

The broker must derive file metadata from trusted ComfyUI history.

Do not accept a caller-provided filename/path for this endpoint.

Flow:

```text
prompt_id + index
    ↓
load history
    ↓
normalize output list
    ↓
select indexed output
    ↓
call upstream /view using metadata from history
    ↓
stream response
```

Upstream call resembles:

```text
GET /view
  ?filename=<history-derived filename>
  &subfolder=<history-derived subfolder>
  &type=<history-derived type>
```

Validate metadata before forwarding.

Stream bytes without loading excessively large files completely into memory when practical.

Preserve a safe content type.

Set a safe download filename.

---

# 17. Cloudflare Response Handling

Detect at least:

```text
401
403
302/redirect to login
HTML response when JSON was expected
```

Normalize auth failures.

Example internal exception:

```text
UpstreamAuthenticationError
```

Example API response:

```json
{
  "error": "upstream_authentication_failed",
  "message": "Cloudflare Access rejected the broker credentials."
}
```

Never include:

```text
client ID
client secret
request headers
Cloudflare cookie
CF_Authorization
```

---

# 18. Logging

Use structured logging if the project supports it.

Allowed:

```text
request_id
prompt_id
route
status code
duration
upstream status
output count
```

Forbidden:

```text
Cloudflare Client ID
Cloudflare Client Secret
Secret Manager payload
full outbound headers
authorization bearer tokens
Google ID tokens
```

Add explicit redaction for:

```text
authorization
cf-access-client-id
cf-access-client-secret
cookie
set-cookie
```

Do not log complete request bodies by default because workflows may eventually contain sensitive user data.

---

# 19. Cloud Run Caller Authentication

The broker must require Cloud Run IAM authentication.

Create an application-side client using Google-signed ID tokens.

For requests from a GCP workload:

1. Use Application Default Credentials / attached service identity.
2. Acquire an ID token with audience equal to the Cloud Run service URL.
3. Send:

```text
Authorization: Bearer <Google-signed ID token>
```

The caller service account is:

```text
wishes-claude-agent
```

It receives only:

```text
roles/run.invoker
```

on this Cloud Run service.

---

# 20. Application-Facing Broker Client

Create:

```text
clients/comfy_broker_client.py
```

Suggested interface:

```python
class ComfyBrokerClient:
    def __init__(
        self,
        base_url: str,
        request_timeout: float = 30,
        poll_interval: float = 1,
        execution_timeout: float = 600,
    ):
        ...

    def health(self) -> dict:
        ...

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

    def wait_for_completion(self, prompt_id: str) -> dict:
        ...

    def execute_workflow(self, workflow: dict) -> dict:
        ...
```

This client must know nothing about Cloudflare authentication.

It may know how to obtain Google Cloud Run ID tokens.

---

# 21. Cloud Run ID Token Helper

Prefer the Google authentication libraries rather than shelling out to `gcloud` inside application code.

Expected dependencies:

```text
google-auth
requests or httpx
```

Use a Google ID token with the broker Cloud Run URL as the audience.

Do not cache ID tokens past their expiration.

Do not print them.

For unit tests, mock the credential/token provider.

---

# 22. End-to-End Example Flow

Create:

```text
examples/comfyui_generate_example.py
```

The example must be fully independent of Cloudflare credentials.

Flow:

```text
1. Load broker URL
2. Obtain Google ID token
3. GET /health
4. GET /system-stats
5. Load ComfyUI API-format workflow JSON
6. Modify configured workflow inputs if requested
7. POST /prompt
8. Capture prompt_id
9. Poll /outputs/{prompt_id}
10. Stop on completed or failed status
11. Download each generated output through /files
12. Print resulting local file paths
```

Do not keep one Cloud Run request open while the workflow executes.

---

# 23. Workflow Files

Create:

```text
examples/workflows/README.md
```

Explain that ComfyUI workflows sent to the API must be in API-compatible workflow format.

Do not invent actual model/checkpoint filenames.

If no real workflow is checked into the repository, include a placeholder/example schema only and explain where the human should place an exported workflow.

Do not assume node IDs.

---

# 24. Explicit Workflow Mutation

Provide a helper.

Example:

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

Application configuration should explicitly define node/input mappings.

Do not use fuzzy guessing to find prompt nodes in production workflow code.

---

# 25. Example CLI

Create:

```text
scripts/comfy.py
```

Commands:

```bash
python scripts/comfy.py health

python scripts/comfy.py stats

python scripts/comfy.py submit \
  examples/workflows/YOUR_WORKFLOW_API.json

python scripts/comfy.py history PROMPT_ID

python scripts/comfy.py outputs PROMPT_ID

python scripts/comfy.py download PROMPT_ID 0 ./output.png

python scripts/comfy.py run \
  examples/workflows/YOUR_WORKFLOW_API.json \
  --output-dir ./generated
```

CLI must use the broker only.

CLI must not accept Cloudflare credentials.

---

# 26. Terraform

Create or update Terraform for the broker.

Important:

```text
NO REAL SECRET VALUES IN TERRAFORM
```

Terraform should manage:

```text
Google APIs if project conventions allow
service accounts
secret containers
secret IAM
Artifact Registry if needed
Cloud Run service
Cloud Run IAM invoker binding
configuration variables
outputs
```

Terraform should not manage:

```text
Cloudflare token value
Secret Manager secret payload/version
Google user credentials
service-account key files
```

## Useful Terraform outputs

Output:

```text
broker_cloud_run_url
broker_service_account_email
agent_service_account_email
cloudflare_client_id_secret_name
cloudflare_client_secret_secret_name
```

Do not output secret values.

---

# 27. Container

Add a production container.

Requirements:

```text
non-root user where practical
minimal base image
pinned major/minor runtime
no credentials baked into image
no .env copied
health endpoint
PORT environment support
```

Cloud Run supplies:

```text
PORT
```

Bind application server inside the container to:

```text
0.0.0.0:$PORT
```

This is correct inside Cloud Run even though the Cloud Run service itself remains IAM-protected.

---

# 28. Deployment Script

Create:

```text
scripts/deploy_comfy_broker.sh
```

or integrate with existing deployment tooling.

It may:

```text
validate required non-secret variables
build image
push image
apply/deploy Cloud Run revision
print broker URL
```

It must not:

```text
prompt for Cloudflare secret
read local .env containing Cloudflare credentials
write secret versions
print secrets
grant unauthenticated access
```

The human administrator handles secret-version creation separately.

---

# 29. Human Boundary

Add an explicit message to deployment output when secret versions are missing.

Example:

```text
Infrastructure is deployed, but Cloudflare credentials must be added
to Google Secret Manager by the human administrator before upstream
ComfyUI connectivity will succeed.
```

Do not ask Claude to obtain the credentials.

---

# 30. Tests

Use the repository's existing test framework.

At minimum implement tests for:

## Credential provider

1. Correct secret resource names are requested.
2. Secret values are not logged.
3. Cache honors TTL.
4. Cache can be disabled in tests.
5. Secret Manager error is converted safely.

## ComfyUI upstream client

6. Cloudflare headers are added internally.
7. Caller cannot override Cloudflare headers.
8. `GET /system_stats` works.
9. `POST /prompt` sends expected payload.
10. `prompt_id` is required in successful response.
11. ComfyUI node validation errors are surfaced safely.
12. `GET /history/{prompt_id}` uses validated prompt ID.
13. 401 is normalized.
14. 403 is normalized.
15. HTML login page is normalized.
16. redirect/login response is normalized.
17. secret values never appear in exception strings.

## Broker API

18. `/health` does not contact Secret Manager.
19. `/system-stats` uses upstream client.
20. `/prompt` validates workflow object.
21. `/prompt` generates client ID when omitted.
22. `/history/{prompt_id}` rejects traversal/input attacks.
23. `/outputs/{prompt_id}` handles pending.
24. `/outputs/{prompt_id}` handles completed images.
25. output parser supports multiple nodes.
26. `/files/{prompt_id}/{index}` derives metadata from history.
27. `/files` rejects invalid output indexes.
28. no generic proxy route exists.

## Cloud Run application client

29. Google ID token is requested for correct audience.
30. Authorization bearer header is included.
31. Cloudflare headers are never present.
32. poll loop handles pending.
33. poll loop handles success.
34. poll loop handles failure.
35. execution timeout works.
36. downloads write expected bytes.

## Security regression

37. Search application source/config fixtures for forbidden real-secret variable usage.
38. Terraform contains no `secret_data` for these credentials.
39. Terraform does not grant Secret Manager accessor to agent service account.
40. Cloud Run IAM does not grant `allUsers`.
41. broker service account receives accessor only on intended secrets.

---

# 31. Static Security Checks

Add a lightweight repository check that fails if code accidentally introduces patterns such as:

```text
CF_ACCESS_CLIENT_SECRET=
CF-Access-Client-Secret: literal-value
```

Do not make the scanner so broad that it rejects documentation placeholders.

If the repository already has secret scanning, integrate with it instead.

---

# 32. Integration Test Mode

Support an opt-in integration test.

It may run only when explicitly enabled, for example:

```text
COMFY_BROKER_INTEGRATION_TEST=1
```

The integration test talks to the deployed broker.

It must not retrieve Secret Manager values.

Suggested sequence:

```text
GET /health
GET /system-stats
optional POST /prompt with human-provided test workflow
poll outputs
```

Skip safely when not configured.

---

# 33. Observability

Provide basic logs for:

```text
broker request status
upstream ComfyUI latency
Cloudflare auth failure count
workflow submission count
history request count
download count
```

Do not log workflow bodies or generated media contents by default.

Cloud Logging through Cloud Run stdout/stderr is sufficient initially.

Do not add a large observability stack for this task.

---

# 34. Error Model

Use stable machine-readable error codes.

Examples:

```text
configuration_error
secret_access_failed
upstream_authentication_failed
upstream_unavailable
upstream_invalid_response
workflow_validation_failed
prompt_not_found
output_not_ready
output_not_found
execution_failed
request_validation_failed
```

Return safe human-readable messages.

---

# 35. Secret Rotation Behavior

The design must support Cloudflare secret rotation without redeploying source code.

Expected human rotation flow:

```text
1. rotate Cloudflare service token secret
2. add new Secret Manager secret version
3. wait for broker cache TTL or restart Cloud Run revision/instances
4. verify /system-stats
5. retire old credential when appropriate
```

The broker should read `versions/latest`.

Do not hardcode Secret Manager version numbers in application source.

---

# 36. Cloudflare Assumptions

The human administrator/friend will configure Cloudflare separately.

Assume:

```text
a dedicated Cloudflare Access Service Token exists
a Service Auth policy allows that token to the ComfyUI Access app
the friend retains normal browser login for human UI access
ComfyUI remains behind Cloudflare
the underlying ComfyUI port is not publicly exposed
```

The broker uses Cloudflare's standard service-token headers:

```text
CF-Access-Client-Id
CF-Access-Client-Secret
```

Do not automate browser authentication.

Do not store or reuse `CF_Authorization` browser cookies.

---

# 37. Optional Future Hardening — Do Not Block Initial Deployment

Document, but do not require in the first deployment:

## Static Cloud Run egress IP

Cloudflare Service Auth can additionally require source IP ranges.

If the owner wants IP-restricted access later, route Cloud Run egress through GCP networking/NAT with a stable outbound address and add that address/range as a Cloudflare Access requirement.

Do not add this networking complexity unless requested.

## VPC Service Controls

Potential later hardening for Secret Manager.

Not required for initial deployment.

## Cloud Armor / external API gateway

Not necessary while Cloud Run IAM is the authentication boundary.

---

# 38. Human Administrator Checklist — Documentation Only

Add a README section containing this checklist, but do not perform these steps or request secret values.

```text
[ ] GCP project selected and billing enabled
[ ] required Google APIs enabled
[ ] Terraform/infrastructure applied
[ ] Cloudflare Service Token created by friend/admin
[ ] Cloudflare Service Auth policy added to ComfyUI application
[ ] Client ID inserted into Secret Manager manually
[ ] Client Secret inserted into Secret Manager manually
[ ] broker deployed
[ ] broker /health verified
[ ] broker /system-stats verified
[ ] test workflow exported from ComfyUI API format
[ ] workflow submitted
[ ] generated output downloaded
[ ] agent identity verified to have run.invoker only
[ ] agent verified to have no Secret Manager access
```

---

# 39. Acceptance Criteria

The implementation is complete when all of the following are true.

## Infrastructure

- GCP resources are reproducibly defined.
- Cloud Run broker requires authentication.
- Broker uses dedicated runtime service account.
- Agent uses separate identity.
- Agent has only Cloud Run invocation access required for this system.
- Secret containers exist.
- Terraform contains no real secret values.

## Credentials

- Real Cloudflare credentials are entered only by the human administrator.
- Broker reads credentials from Secret Manager.
- No service-account JSON keys are required for Cloud Run.
- Credentials are not present in repository, logs, images, or Terraform state input.

## API

- `/health` works.
- `/system-stats` reaches remote ComfyUI through Cloudflare.
- `/prompt` submits API-format workflows.
- `/history/{prompt_id}` works.
- `/outputs/{prompt_id}` normalizes generated outputs.
- `/files/{prompt_id}/{index}` downloads generated files.
- no generic proxy exists.

## Caller

- Claude/application authenticates to Cloud Run using Google identity.
- Claude/application does not know the Cloudflare credentials.
- Claude/application cannot read the two secrets through IAM.

## Testing

- Unit tests pass.
- Security regression tests pass.
- Optional integration check succeeds after the human adds real credentials.

---

# 40. Final End-to-End Example

Expected external behavior:

```text
Claude/Application
        │
        │ Google ID token
        ▼
POST https://BROKER_RUN_URL/prompt
{
  "prompt": { ... }
}
        │
        ▼
Broker
        │
        ├── Secret Manager:
        │      read CF client ID/secret internally
        │
        └── POST https://FRIEND_COMFY_URL/prompt
             CF-Access-Client-Id: ***
             CF-Access-Client-Secret: ***
        │
        ▼
{
  "prompt_id": "..."
}
```

Caller polls:

```text
GET https://BROKER_RUN_URL/outputs/PROMPT_ID
Authorization: Bearer <Google ID token>
```

Pending:

```json
{
  "prompt_id": "...",
  "status": "pending",
  "outputs": []
}
```

Completed:

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

Caller downloads:

```text
GET https://BROKER_RUN_URL/files/PROMPT_ID/0
Authorization: Bearer <Google ID token>
```

The caller never receives or sends the Cloudflare service-token credentials.

---

# 41. Implementation Order

Execute in this order:

```text
1. inspect repository conventions
2. implement broker models/config/errors
3. implement Secret Manager provider
4. implement upstream ComfyUI client
5. implement output normalization
6. implement FastAPI routes
7. implement Cloud Run-authenticated application client
8. implement CLI/example
9. implement tests
10. implement Terraform
11. implement container/deployment files
12. update documentation
13. run test suite
14. run static/security checks
15. report human-only remaining actions
```

Do not stop to ask the human for Cloudflare credentials.

If real secret values are missing, complete all code/infrastructure work possible and clearly report that the human must add the secret versions before the live `/system-stats` integration check can pass.

---

# 42. Final Claude Report

At completion, report:

1. repository structure inspected
2. files created
3. files modified
4. infrastructure resources added
5. GCP region/project variables expected
6. broker service account
7. agent service account
8. secret container names
9. Cloud Run service name
10. build/deploy command
11. Terraform plan/apply command
12. test commands
13. test results
14. security checks run
15. Cloudflare credentials confirmed absent from repository/state inputs
16. broker authentication model
17. remaining human-only actions
18. exact command the human can use to verify `/health`
19. exact command the human can use to verify `/system-stats`
20. exact command/example for submitting one ComfyUI API workflow

Never print or ask for real credential values.

---

# Official References

Cloudflare Access service tokens:
https://developers.cloudflare.com/cloudflare-one/access-controls/service-credentials/service-tokens/

Cloudflare Access policies:
https://developers.cloudflare.com/cloudflare-one/access-controls/policies/

Google Secret Manager authentication:
https://cloud.google.com/secret-manager/docs/authentication

Google Secret Manager IAM/access:
https://cloud.google.com/secret-manager/docs/manage-access-to-secrets

Google Secret Manager best practices:
https://cloud.google.com/secret-manager/docs/best-practices

Cloud Run authentication overview:
https://cloud.google.com/run/docs/authenticating/overview

Cloud Run service-to-service authentication:
https://cloud.google.com/run/docs/authenticating/service-to-service
