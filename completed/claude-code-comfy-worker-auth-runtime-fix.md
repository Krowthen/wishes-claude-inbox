# Claude Code Patch Task — Fix Comfy Worker Broker Authentication Runtime Failure

## Context

The GCP worker is deployed successfully.

Production worker:
- Service: `wishes-comfy-worker`
- URL: `https://wishes-comfy-worker-910633976836.us-west1.run.app`
- Runtime service account: `wishes-comfy-worker@wishes-506905.iam.gserviceaccount.com`

Production broker:
- Service: `wishes-comfy-broker`
- URL: `https://wishes-comfy-broker-910633976836.us-west1.run.app`

IAM is correct: `wishes-comfy-worker` has `roles/run.invoker` on `wishes-comfy-broker`.

Observed production behavior:
- `GET worker /health` → `200 OK`
- `GET worker /system-stats` → `502`
- External response:
```json
{
  "error": "remote_broker_failure",
  "message": "The broker call failed."
}
```
- Worker logs show `broker_call_failed`.
- Broker logs show no corresponding request from the worker.
- Deployed worker env contains the correct `COMFY_BROKER_URL` and `COMFY_BROKER_AUDIENCE`.

## Primary suspected cause

Inspect the worker's Google ID-token code.

The production image build installed `google-auth` and `httpx`, but the Cloud Build package list did not show `requests`.

If the worker uses:
```python
from google.auth.transport.requests import Request
```
or `google.oauth2.id_token.fetch_id_token(...)` through the requests transport, the runtime image must include the `requests` dependency.

This failure would appear only when the first broker call attempts to mint an ID token, matching production behavior.

Do not assume this is definitely the cause. Inspect the actual source and prove the failure path.

## Task

### 1. Inspect
Inspect:
- `clients/comfy_broker_client.py`
- `server/comfy-worker/`
- `server/comfy-worker/pyproject.toml`
- `server/comfy-worker/Dockerfile`

Identify the exact exception path that can produce `remote_broker_failure` / `broker_call_failed` before an HTTP request reaches the broker.

### 2. Fix the runtime dependency or token code
If `google.auth.transport.requests` is used and `requests` is missing, add an explicit runtime dependency:
```text
requests>=2.32
```
or use the appropriate supported `google-auth` extra if that is already the repository convention.

Do not rely on packages accidentally present in a developer virtualenv.
Do not introduce service-account JSON keys.
Do not change IAM.
Do not bypass Cloud Run authentication.

### 3. Improve safe error logging
The current production log only shows `broker_call_failed`, hiding the diagnostic reason.

Improve internal logging to include:
- exception class
- safe exception message
- operation/route

Never log:
- Google ID token
- Authorization header
- Cloudflare credentials
- cookies
- Secret Manager values

Keep the existing safe external API response unchanged.

### 4. Add regression tests
Add tests proving:
1. Production worker dependencies support the Google ID-token transport used by the code.
2. The broker token provider can be instantiated in the production package environment.
3. Token-generation failures are logged safely.
4. ID tokens never appear in logs.
5. Authorization headers never appear in logs.
6. Broker failures still return the safe normalized external error.
7. Existing local-vs-remote security boundaries remain unchanged.

If practical, add a lightweight image/runtime smoke check that verifies the required Google auth transport imports successfully inside the built container.

### 5. Container verification
Build the worker image locally using the same repository-root build context as production:
```text
docker build -f server/comfy-worker/Dockerfile .
```
Verify:
- application starts
- `GET /health` succeeds
- Google auth transport/token-provider dependencies import successfully

Do not attempt to access the production broker using local Owner credentials.

### 6. Run full validation
Run:
- worker tests
- shared provider tests
- security regression tests
- full relevant pytest suite
- `terraform validate` if Terraform files changed

Do not deploy to GCP.
Do not change IAM.

## Security requirements

Do not:
- access Secret Manager
- read or print Cloudflare credentials
- create service-account keys
- run `gcloud run deploy`
- run `terraform apply`
- grant IAM roles
- make worker or broker public
- change the broker credential model

## Final report

Provide:
1. exact root cause
2. files changed
3. dependency change, if any
4. tests run
5. test results
6. Docker/runtime verification result
7. exact Cloud Build command for the human
8. recommended new image tag, e.g. `authfix1`
9. exact human Cloud Run redeploy command
10. any remaining diagnostic uncertainty

---

## Completion report (2026-08-28, wishes-game session)

Root cause confirmed (not assumed) by building the pre-fix
`server/comfy-worker/Dockerfile` image and reproducing the exact production
failure inside it: `clients/comfy_broker_client.py`'s
`_default_id_token_provider()` uses `google.auth.transport.requests`, which
hard-imports the third-party `requests` package; `server/comfy-worker/
pyproject.toml` declared only `google-auth`, not `requests`. Token minting
happens before any HTTP request reaches the broker, matching every observed
symptom (`/health` 200, `/system-stats` 502, empty broker logs). A second,
independent bug was also found and fixed: the worker had no logging
configuration, so `extra={...}` fields were silently dropped, which is why
production logs showed only a bare `broker_call_failed`.

Fix: `google-auth[requests]>=2.30` (the package's own supported extra) in
`server/comfy-worker/pyproject.toml`; `_call_broker` in
`server/comfy-worker/src/comfy_worker/app.py` now logs
`operation=/error_type=/error=` embedded directly in the message text (not
`extra=`), passed through a new `_safe_exception_message()` redaction filter
that strips Authorization/Bearer/CF-Access-Client-Id/CF-Access-Client-Secret/
Cookie-shaped content before it ever reaches a log sink.

7 new regression tests added (`test_app.py`, new `test_auth_transport.py`).
Full suite: **116/116 passing**
(`python -m pytest server/comfy-broker/test/ server/comfy-worker/test/
clients/ --import-mode=importlib -q`). Verified via two real Docker builds
(pre-fix reproduces the bug; post-fix: `import requests` +
`google.auth.transport.requests` succeed, `/health` 200, `/system-stats` now
fails with `DefaultCredentialsError` — expected, since this sandbox has no
GCP credentials and never touched the real broker).

Human build: `docker build -f server/comfy-worker/Dockerfile -t <image-uri> .`
Recommended tag: `authfix1`. Redeploy: `IMAGE_TAG=authfix1
scripts/deploy_comfy_worker.sh`. Full report (all 10 items) in
`wishes-game`'s `docs/claude/todo.md`.

No deploy, no Secret Manager access, no Cloudflare credentials read, no IAM
change, no `terraform apply` — zero GCP credentials configured in this
session throughout.
