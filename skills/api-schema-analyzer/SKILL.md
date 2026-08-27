---
name: api-schema-analyzer
description: Parses Swagger, OpenAPI, and GraphQL descriptions to inventory operations, infer authentication expectations, and prioritize high-risk anonymous attack surface.
version: 0.1.0
---

# API Schema Analyzer

## Goal

Convert API schemas into an actionable, security-focused operation inventory. Prefer schema reasoning before sending active requests.

## Inputs

```yaml
openapi_documents: []
swagger_documents: []
graphql_schema: null
observed_http: []
scope_allowlist: []
```

## OpenAPI / Swagger analysis

For every operation extract:

- server/host and base path
- path and HTTP method
- operation ID and tags
- parameters and request body schema
- response schemas
- declared security requirements
- deprecated flag
- descriptions mentioning admin/internal/export/import/debug

Classify auth expectation:

```text
required
explicitly-public
inherited
ambiguous
```

Do not assume `security: []` is a vulnerability; it may intentionally mark a public operation.

## GraphQL analysis

Inventory query and mutation fields, arguments, return types, and naming signals. Prioritize fields suggesting:

- account/private profile data
- bulk lookup/search
- export/download
- admin/moderation
- internal/debug operations
- mutation of sensitive objects

Do not perform broad introspection or high-volume field probing if the program forbids it. Prefer an already exposed schema or observed client queries.

## Risk ranking

High-priority combinations include:

```text
sensitive operation + no declared auth
sensitive operation + anonymous runtime success
bulk/export operation + anonymous reachability
deprecated operation + weaker auth than current equivalent
schema says auth required + runtime ignores it
```

## Runtime validation

When authorized, validate only the highest-value candidates with minimal non-destructive requests. For mutations, do not execute state-changing operations merely to see whether they are protected; report the risky schema/runtime evidence and mark further validation `blocked` unless a harmless dry-run is explicitly supported.

## Output

```yaml
operations:
  - id: getExample
    method: GET
    path: /api/example
    tags: []
    auth_expectation: required
    sensitive: false
    deprecated: false
    anonymous_runtime: unknown
    risk_score: 0
    reasons: []
    next_safe_action: null
```

## False-positive controls

Downgrade findings when the operation is clearly public documentation/content, health metadata with no sensitive details, or browser-facing public search whose returned fields match the public UI.