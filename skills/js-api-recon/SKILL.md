---
name: js-api-recon
description: Extracts and correlates routes, API endpoints, parameters, service hosts, feature flags, source maps, and security-relevant clues from public HTML and JavaScript.
version: 0.1.0
---

# JS API Recon

## Goal

Turn public front-end code into a structured map of the application's reachable and referenced attack surface. This is primarily a static-analysis skill; active requests should be limited to fetching already public resources and safely confirming in-scope endpoints.

## Inputs

```yaml
base_urls: []
html_documents: []
js_documents: []
source_maps: []
http_history: []
scope_allowlist: []
```

## Extract

Look for:

- absolute and relative HTTP(S) endpoints
- API base URLs and environment-specific hosts
- REST paths and version markers such as `/api/v1/`
- GraphQL and WebSocket endpoints
- Swagger/OpenAPI locations
- route templates and path parameters
- request methods when inferable
- query/body/header parameter names
- JSON response field names
- upload/download/export/import paths
- admin/internal/debug/test/dev/staging references
- feature flags and role/permission names
- source map references
- storage keys and client-side auth state names
- third-party service endpoints

## Do not overclaim secrets

Strings that resemble API keys, tokens, IDs, or credentials are only `secret-candidates`. Public browser SDK identifiers are often intentionally exposed.

For every candidate record:

```yaml
value_fingerprint: redacted hash or partial representation
source_file: app.js
source_location: module/function if known
provider_hint: null
likely_client_side: true
impact_status: unverified
```

Never use discovered credentials against unrelated services or out-of-scope assets.

## Correlation workflow

### A. Normalize endpoints

Canonicalize equivalent forms:

```text
/api/users/123
/api/users/{id}
https://api.example.com/api/users/{id}
```

Keep the raw source alongside the normalized route.

### B. Attach provenance

Each finding should state where it came from:

```text
index.html -> app.abc123.js -> module UserService -> GET /api/users/{id}
```

### C. Infer endpoint purpose

Use function names, surrounding strings, UI labels, and request/response shapes to infer categories such as:

- profile
- user/member/student
- admin
- search
- export/download
- order/payment
- config/debug
- upload/import

Mark inference confidence separately from observed facts.

### D. Compare observed vs referenced

Classify endpoints as:

- `observed-in-traffic`
- `referenced-in-code`
- `both`
- `historical/source-map-only`

Endpoints referenced in code but never observed in normal browsing are strong candidates for deeper review.

### E. Detect environment clues

Flag references containing patterns like:

```text
dev
stage
staging
test
qa
uat
old
legacy
internal
beta
v1/v2/v3
```

Do not actively probe a newly discovered hostname until scope is confirmed.

## Priority heuristics

Increase priority when an endpoint is:

- undocumented but clearly tied to sensitive functionality
- referenced by privileged UI code
- an older version of a current protected endpoint
- an export/download endpoint
- a bulk lookup/search endpoint
- related to config, debug, internal tooling, or administration
- reachable anonymously based on existing traffic evidence

## Output

```yaml
endpoints:
  - normalized: /api/users/{id}
    methods: [GET]
    hosts: [api.example.com]
    sources:
      - app.js:UserService
    parameters:
      path: [id]
      query: []
      body: []
    tags: [user]
    observed_in_traffic: false
    confidence: 0.94
    scope_status: in-scope
    next_safe_action: Compare anonymous response with documented public behavior

service_hosts: []
source_maps: []
secret_candidates: []
feature_flags: []
```

## Success criterion

The output should be a navigable application/API map, not a dump of regex matches. Deduplicate aggressively and preserve provenance.