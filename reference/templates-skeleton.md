# Template skeletons (fallback)

Copy these into the new project's `00-meta/templates/` when the live company-knowledge
templates at `~/Documents/unpublic/src/company-knowledge/00-meta/templates/` are not available.
Translate the prose into the chosen output language; keep frontmatter **keys** in English.

The shared frontmatter block (every doc carries it):

```yaml
title: <…>
id: <see id patterns in SKILL.md>
type: product | functional-doc | technical-spec | api-spec | decision | feature-discovery
status: draft
owner: <email>
reviewers: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
review_by: <YYYY-MM-DD>   # created + 6 months
audience: []              # all|engineering|product|design|sales|support|devops|data|legal|leadership
sensitivity: internal     # public|internal|confidential|restricted
tags: []                  # kebab-case, max 6
related: []               # ["[[other-doc-id]]"]
ai_usage: allowed         # allowed|restricted|forbidden
```

---

## product.md / architecture.md  (`type: product`)

Sections: `## What it is` (2–3 sentences) · `## Status` (maturity, version, stack) ·
`## Team` · `## Architecture` (high-level + diagram) · `## Main components` ·
`## Linked documentation` (wikilinks) · `## Contacts`.

---

## functional-doc.md  (`type: functional-doc`) — USER POV, no code/endpoints/DB

Extra frontmatter: `product:`, `user_roles: [ … ]`.

Sections: `## What it is` (for the user) · `## When you use it` · `## Who it is for` (per role) ·
`## How it works — the user flow` (numbered steps, inline screenshots `assets/<slug>-NN-*.png` w/ alt+caption) ·
`## What you see when everything goes right` · `## What can go wrong (and what to do)` (user language) ·
`## Limits and rules worth knowing` · `## Privacy and security for the user` (plain language) ·
`## FAQ` · `## Related features` (wikilinks) · `## History`.

---

## technical-spec.md  (`type: technical-spec`) — ENGINEERING POV

Extra frontmatter: `product:`, `component:`, `code_references: []` (paths, filled once code exists).

Sections: `## Summary` (what + why, for a new colleague) ·
`## Expected behaviour` (step-by-step) ·
`## Business rules` (numbered, each a verifiable "the system must / when X then Y", cite `file.ts:line`) ·
`## Edge cases` (table: Case | Expected behaviour | Where handled) ·
`## States and transitions` (only if stateful: text diagram + table) ·
`## Implementation` (routes / service / repo+tables / tests / events / external deps) ·
`## References` (wikilinks to PRD, ADRs, api-spec) · `## History`.

---

## api-spec.md  (`type: api-spec`)

Extra frontmatter: `product:`, `api_version:`, `endpoint_count:`, `code_references: []`.

Sections: `## Summary` · `## Authentication required` (method, scope/roles, public endpoints) ·
`## Endpoints` (per endpoint: `### METHOD /path`, params table, request body + Zod source,
200 response, curl example, error-codes table 400/401/403/404/409/429/500) ·
`## Rate limiting` · `## Versioning and deprecation policy` · `## References` · `## History`.

---

## decision.md  (`type: decision`) — ADR, id `adr-NNNN`

Sections: `## Status` (proposed|accepted|superseded by [[adr-XXXX]]|deprecated) ·
`## Context` (the situation forcing a decision; explicit enough for someone absent) ·
`## Decision` (1–2 crisp sentences) ·
`## Alternatives considered` (Option A/B/C with Pros / Cons / why rejected-or-chosen) ·
`## Consequences` (easier / harder / needs future attention) ·
`## Review` (when to revisit) · `## References`.

---

## feature-discovery.md  (`type: feature-discovery`) — PRD light, id `feat-<area>-<slug>`

Extra frontmatter: `functional_spec_target:` (paths of the functional-doc + technical-spec to create at release).

Sections: `## Discovery status` · `## Context` (origin, why now) · `## Users involved` (primary/secondary/exclusions, tiers) ·
`## Problem` (concrete scenario) · `## Proposed solution` (user POV + main flow + edge cases) ·
`## Alternatives considered` · `## Success criteria` (quantitative + qualitative) · `## Prototype` ·
`## Dependencies` (technical/product/org) · `## Risks` · `## Rough estimate` (t-shirt sizing) ·
`## Epics and issues` · `## Functional spec post-release` · `## Notes` · `## History`.

---

## taxonomy.md (condensed, for `00-meta/`)

Document `type` values and their folders:

| type | contains | folder |
|---|---|---|
| product | product/architecture overview | root `architecture.md` |
| functional-doc | feature from end-user POV, no code | `functional/` |
| technical-spec | current technical behaviour, business rules, code refs | `technical/<area>/` |
| api-spec | endpoint contracts | `technical/api/` |
| decision | ADR with context + alternatives | `decisions/` |
| feature-discovery | PRD light for a feature in discovery | (optional) |

`status`: draft (don't cite as source) · active (authoritative) · deprecated · archived.
`sensitivity`: public · internal · confidential · restricted.
`ai_usage`: allowed · restricted · forbidden.
