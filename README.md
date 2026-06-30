# kb-kickoff

A [Claude Code](https://claude.com/claude-code) skill that bootstraps a brand-new project with a **structured, self-contained knowledge base** — then hands you ready-to-run prompts to build the project *from* that KB.

You answer a guided interview. It writes the docs. You get a spec-first project instead of a blank repo.

```
/kb-kickoff
```

---

## What it does

The skill runs in three phases:

1. **Builds the KB** — interviews you across functional analysis, technical analysis, infrastructure, existing design, and key decisions, then writes a populated, cross-linked documentation tree.
2. **Emits implementation prompts** — writes `PROMPTS.md`, a sequenced set of copy-paste prompts that drive an agent to scaffold and implement the project, each grounded in the specs it depends on.
3. **Stops.** It does *not* write application code unless you explicitly ask. The deliverable is the KB + the prompts.

The doc structure and conventions mirror a real product knowledge base: a functional/technical split, ADRs for decisions, frontmatter on every doc, and `[[wikilinks]]` between related docs.

## Why

A new repo usually starts as a blank folder and a vague idea. Code gets written before anyone writes down what the system is supposed to *do*, what the rules are, or why a choice was made. `kb-kickoff` inverts that: it forces the thinking up front into durable, navigable docs, and the implementation prompts keep the build anchored to those docs instead of to guesswork.

## What it generates

A lean, portable, per-project tree (no external vault dependency):

```
<project>/
  CLAUDE.md            # how an agent should use this KB
  README.md            # project summary + index of the KB
  00-meta/
    taxonomy.md        # allowed frontmatter values
    templates/         # blank templates so the KB can grow consistently
  architecture.md      # high-level technical architecture (type: product)
  functional/          # one doc per user flow — user POV, no code
  technical/           # one spec per feature/area — business rules, edge cases, data model
    api/               # endpoint contracts (if there's an API)
  infrastructure/      # topology, deployment, security
  decisions/           # one ADR per significant choice
  design/              # points to existing design (prototype/Figma) — read, don't invent
  PROMPTS.md           # sequenced implementation prompts (the build plan)
```

Empty branches are omitted gracefully (no API → no `technical/api/`).

## Conventions

- **Frontmatter on every doc** (`title, id, type, status, owner, created, updated, review_by, audience, sensitivity, tags, related, ai_usage`) plus type-specific keys.
- **Functional ≠ technical.** Functional docs are user-POV only — no endpoints, schemas, or code paths. Technical specs carry verifiable business rules, edge cases, states/transitions, and a `code_references` array to fill once code exists.
- **Everything ships as `status: draft`** until a human reviews it.
- **`> TODO:` beats fiction.** Unknowns are flagged, never fabricated.
- **You pick the language** at the start; all docs are written in it (frontmatter keys stay English).

## Usage

```
/kb-kickoff                          # interview + generate KB in ./<project-slug>/
/kb-kickoff <project-name>           # pre-seed the project name
/kb-kickoff <path>                   # generate at a specific path
/kb-kickoff --from <brief-file.md>   # read an existing brief first, ask only what's missing
/kb-kickoff --design <dir-or-file>   # design already exists here: read it, don't invent it
/kb-kickoff --resume                 # continue an interrupted interview from what's on disk
```

## Install

This is a Claude Code skill. Drop the folder into your skills directory:

```bash
git clone https://github.com/nzereik/kb-kickoff.git ~/.claude/skills/kb-kickoff
```

Then, optionally, register the trigger in your `~/.claude/CLAUDE.md` so it's always discoverable:

```markdown
# kb-kickoff
- **kb-kickoff** (`~/.claude/skills/kb-kickoff/SKILL.md`) - bootstrap a new project: guided interview → structured knowledge base → implementation prompts. Trigger: `/kb-kickoff`
When the user types `/kb-kickoff`, invoke the Skill tool with `skill: "kb-kickoff"` before doing anything else.
```

Restart Claude Code (or start a new session) and run `/kb-kickoff`.

## Layout of this repo

```
kb-kickoff/
  SKILL.md                        # the skill definition (what Claude Code loads)
  reference/
    templates-skeleton.md         # fallback doc templates, keeps the skill self-contained
  README.md                       # this file
```

## License

MIT — see `LICENSE`.
