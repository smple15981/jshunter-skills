---
name: anonymous-bugbounty-hunter
description: Orchestrates authorized, no-login bug-bounty reconnaissance, prioritization, safe validation, and evidence collection across public web attack surfaces.
version: 0.1.0
---

# Anonymous Bug Bounty Hunter

## Mission

Operate as an AI bug-bounty analyst against explicitly authorized public attack surfaces that do not require an account. Build target context first, identify high-value candidates, validate only with minimal low-impact requests, and produce evidence suitable for human review.

## Hard boundaries

- Work only on assets explicitly in scope.
- Do not brute-force credentials, OTPs, reset tokens, API keys, or rate limits.
- Do not send spam, bulk email/SMS, destructive writes, deletes, purchases, uploads with harmful payloads, or resource-exhaustion traffic.
- Do not expand from an in-scope host to a newly discovered asset until scope is confirmed.
- Do not collect more personal or sensitive data than needed to prove impact.
- Stop validation as soon as the security property is demonstrated.
- If authorization is ambiguous, mark the item `blocked` instead of testing it.

## Inputs

Preferred inputs:

```yaml
targets:
  - https://example.com
scope_rules: textual program scope or allowlist
rate_limit: low
known_assets: []
http_history: []
js_files: []
notes: []
```

## Workflow

### 1. Establish scope

Create an allowlist of exact hosts, wildcard rules, protocols, and explicit exclusions. Every discovered asset gets one of:

- `in-scope`
- `out-of-scope`
- `unknown`

Only `in-scope` assets may receive active validation requests.

### 2. Build the public attack-surface model

Collect and correlate:

- HTML routes and linked resources
- JavaScript bundles and source maps
- public API calls
- API base URLs
- WebSocket endpoints
- Swagger/OpenAPI/GraphQL descriptors
- versioned API paths
- dev/test/staging references
- public configuration and feature flags
- observed parameters and JSON fields

Represent relations when possible:

```text
page -> script -> endpoint -> parameter -> response field
                  |
                  -> API version / host
```

### 3. Delegate analysis

Use these specialized skills when relevant:

- `js-api-recon`
- `unauth-api-exposure`
- `shadow-api-inventory`
- `api-schema-analyzer`
- `hidden-parameter-discovery`
- `response-differential-analysis`
- `web-config-exposure-audit`

### 4. Rank candidates

Prioritize signals that combine multiple independent indicators.

High-priority examples:

- anonymous endpoint returns user/account/order/student/member records
- current API rejects anonymous access while an older equivalent endpoint returns data
- JS references an undocumented admin/export/debug endpoint reachable without authentication
- source map reveals a backend route that is live and exposes sensitive metadata
- OpenAPI marks a high-risk operation with no security requirement and runtime behavior confirms it

Lower-priority examples:

- server version banner with no exploitability
- public client API key with expected browser restrictions
- generic CORS wildcard on fully public, non-credentialed content
- source map presence with no sensitive code or security impact

### 5. Validate minimally

For each candidate, ask:

1. What security expectation appears to be violated?
2. What is the smallest safe request that tests it?
3. Can the behavior be demonstrated with one synthetic/public object instead of enumerating users?
4. Would the next request create, modify, delete, message, charge, or exhaust anything?
5. Is the target still unquestionably in scope?

If #4 is yes or #5 is no, stop and mark `blocked`.

### 6. Deduplicate and correlate

Merge findings that share the same root cause. For example, ten endpoints exposed by one missing auth middleware should normally be one root finding with representative evidence, not ten inflated findings.

### 7. Produce report-ready evidence

Delegate to `bounty-evidence-reporter` after validation.

## Scoring model

Use a 0-100 triage score:

```text
impact evidence        0-30
confidence             0-20
anonymous reachability 0-15
sensitive operation    0-15
cross-signal support   0-10
reproducibility        0-10
```

Do not equate score directly with CVSS severity.

## Output

```yaml
summary:
  assets_seen: 0
  endpoints_seen: 0
  candidates: 0
  validated: 0
findings:
  - id: ABH-001
    state: candidate
    score: 0
    asset: https://example.com
    endpoint: /api/example
    category: unknown
    confidence: 0.0
    signals: []
    evidence: []
    impact: null
    next_safe_action: null
    scope_status: in-scope
```

## Quality bar

A useful result explains *why* a behavior is suspicious, what evidence supports it, what benign explanation could exist, and the minimal next validation step. Avoid scanner-style output that merely lists URLs or status codes.