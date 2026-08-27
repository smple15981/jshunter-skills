---
name: response-differential-analysis
description: Compares HTTP responses across carefully controlled request variants to surface authentication, authorization, schema, cache, and error-handling differences.
version: 0.1.0
---

# Response Differential Analysis

## Goal

Detect security-significant behavioral differences between a small set of controlled HTTP requests. Focus on semantic differences rather than raw byte diffs.

## Inputs

```yaml
request_variants: []
responses: []
endpoint_context: []
scope_allowlist: []
```

## Compare

Normalize and compare:

- status code
- redirect target
- response content type
- body length as a weak signal only
- JSON keys and schema
- count of returned objects
- sensitive-field presence
- error code/message category
- cache headers and cache status
- CORS headers
- authentication challenge headers
- timing only as a low-confidence supporting signal

Ignore or downweight noisy values such as timestamps, request IDs, CSRF nonces, analytics IDs, random ordering, and dynamic cache ages.

## Useful comparison sets

Examples of low-impact comparisons:

```text
no-auth vs normal public request
current API version vs legacy equivalent
parameter omitted vs harmless parameter value
Origin absent vs controlled alternate Origin
GET vs HEAD when supported
```

Avoid high-volume mutation fuzzing.

## Semantic diff model

Represent differences as structured events:

```yaml
- type: field-added
  field: email
  security_relevance: high
- type: auth-state-change
  from: 401
  to: 200
  security_relevance: high
- type: error-detail-change
  detail: stack-trace-present
  security_relevance: medium
```

## Risk heuristics

Escalate when a tiny request change causes:

- authentication requirement to disappear
- privileged or sensitive fields to appear
- a bulk data response where a protected response was expected
- an internal error/stack trace to become visible
- cache behavior to cross user/security boundaries
- a legacy API to expose more data than a current API

## Output

```yaml
comparison:
  endpoint: /api/example
  variants: [A, B]
  normalized_differences: []
  security_signals: []
  noise_removed: []
  confidence: 0.0
  conclusion: no-significant-difference
  next_safe_action: null
```

## Quality bar

Never treat content-length differences alone as a vulnerability. Explain the semantic change and why it matters to a security boundary.