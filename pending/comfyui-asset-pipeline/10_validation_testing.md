# 10 — Validation, Testing & Quality Assurance Specification

## Objective

Define the complete validation, testing, and quality assurance strategy for the Wishes AI Asset Pipeline.

The goal of this document is to ensure every generated asset is technically valid, visually consistent, reproducible, and safe for publication before it reaches the game.

Testing must validate:

- Database integrity
- API behavior
- Queue processing
- Workflow execution
- Prompt construction
- ComfyUI integration
- Asset storage
- Review workflow
- Publication
- Unity compatibility

## Testing Philosophy

Testing is divided into five layers:

```text
Unit Tests
    ↓
Integration Tests
    ↓
Workflow Tests
    ↓
System Tests
    ↓
Manual QA
```

No single layer replaces another.

## Quality Gates

Every asset progresses through quality gates.

```text
Queued
    ↓
Generated
    ↓
Technical Validation
    ↓
Review Pending
    ↓
Human Approval
    ↓
Publication
```

Technical validation must succeed before an asset becomes reviewable.

## Unit Testing

Every service should have automated unit tests.

Required coverage:

```text
PromptBuilderService
WorkflowRegistryService
StorageService
AssetDependencyService
AssetReviewService
AssetPublishService
ComfyDispatcher (mocked)
Path Builder
Metadata Builder
Validation Rules
```

Target coverage:

```text
80% minimum
90% preferred
```

## API Testing

Validate every endpoint.

Required checks:

- Success responses
- Invalid payloads
- Missing required fields
- Authentication failures
- Invalid UUIDs
- Missing assets
- Stale assets
- Duplicate approval attempts

Example matrix:

| Endpoint | Happy Path | Invalid Input | Auth | Not Found |
|----------|------------|---------------|------|-----------|
| POST /requests | ✓ | ✓ | ✓ | ✓ |
| POST /approve | ✓ | ✓ | ✓ | ✓ |
| POST /publish | ✓ | ✓ | ✓ | ✓ |

## Database Validation

Verify:

- Foreign keys
- Check constraints
- Trigger execution
- Audit records
- Queue locking
- Cascade behavior
- Versioning
- Dependency graph integrity

Regression tests should recreate the schema from scratch on every CI run.

## Workflow Validation

Before deployment every workflow must be verified.

Checklist:

```text
Workflow JSON loads
Manifest exists
Injection schema valid
Required nodes present
Output node exists
Workflow executes
Output image produced
```

Reject workflows with missing injection points.

## Prompt Validation

Every generated prompt should be validated before submission.

Confirm:

- Positive prompt exists
- Negative prompt exists
- Identity block exists
- Role instruction exists
- Width/height valid
- Seed recorded

## ComfyUI Integration Testing

Tests:

- Connection available
- Workflow submission
- Prompt ID returned
- History retrieval
- Output download
- Timeout handling
- Error propagation

Mock ComfyUI for automated CI where GPU is unavailable.

## Asset Validation

Every generated asset should pass deterministic checks.

Images:

```text
Exists
Readable
Correct dimensions
Correct format
Expected file size
Transparent background where required
```

Metadata:

```text
JSON valid
Required fields present
Source references valid
Workflow version recorded
```

## Review Workflow Testing

Verify:

- Approve
- Reject
- Revise
- Publish
- Replace portrait
- Descendant invalidation
- Regeneration queue

## Sprite Validation

Verify:

- Tile size
- Frame count
- Metadata consistency
- Animation playback
- Transparency
- Alignment

## Card Validation

Verify:

- Text readable
- Icons correct
- Stats correct
- No clipping
- Safe margins
- Artwork source approved

## Performance Targets

Portrait generation:

```text
Queue latency < 5 sec
```

Rendering:

```text
Card composition < 250 ms
```

API:

```text
Health endpoint < 100 ms
```

Worker:

```text
Idle polling < configured interval
```

## Logging Validation

Confirm logs include:

- Job lifecycle
- Workflow lifecycle
- Review actions
- Publication
- Errors
- Performance timing

Structured JSON logs preferred.

## Manual QA Checklist

Portrait:

- Identity preserved
- Style correct
- No anatomy issues

Full Body:

- Outfit complete
- Feet visible
- Hands visible

Card:

- Text readable
- Layout correct

Sprite:

- Animation readable
- Metadata correct

## Regression Testing

Run after:

- Workflow changes
- Template changes
- Database migrations
- Prompt updates
- ComfyUI upgrades
- Model upgrades

Regression suite should recreate canonical test assets and compare outputs where deterministic.

## Failure Classification

Categories:

```text
Configuration
Database
Workflow
Generation
Storage
Review
Publication
Runtime
```

Every failure should include:

```json
{
  "code":"WORKFLOW_NODE_MISSING",
  "severity":"error",
  "retryable":true
}
```

## CI Recommendations

Run automatically:

1. Lint
2. TypeScript compile
3. Unit tests
4. Integration tests
5. Database migration test
6. Asset service startup
7. Mock workflow execution

GPU-dependent tests may run in a nightly pipeline.

## Acceptance Criteria

1. Unit coverage meets target.
2. All APIs validated.
3. Workflow validation passes.
4. Database constraints verified.
5. Review workflow fully tested.
6. Sprite metadata validated.
7. Card composition validated.
8. Manual QA checklist completed.
9. Regression suite passes.
10. Production deployment blocked on failed validation.
