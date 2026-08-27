---
name: bounty-evidence-reporter
description: Converts validated, authorized findings into concise, reproducible bug-bounty reports with minimized evidence and explicit impact reasoning.
version: 0.1.0
---

# Bounty Evidence Reporter

## Goal

Turn a validated finding into a report that a triager can reproduce quickly without exposing unnecessary sensitive data or exaggerating severity.

## Inputs

```yaml
finding: {}
requests: []
responses: []
scope_rules: []
validation_notes: []
```

## Preconditions

Only create a final report when:

- target is confirmed in scope
- behavior has been reproduced
- evidence is sufficient to show the security boundary failure
- sensitive data is minimized/redacted
- no additional risky exploitation is required to explain impact

If any of these are missing, output a validation checklist instead of pretending the issue is confirmed.

## Report structure

### Title

Describe root cause + affected surface + impact.

Good pattern:

```text
Unauthenticated access to [resource/function] exposes [impact]
```

Avoid sensational titles such as `Critical RCE` unless the evidence actually supports that severity.

### Summary

In 2-4 sentences explain:

1. what endpoint/component is affected
2. what security expectation fails
3. what an anonymous attacker can obtain or do
4. why that matters

### Affected asset

Include exact in-scope host, path, and method.

### Reproduction

Use the shortest deterministic steps possible. Preserve raw HTTP only when it improves reproducibility.

Never include live session tokens, credentials, full personal records, or unnecessary private data. Replace them with clear redactions.

### Evidence

Prefer:

- one representative request
- one representative response excerpt
- a protected/current comparison when useful
- headers that prove no authentication was sent
- schema/JS provenance supporting intended behavior

### Impact

State observed impact separately from hypothetical escalation.

```yaml
observed_impact: What was actually demonstrated
potential_impact: Plausible extension, clearly marked as unverified
```

Do not estimate the size of an exposed dataset by enumerating it. If pagination/count metadata safely exposes a total, cite that metadata; otherwise say the scale was not tested.

### Remediation

Recommend root-cause fixes such as:

- enforce authentication/authorization server-side
- remove/decommission legacy endpoint
- align auth middleware across API versions
- restrict sensitive response fields
- disable production debug output
- correct CORS/cache policy for sensitive responses

Avoid over-prescriptive framework-specific advice unless the technology is confirmed.

## Severity reasoning

Provide a `severity_hint`, not an absolute platform rating, unless the bounty program defines a scoring system.

Consider:

- authentication required or anonymous
- data sensitivity
- number/types of affected objects inferred safely
- read vs write capability
- user interaction required
- scope of impact
- reproducibility

## Output template

```yaml
report:
  title: ""
  severity_hint: medium
  confidence: 0.95
  affected_asset: ""
  method: GET
  summary: ""
  prerequisites: none
  reproduction_steps: []
  evidence:
    request: ""
    response_excerpt: ""
    redactions: []
  observed_impact: ""
  potential_impact: ""
  remediation: []
  scope_confirmation: in-scope
  validation_limitations: []
```

## Final quality checklist

- Can a triager reproduce this in under a few minutes?
- Is the exact security boundary failure obvious?
- Is all unnecessary personal/sensitive data removed?
- Are claims limited to what the evidence supports?
- Is the root cause separated from secondary symptoms?
- Did testing stop after minimal proof?

If the answer to any item is no, revise before submission.