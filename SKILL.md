---
name: kb-kickoff
description: "Bootstrap a brand-new project with a structured knowledge base (architecture + functional + technical + infrastructure + decisions + design), built from a guided interview, then emit ready-to-run prompts to build and implement the project from that KB. Use when the user wants to start a new project from scratch, scaffold project docs/specs before coding, or set up a knowledge base like the company-knowledge product layout."
trigger: /kb-kickoff
---

# /kb-kickoff

Turn an idea into a **complete, self-contained project knowledge base** through a guided interview, then hand the user a set of implementation prompts that build the project *from* that KB.

This skill does two things, in order:

1. **Builds the KB** — interviews the user (functional analysis, technical analysis, infrastructure, existing design, key decisions), then writes a populated, cross-linked doc tree using the company-knowledge templates and conventions.
2. **Emits implementation prompts** — writes `PROMPTS.md`, a sequenced set of copy-paste prompts that drive an agent to scaffold and implement the project grounded in the KB.

It does **not** write application code itself. The deliverable is the KB plus the prompts. Stop after that unless the user explicitly asks you to start implementing.

## Usage

```
/kb-kickoff                          # interview + generate KB in a new ./<project-slug>/ folder
/kb-kickoff <project-name>           # same, pre-seeding the project name
/kb-kickoff <path>                   # generate the KB at a specific path
/kb-kickoff --from <brief-file.md>   # read an existing brief/notes file first, then only ask what's missing
/kb-kickoff --design <dir-or-file>   # design already exists here (prototype HTML, Figma export, mockups): read it, don't invent it
/kb-kickoff --resume                 # an interview was interrupted; re-read what's on disk and continue
```

## Target layout (lean, per-project, self-contained)

Generate exactly this tree. It is portable — it does **not** depend on any company vault being present.

```
<project>/
  CLAUDE.md                      # how an agent should use this KB (navigation + rules)
  README.md                      # what this project is, index of the KB
  00-meta/
    taxonomy.md                  # allowed frontmatter values (condensed)
    templates/                   # the blank templates, so the KB can grow consistently
  architecture.md                # type: product — high-level technical architecture
  functional/
    README.md                    # index of functional docs
    <use-case>.md                # one functional-doc per primary user flow
    assets/                      # screenshots/mockups referenced by functional docs
  technical/
    README.md                    # index of technical specs
    api/
      <api-group>.md             # one api-spec per endpoint group (if there's an API)
    <area>/
      <feature>.md               # one technical-spec per feature/area
  infrastructure/
    overview.md                  # runtime topology, environments
    deployment.md                # how it ships
    security.md                  # edge/auth/secrets posture
  decisions/
    0001-<slug>.md               # one ADR per significant choice
  design/
    README.md                    # what design input exists and where it lives (READ, don't invent)
  PROMPTS.md                     # sequenced implementation prompts (the build plan)
```

Omit empty branches gracefully: no API → skip `technical/api/`; no design provided → `design/README.md` records "none yet".

## Conventions (match the company-knowledge KB exactly)

- **Frontmatter on every doc.** Required keys: `title, id, type, status, owner, reviewers, created, updated, review_by, audience, sensitivity, tags, related, ai_usage`. Plus type-specific keys (see templates below). `created`/`updated` = today's date. `review_by` = today + 6 months. `owner` = the user's email if known, else a placeholder.
- **`id` patterns**: `prod-<slug>` (architecture/product), `fdoc-<product>-<slug>-NNN` (functional), `tspec-<product>-<area>-<slug>-NNN` (technical), `api-<product>-<area>-<slug>-NNN` (api), `adr-NNNN` (decisions).
- **Cross-link with `[[wikilinks]]`** in the `related:` frontmatter and inline. A functional-doc and its sibling technical-spec must link each other. ADRs link the specs they affect.
- **`status: draft`** for everything generated (it's unverified until a human reviews it).
- **functional-doc = user POV only** — no endpoints, no DB schemas, no code paths. Product vocabulary only.
- **technical-spec = engineering POV** — business rules as verifiable assertions, edge cases, states/transitions, a `code_references` frontmatter array (left as paths to fill once code exists), an Implementation section.
- **Screenshots/assets**: `assets/<slug>-NN-<descriptor>.png`, embedded inline with alt-text + caption.
- **Language**: write **all** generated docs in the language the user picks in Q0. Keep frontmatter keys in English regardless.

The blank templates to copy into `00-meta/templates/` and to follow when writing each doc are the canonical company-knowledge ones. If `~/Documents/unpublic/src/company-knowledge/00-meta/templates/` exists, copy the files from there verbatim. Otherwise, reconstruct them from the skeletons in `reference/templates-skeleton.md` next to this skill.

## What you must do when invoked

### Phase 0 — Setup (one batched question)

Resolve, asking only what's unknown. Read any `--from` brief and any `--design` input first so you don't ask what's already answered.

Ask in **one** `AskUserQuestion` batch:
1. **Output language** for the KB docs (Italian / English / other).
2. **Project type / stack family** (e.g. web app, API service, CLI, mobile, data pipeline, library) — drives which doc branches matter.
3. **Does design already exist?** (yes, I'll point you to it / no, none yet). If yes and `--design` wasn't passed, ask where.

Confirm the project name, slug, and target path. Create the folder tree (empty branches deferred until you know they're needed).

### Phase 1 — Interview (the heart of the skill)

Conduct a **structured interview**, grouped into the sections below. Mix `AskUserQuestion` (for bounded choices — maturity, roles, yes/no, which areas) with **plain free-text questions** (for descriptions, rules, flows — these don't fit multiple choice). Ask one group at a time; reflect answers back briefly before moving on. Adapt: skip groups that don't apply (no API → skip API questions).

Do **not** invent product facts. If the user doesn't know something, mark it `> TODO` in the doc rather than fabricating. The one thing you may infer is standard engineering scaffolding (folder conventions, typical edge cases to *prompt about*) — but flag inferences.

**Group A — Product overview** (→ `architecture.md` intro, `README.md`)
- One-liner: what it does, for whom, the core value.
- Maturity target (MVP / beta / production), current version.
- Primary stack & languages/frameworks (confirm or refine the Phase-0 guess).
- Main components and what each does.

**Group B — Functional analysis** (→ one `functional/<use-case>.md` per flow)
- Who are the user roles/personas? (one functional doc set is scoped per role mix)
- What are the primary user flows / jobs-to-be-done? Enumerate them — each becomes a functional-doc.
- For each flow: what the user does, what they see, the happy path, what can go wrong (user-language), limits/rules worth knowing.
- Any compliance/privacy expectations the user cares about.

**Group C — Technical analysis** (→ `architecture.md` + `technical/<area>/<feature>.md`)
- High-level architecture: processes/services, how they talk, data stores, external dependencies, AI/LLM usage if any.
- Break the system into **areas/features** (auth, billing, the core domain, etc.) — each becomes a technical-spec.
- For each area: expected behaviour, the explicit **business rules** (verifiable assertions), edge cases, any stateful entities (states + transitions), data model sketch.
- Is there an HTTP/API surface? If yes → enumerate endpoint groups for `technical/api/` (method, path, auth, request/response shape, error codes).
- Non-functional needs: performance, scale, SLAs, security posture.

**Group D — Infrastructure** (→ `infrastructure/*.md`)
- Runtime topology and environments (local/staging/prod).
- How it deploys (containers, serverless, CI/CD, hosting).
- Security & secrets posture, edge/CDN/WAF, auth boundary.

**Group E — Design** (→ `design/README.md`)
- If design exists: read the provided files/prototype/Figma export. Summarize what screens/components exist, the visual language, and **link to the source files** — do not redraw or re-specify it; the KB just points to it and notes design decisions worth capturing.
- If none: record `status: none yet` and list the screens that will need design, derived from the functional flows.

**Group F — Key decisions** (→ `decisions/NNNN-*.md`)
- Surface 2–6 decisions that are genuinely consequential and not obvious (stack choice, a build-vs-buy, a data-model fork, an auth model). For each: context, the decision, alternatives considered, consequences. One ADR each. Don't manufacture ADRs for trivial defaults.

### Phase 2 — Generate the KB

- Create `00-meta/templates/` (copy or reconstruct) and `00-meta/taxonomy.md` (condensed allowed-values table).
- Write every doc from its template, fully populated from the interview, in the chosen language, with correct frontmatter, ids, and `[[wikilinks]]`. Unknowns become `> TODO:` lines, never fabrications.
- Write `README.md` (project summary + a linked index of every KB doc) and `CLAUDE.md` (a short "how to use this KB" guide adapted from the agents-instructions: prefer `active` docs, cite paths, follow wikilinks, functional vs technical split).
- After writing, print a tree of what was created and a one-line note of every `TODO` left open, so the user knows the gaps.

### Phase 3 — Emit implementation prompts (`PROMPTS.md`)

Write `PROMPTS.md`: an ordered sequence of **copy-paste-ready prompts** that build the project from the KB. Each prompt must:
- Reference the specific KB docs it depends on (by path), so the implementing agent reads the spec, not guesses.
- Be self-contained and produce a verifiable increment.

Default sequence (adapt to the actual project):
1. **Scaffold** — repo/monorepo layout, tooling, CI skeleton, per `architecture.md` + `infrastructure/`.
2. **Data layer** — schema/migrations from the data models in the technical specs.
3. **Per-area implementation** — one prompt per `technical/<area>/<feature>.md`, instructing the agent to implement to that spec's business rules and edge cases, then backfill the spec's `code_references`.
4. **API surface** — endpoints per `technical/api/*.md`.
5. **Frontend / UI** — per functional flows + `design/` (build to the existing design if provided).
6. **Tests** — assert the business rules and edge cases enumerated in each technical-spec.
7. **Wire-up & verify** — end-to-end against the functional happy paths.

End `PROMPTS.md` with a short "How to use this file" note: run prompts in order, review each diff against the cited spec, update the spec's `status: draft → active` and fill `code_references` once implemented.

## Guardrails

- Interview first, generate second. Never emit a KB full of invented specifics — `TODO` beats fiction.
- One concept per file; keep functional and technical strictly separated.
- Everything `status: draft` until a human reviews.
- Keep the output self-contained and portable — no dependency on an external vault at read time.
- Stop after the KB + `PROMPTS.md`. Implementing is a separate, explicit step.
