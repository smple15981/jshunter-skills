---
name: web-config-exposure-audit
description: Reviews public source maps, debug output, configuration, headers, CORS, cache behavior, and error disclosures for security-relevant exposure.
version: 0.1.0
---

# Web Configuration & Exposure Audit

## Goal

Identify security-relevant information exposure and web configuration mistakes on authorized public assets while separating harmless implementation details from reportable impact.

## Inputs

```yaml
urls: []
responses: []
js_analysis: []
source_maps: []
scope_allowlist: []
```

## Review areas

### Source maps and client artifacts

Check whether source maps or bundled code reveal:

- otherwise hidden API routes
- internal hostnames
- debug/test endpoints
- privileged feature names
- server-side assumptions copied into the client
- secret candidates

Source-map availability alone is not necessarily a vulnerability. Escalate only when the recovered information creates meaningful exposure or enables a demonstrable in-scope security issue.

### Error disclosure

Flag responses that expose:

- stack traces
- filesystem paths
- internal service names
- database/backend implementation details
- verbose framework exceptions
- internal request routing information

Treat version banners as informational unless they materially contribute to a validated issue.

### CORS

Review combinations of:

- `Access-Control-Allow-Origin`
- `Access-Control-Allow-Credentials`
- reflected origins
- `null` origin behavior
- sensitive response content

A wildcard origin on intentionally public, unauthenticated content is usually not a security finding by itself.

### Cache behavior

Review:

- `Cache-Control`
- `Vary`
- CDN/cache status headers
- redirects
- whether security-sensitive responses appear shared-cacheable

Do not attempt cache poisoning techniques that could affect other users. Restrict analysis to passive headers and harmless self-contained comparisons unless the program explicitly permits deeper testing.

### Public configuration files

Prioritize configuration that reveals live sensitive endpoints, credentials with demonstrable in-scope impact, private infrastructure details tied to another issue, or security controls disabled in production.

Do not use discovered credentials to pivot to unrelated or out-of-scope systems.

## Candidate output

```yaml
findings:
  - id: WCEA-001
    category: source-map-exposure
    asset: https://example.com/app.js.map
    state: candidate
    signals: []
    impact: unverified
    confidence: 0.0
    next_safe_action: Determine whether recovered routes reveal live protected functionality
```

## False-positive checklist

Before escalating ask:

- Is this data already intentionally public?
- Is the value a browser/client identifier rather than a secret?
- Does the disclosure cross a security boundary?
- Is there a concrete impact beyond technology fingerprinting?
- Can the impact be shown safely without abusing credentials or affecting users?

Prefer one well-supported finding over dozens of low-value header observations.