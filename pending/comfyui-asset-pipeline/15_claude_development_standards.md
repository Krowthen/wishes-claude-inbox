# 15 — Claude Development Standards

## Objective

This document defines the engineering standards Claude Code must follow while working within the Wishes repository.

Unlike the implementation guide, this document is persistent and should remain valid throughout the lifetime of the project.

Whenever Claude modifies the Wishes repository it should assume these standards unless explicitly instructed otherwise.

---

# Primary Goals

Every implementation should prioritize:

1. Correctness
2. Determinism
3. Maintainability
4. Extensibility
5. Performance
6. Readability
7. Documentation
8. Testability

Optimization should never come at the expense of maintainability.

---

# Core Engineering Principles

## Build for Expansion

Every implementation should assume the system will eventually become:

- distributed
- cloud hosted
- multi-region
- multi-GPU
- multi-service

Avoid solutions that cannot naturally scale.

---

## Modular Design

Every feature should have a single responsibility.

Avoid large classes.

Instead prefer:

```text
Controller

↓

Service

↓

Repository

↓

Storage

↓

Infrastructure
```

---

## Dependency Direction

Dependencies should always flow downward.

```text
Route

↓

Controller

↓

Service

↓

Storage

↓

Database
```

Never reverse this flow.

---

## Composition Over Inheritance

Favor:

- interfaces
- dependency injection
- composition

Avoid unnecessary inheritance.

---

## Deterministic Systems

Outside AI image generation:

Everything should produce identical output given identical input.

Examples:

- Card Rendering
- Metadata
- Prompt Building
- Storage Paths
- Review Logic

Randomness should never exist unless explicitly requested.

---

# Project Architecture

Preferred architecture:

```text
Presentation

↓

Application

↓

Domain

↓

Infrastructure
```

Business logic belongs in Domain/Application.

Infrastructure should never contain gameplay rules.

---

# API Standards

REST endpoints should:

- use nouns
- return consistent errors
- validate requests
- never expose internal implementation

Example:

```text
POST /api/assets/request

POST /api/assets/{uuid}/approve

POST /api/assets/{uuid}/publish
```

---

# Database Standards

The database is authoritative.

Never duplicate business rules solely in application code.

Use:

- foreign keys
- constraints
- indexes
- transactions
- SQL functions where appropriate

Never trust client validation.

---

# UUID Usage

All primary entities use UUID.

Never expose sequential IDs.

---

# JSON Standards

Prefer JSONB for flexible metadata.

Strongly type application models.

Database JSON should still validate against schemas.

---

# Configuration Standards

Never hardcode:

- paths
- URLs
- ports
- credentials
- workflow names
- storage roots

Everything configurable belongs in ConfigurationService.

---

# Logging Standards

Every significant action should log:

- timestamp
- actor
- object
- action
- duration
- outcome

Preferred:

Structured JSON logs.

---

# Error Handling

Never swallow exceptions.

Every error should include:

```text
Code

Message

Severity

Retryable

Context
```

Application errors should never leak internal stack traces to clients.

---

# Testing Standards

Every feature should include:

Unit Tests

Integration Tests

Validation Tests

Manual Smoke Tests

New features should not reduce coverage.

---

# Naming Standards

Folders:

```text
lowercase-kebab-case
```

Files:

```text
camelCase.ts
```

Interfaces:

```text
PascalCase
```

Enums:

```text
PascalCase
```

Database:

```text
snake_case
```

Environment Variables:

```text
UPPER_CASE
```

---

# Service Design

Services should:

- have one responsibility
- be stateless
- use dependency injection
- avoid global state

---

# Controller Design

Controllers should only:

- validate input
- call services
- format responses

Controllers should not contain business logic.

---

# Worker Design

Workers should:

- be restartable
- be idempotent
- be stateless
- recover automatically
- support concurrency

---

# Asset Pipeline Rules

Portrait is authoritative.

Every downstream asset references:

```text
Portrait

↓

Full Body

↓

Sprite

↓

Card

↓

Published Asset
```

Replacing portraits invalidates descendants.

---

# Prompt Standards

Prompt Builder is the only location that assembles prompts.

Never concatenate prompt strings throughout the codebase.

Prompt components:

```text
Global Style

Identity Block

Role Prompt

User Prompt

Negative Prompt

Revision Notes
```

---

# Workflow Standards

Every workflow requires:

Workflow JSON

↓

Manifest

↓

Injection Schema

↓

Version

↓

Validation

Workflow JSON is immutable during runtime.

---

# Storage Standards

Published assets:

Immutable.

Pending assets:

Mutable.

Rejected assets:

Preserved.

Archive assets:

Never overwritten.

---

# Card Rendering Standards

Cards are deterministic.

AI only generates artwork.

Renderer generates:

- layout
- icons
- text
- statistics
- effects

---

# Sprite Standards

Sprites derive from approved assets.

Metadata accompanies every sprite sheet.

Unity never guesses frame layout.

---

# Review Standards

Every asset supports:

Approve

Reject

Revise

Publish

Archive

Review events become permanent history.

---

# Documentation Standards

Every new subsystem requires:

Architecture

API

Configuration

Acceptance Criteria

Testing Notes

Future Work

---

# Performance Standards

Prefer:

Readable code

↓

Correct code

↓

Optimized code

Premature optimization should be avoided.

---

# Security Standards

Never expose:

ComfyUI

Model directories

Workflow directories

Secrets

Internal storage paths

Use least privilege.

---

# Git Standards

Each commit should:

- compile
- pass tests
- implement one logical feature

Avoid mixed-purpose commits.

---

# Claude-Specific Rules

Claude should:

- preserve repository style
- avoid unnecessary rewrites
- minimize breaking changes
- document assumptions
- leave TODOs only when explicitly approved

Claude should never:

- silently delete functionality
- remove validation
- bypass review systems
- weaken deterministic behavior

---

# Future Compatibility

Every implementation should assume:

- distributed rendering
- multiple AI providers
- cloud deployment
- versioned workflows
- multiple render pipelines

Avoid assumptions that prevent future expansion.

---

# Definition of Done

A feature is complete only when:

✓ Implementation complete

✓ Compiles successfully

✓ Tests pass

✓ Documentation updated

✓ Configuration documented

✓ Acceptance criteria satisfied

✓ No regression introduced

✓ Ready for review

---

# Engineering Philosophy

The Wishes codebase should remain understandable by a new developer years after initial implementation.

Every design decision should favor clarity, deterministic behavior, modular architecture, and long-term maintainability over short-term convenience.

The repository should be treated as a long-lived platform rather than a collection of features.