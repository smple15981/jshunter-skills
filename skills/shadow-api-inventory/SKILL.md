---
name: shadow-api-inventory
description: Correlates current, legacy, dev, test, staging, and versioned API references to identify forgotten or inconsistently protected public attack surface.
version: 0.1.0
---

# Shadow API Inventory

## Goal

Build a version-aware inventory of API hosts and paths, then identify older, forgotten, or environment-specific interfaces that may have weaker controls than the current production API.

## Inputs

```yaml
hosts: []
endpoints: []
js_references: []
http_history: []
scope_allowlist: []
```

## Discovery signals

Look for names and paths containing:

```text
v1 v2 v3
old legacy deprecated
beta alpha
api-old old-api
api-dev dev-api
api-test test-api
stage staging uat qa
internal debug
```

Sources may include JavaScript, source maps, HTML, public API documentation, response links, and existing reconnaissance data.

## Scope rule

A discovered hostname is not automatically authorized. Assign each asset:

```text
in-scope
out-of-scope
unknown
```

Only active-check `in-scope` assets.

## Correlation model

Group endpoints that appear semantically equivalent:

```yaml
operation: get-user-profile
variants:
  - host: api.example.com
    path: /v3/profile
  - host: api.example.com
    path: /v2/profile
  - host: old-api.example.com
    path: /profile
```

Compare controls, not just status codes:

- authentication requirement
- response schema
- sensitive fields
- CORS behavior
- cache behavior
- rate-limit headers
- error verbosity
- method support

## High-value patterns

Increase priority when:

- newest endpoint returns 401/403 but older equivalent returns 200 anonymously
- an old API returns extra sensitive fields removed from the current version
- dev/test host references production data
- a deprecated API is still publicly reachable
- older docs expose operations absent from current docs
- one version omits an auth/security declaration present in another

## Safe comparison

Use the minimum number of low-impact requests necessary to establish a difference. Do not crawl an old API exhaustively or enumerate data sets.

## Output

```yaml
api_families:
  - operation: example
    versions:
      - host: api.example.com
        path: /v3/example
        auth_behavior: required
      - host: api.example.com
        path: /v1/example
        auth_behavior: unknown
    suspicious_differences: []
    confidence: 0.0
    next_safe_action: null
```

## Common false positives

- versioned static content APIs intended for public use
- separate public/mobile APIs with intentionally different schemas
- development hostnames that resolve but are out of scope
- version labels that are resource versions rather than API generations

Always preserve the evidence explaining why two endpoints were grouped together.