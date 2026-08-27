# jshunter-skills

Modular AI skills for authorized, anonymous bug-bounty reconnaissance and validation.

> Use these skills only on assets you own or targets that explicitly authorize security testing. Default behavior is low-impact, low-concurrency, evidence-first testing. Destructive actions, brute force, resource exhaustion, spam, persistence, credential attacks, and out-of-scope pivoting are excluded.

## Skill map

| Skill | Purpose |
|---|---|
| `anonymous-bugbounty-hunter` | Orchestrates the full anonymous workflow and prioritizes findings |
| `js-api-recon` | Extracts endpoints, parameters, assets, routes, feature flags, and API clues from HTML/JS/source maps |
| `unauth-api-exposure` | Finds APIs that expose data or functionality without expected authentication |
| `shadow-api-inventory` | Correlates current, legacy, dev, test, staging, and versioned APIs |
| `api-schema-analyzer` | Analyzes Swagger/OpenAPI/GraphQL schemas and prioritizes anonymous high-risk operations |
| `hidden-parameter-discovery` | Correlates code and traffic to identify undocumented or unused parameters |
| `response-differential-analysis` | Compares HTTP responses and highlights security-significant differences |
| `web-config-exposure-audit` | Reviews source maps, debug output, headers, CORS, cache behavior, and exposed configuration |
| `bounty-evidence-reporter` | Turns validated findings into reproducible, minimally invasive bounty reports |

## Recommended orchestration

```text
Authorized target
   ↓
Scope guard
   ↓
Asset / URL / JS collection
   ↓
JS & API recon ───────────────┐
   ↓                          │
API/schema inventory          │
   ↓                          │
Anonymous-access checks       │
   ↓                          │
Response differential analysis│
   ↓                          │
Config/exposure review        │
   ↓                          │
Risk ranking & deduplication ←┘
   ↓
Human-safe validation
   ↓
Evidence & report
```

## Core operating rules

1. Verify the target is in scope before any active request.
2. Prefer passive collection before active validation.
3. Keep request volume low and deterministic.
4. Never brute-force credentials, OTPs, reset flows, or rate limits.
5. Never perform destructive writes, delete data, send bulk messages, or cause resource exhaustion.
6. Treat secrets found in client-side code as candidates until impact is verified without abusing them.
7. Stop when a minimal proof is sufficient; do not enumerate unnecessary user records or sensitive data.
8. Record every request needed to reproduce a finding.
9. Separate `candidate`, `validated`, and `reportable` states.
10. Let a human make the final submission decision.

## Suggested finding states

```text
candidate   -> suspicious signal only
validated   -> behavior reproduced with minimal safe evidence
reportable  -> clear security impact + reproducible steps + in-scope target
rejected    -> expected/public behavior, duplicate signal, or no security impact
blocked     -> validation would exceed authorization or safety limits
```

## Output contract

Skills should prefer machine-readable findings with these common fields:

```yaml
id: finding-001
state: candidate
asset: https://example.com
endpoint: /api/example
method: GET
category: unauthenticated-data-exposure
confidence: 0.86
severity_hint: medium
summary: Short description
signals:
  - Why this looks suspicious
evidence:
  requests: []
  responses: []
impact: Pending validation
next_safe_action: Minimal next step
scope_status: in-scope
```

## Composition

Start with `skills/anonymous-bugbounty-hunter/SKILL.md`. The orchestrator delegates specialized analysis to the other skills and merges their findings into one ranked queue.
