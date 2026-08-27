---
name: hidden-parameter-discovery
description: Correlates front-end code, API schemas, and observed traffic to identify undocumented or normally unused request parameters worth safe security review.
version: 0.1.0
---

# Hidden Parameter Discovery

## Goal

Find parameters that exist in code or schemas but are absent from normal requests, then rank them by security relevance. This skill discovers candidates; it does not blindly fuzz every parameter name.

## Inputs

```yaml
endpoints: []
js_analysis: []
api_schemas: []
http_history: []
scope_allowlist: []
```

## Candidate sources

Extract parameter names from:

- JavaScript object literals
- request builders and API clients
- TypeScript interfaces/types
- OpenAPI request schemas
- GraphQL arguments
- form definitions
- feature-flag/config objects
- error messages that name accepted fields
- historical/source-map code

## Correlation classes

For each endpoint classify a parameter as:

```text
observed            present in normal traffic
schema-documented   declared in API schema
code-referenced     referenced by client code
hidden-candidate    referenced but not normally sent
historical-only     appears only in old/source-map code
```

## Security-priority names

Raise priority for names related to authorization, object selection, visibility, tenancy, debug behavior, or response expansion, for example conceptual categories such as:

```text
role / privilege
user or object identifier
tenant / organization
admin or internal mode
debug / verbose
include hidden/deleted fields
field selection / expansion
export / format
```

A suspicious name is not evidence of a vulnerability.

## Safe validation

When testing a candidate parameter:

1. Start with a harmless value.
2. Send at most a small number of deterministic comparisons.
3. Observe status, response schema, field set, and error behavior.
4. Do not use values that modify authorization state, create users, perform payments, delete data, or send messages.
5. Stop if behavior implies a state-changing action.

Prefer testing whether a parameter changes *read-only response behavior* over attempting privileged writes.

## Ranking

Increase confidence when multiple sources agree:

```text
JS reference + schema field
JS reference + server error mentioning field
historical client code + live endpoint response change
privileged component + endpoint parameter
```

## Output

```yaml
candidates:
  - endpoint: /api/example
    parameter: includeInternal
    locations: [query]
    sources: [app.js]
    observed_normally: false
    security_category: response-expansion
    confidence: 0.78
    validation_state: untested
    next_safe_action: Compare read-only response with false vs omitted
```

## False-positive controls

Common benign cases include unused SDK options, dead code, framework internals, analytics parameters, and fields accepted but ignored server-side. Require runtime evidence before escalating.