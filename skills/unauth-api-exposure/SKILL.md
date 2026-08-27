---
name: unauth-api-exposure
description: Identifies public APIs that appear to expose sensitive data or privileged functionality without expected authentication, using minimal safe validation.
version: 0.1.0
---

# Unauthenticated API Exposure

## Goal

Find endpoints where anonymous users receive data or capabilities that the application appears to intend for authenticated or privileged users.

This skill focuses on missing or inconsistent authentication at the endpoint level. Do not call every public endpoint a vulnerability; distinguish intentionally public content from protected business data.

## Inputs

```yaml
endpoints: []
http_history: []
application_context: []
scope_allowlist: []
```

## Anonymous request profiles

When active validation is permitted, prefer a very small comparison set:

```text
A: normal request with no Cookie and no Authorization header
B: same request with authentication headers removed from an observed public request template
C: OPTIONS/HEAD only when useful and non-disruptive
```

Do not brute force bearer tokens, session IDs, API keys, user IDs, or object identifiers.

## High-value signals

Prioritize responses containing fields or operations related to:

- personal profiles
- email/phone/address
- student/member/employee records
- orders/invoices/payment metadata
- private documents/downloads
- account configuration
- admin or moderation data
- internal identifiers combined with private attributes
- bulk exports or search results

## Public-vs-private reasoning

For every candidate, explicitly consider benign explanations:

```yaml
public_by_design_signals:
  - linked directly from public UI
  - documented as public API
  - contains only intentionally public profile fields
  - equivalent data appears in public page HTML

private_by_design_signals:
  - privileged UI reference
  - route name suggests account/admin/internal context
  - sibling/current endpoint requires authentication
  - response includes non-public account or operational fields
  - API schema declares security while runtime does not enforce it
```

## Safe validation strategy

1. Reproduce the endpoint anonymously once.
2. Confirm the sensitive field or privileged behavior with the minimum number of records.
3. Prefer a public/demo/test object if available.
4. Avoid pagination through datasets simply to determine scale.
5. Never modify another user's data.
6. Stop once unauthorized access is clearly demonstrated.

## Candidate categories

```text
unauthenticated-data-exposure
missing-endpoint-authentication
anonymous-export-or-download
anonymous-admin-metadata
inconsistent-authentication-between-api-versions
```

## Evidence quality

Record:

- exact URL and method
- relevant request headers with secrets removed
- whether Cookie/Authorization were absent
- status code
- minimal redacted response excerpt
- why the fields should not be public
- a comparison endpoint or UI behavior when available

## Output

```yaml
findings:
  - id: UAE-001
    state: candidate
    endpoint: /api/example
    method: GET
    anonymous_status: 200
    category: unauthenticated-data-exposure
    sensitive_fields: [email, phone]
    public_by_design_probability: low
    confidence: 0.90
    evidence_minimized: true
    next_safe_action: Confirm the same endpoint is intended for authenticated account context
```

## Stop conditions

Mark `blocked` and do not continue if proving impact would require:

- enumerating large numbers of users
- guessing identifiers at scale
- downloading full private datasets
- creating/modifying/deleting records
- triggering email/SMS/payment flows
- testing outside scope
